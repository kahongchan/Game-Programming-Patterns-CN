^title 字節碼 Bytecode
^section Behavioral Patterns

## 意圖

*將行爲編碼爲虛擬機器上的指令，賦予其數據的靈活性。*

## 動機

<span name="sprawling"></span>

製作遊戲也許很有趣，但絕不容易。
現代遊戲的代碼庫很是龐雜。
主機廠商和應用市場有嚴格的質量要求，
小小的崩潰漏洞就能阻止遊戲發售。

<aside name="sprawling">

我曾參與製作有六百萬行C++代碼的遊戲。作爲對比，控制好奇號火星探測器的軟件還沒有其一半大小。

</aside>

與此同時，我們希望榨乾平臺的每一點性能。
遊戲對硬件發展的推動首屈一指，只有堅持不懈地優化才能跟上競爭。

爲了保證穩定和性能的需求，我們使用如C++這樣的重量級的編程語言，它既有能兼容多數硬件的底層表達能力，又擁有防止漏洞的強類型系統。

我們對自己的專業技能充滿自信，但其亦有代價。
專業程序員需要多年的訓練，之後又要對抗代碼規模的增長。
構建大型遊戲的時間長度可以在“喝杯咖啡”和
“烤咖啡豆，手磨咖啡豆，弄杯espresso，打奶泡，在拿鐵咖啡里拉花。”之間變動。

除開這些挑戰，遊戲還多了個苛刻的限制：“樂趣”。
玩家需要仔細權衡過的新奇體驗。
這需要不斷的迭代，但是如果每個調整都需要讓工程師修改底層代碼，然後等待漫長的編譯結束，那就毀掉了創作流程。

### 法術戰鬥！

假設我們在完成一個基於法術的格鬥遊戲。
兩個敵對的巫師互相丟法術，直到分出勝負。
我們可以將這些法術都定義在代碼中，但這就意味着每次修改法術都會牽扯到工程師。
當設計者想修改幾個數字感覺一下效果，就要重新編譯整個工程，重啓，然後進入戰鬥。

像現在的許多遊戲一樣，我們也需要在發售之後更新遊戲，修復漏洞或是添加新內容。
如果所有法術都是硬編碼的，那麼每次修改都意味着要給遊戲的可執行文件打補丁。

再扯遠一點，假設我們還想支持*模組*。我們想讓*玩家*創造自己的法術。
如果這些法術都是硬編碼的，那就意味着每個模組製造者都得擁有編譯遊戲的整套工具鏈，
我們也就不得不開放源代碼，如果他們的自創法術上有個漏洞，那麼就會把其他人的遊戲也搞崩潰。

### 數據 &gt; 代碼

很明顯實現引擎的編程語言不是個好選擇。
我們需要將法術放在與遊戲核心隔絕的沙箱中。
我們想要它們易於修改，易於加載，並與其他可執行部分相隔離。

我不知道你怎麼想，但這聽上去讓我覺得有點像是*數據*。
如果能在分離的數據文件中定義行爲，遊戲引擎還能加載並“執行”它們，就可以實現所有目標。

這裏需要指出“執行”對於數據的意思。如何讓文件中的數據表示爲行爲呢？這裏有幾種方式。
與<a href="http://en.wikipedia.org/wiki/Interpreter_pattern" class="gof-pattern">解釋器模式</a>對比着看會好理解些。

### 解釋器模式

關於這個模式我就能寫整整一章，但是有四個傢伙的工作早涵蓋了這一切，
所以，這裏給一些簡短的介紹。

它源於一種你想要執行的語言——想想*編程*語言。

比如，它支持這樣的算術表達式

    (1 + 2) * (3 - 4)

然後，把每塊表達式，每條語言規則，都裝到*對象*中去。數字字面量都變成對象：

<img src="images/bytecode-numbers.png" alt="一系列數字字面量對象。" />

<span name="magic"></span>

簡單地說，它們在原始值上做了個小封裝。
運算符也是對象，它們擁有操作數的引用。
如果你考慮了括號和優先級，那麼表達式就魔術般變成這樣的小樹：

<img src="images/bytecode-ast.png" alt="一個語法樹。數字字面量被運算符對象連接着。" />

<aside name="magic">

這裏的“魔術”是什麼？很簡單——*語法分析*。
語法分析器接受一串字符作爲輸入，將其轉爲*抽象語法樹*，即一個包含了表示文本語法結構的對象集合。

完成這個你就得到了半個編譯器。

</aside>

解釋器模式與*創建*這棵樹無關，它只關於*執行*這棵樹。
它工作的方式非常巧妙。樹中的每個對象都是表達式或子表達式。
用真正面向對象的方式描述，我們會讓表達式自己對自己求值。

首先，我們定義所有表達式都實現的基本接口：

^code expression

然後，爲我們語言中的每種語法定義一個實現這個接口的類。最簡單的是數字：

^code number

一個數字表達式就等於它的值。加法和乘法有點複雜，因爲它們包含子表達式。在一個表達式計算自己的值之前，必須先遞歸地計算其子表達式的值。像這樣：

<span name="addition"></span>

^code addition

<aside name="addition">

你肯定能想明白乘法的實現是什麼樣的。

</aside>

很優雅對吧？只需幾個簡單的類，現在我們可以表示和計算任意複雜的算術表達式。
只需要創建正確的對象，並正確地連起來。

<aside name="ruby">

Ruby用了這種實現方法差不多15年。在1.9版本，他們轉換到了本章所介紹的字節碼。看看我給你節省了多少時間！

</aside>

<span name="ruby"></span>
這是個優美、簡單的模式，但它有一些問題。
看看插圖，看到了什麼？大量的小盒子，以及它們之間大量的箭頭。
代碼被表示爲小物體組成的巨大分形樹。這會帶來些令人不快的後果：

* 從磁盤上加載它需要實例化並連接成噸的小對象。

<span name="vtable"></span>

* 這些對象和它們之間的指針會佔據大量的內存。在32位機上，那個小的算術表達式至少要佔據68字節，這還沒考慮內存對其呢。

    <aside name="vtable">

    如果你想自己算算，別忘了算上虛函數表指針。

    </aside>

<span name="cache"></span>

* 順着那些指針遍歷子表達式是對數據緩存的謀殺。同時，虛函數調用是對指令緩存的屠戮。

    <aside name="cache">

    參見<a href="data-locality.html" class="pattern">數據局部性</a>一章以瞭解什麼是緩存以及它是如何影響遊戲性能的。

    </aside>

將這些拼到一起，怎麼念？S-L-O-W。
這就是爲什麼大多數廣泛應用的編程語言不基於解釋器模式：
太慢了，也太消耗內存了。

### 虛擬的機器碼

想想我們的遊戲。玩家電腦在運行遊戲時並不會遍歷一堆C++語法結構樹。
我們提前將其編譯成了機器碼，CPU基於機器碼運行。機器碼有什麼好處呢？

* *密集。* 它是一塊堅實連續的二進制數據塊，沒有一位被浪費。

* *線性。* 指令被打成包，一條接一條地執行。不會在內存裏到處亂跳（除非你的控制流代碼真真這麼幹了）。

* *底層。* 每條指令都做一件小事，有趣的行爲從*組合*中誕生。

* *速度快。* 綜合所有這些條件（當然，也包括它直接由硬件實現這一事實），機器碼跑得跟風一樣快。

<span name="jit"></span>

這聽起來很好，但我們不希望真的用機器代碼來寫咒語。
讓玩家提供遊戲運行時的機器碼簡直是在自找麻煩。我們需要的是機器代碼的性能和解釋器模式的安全的折中。

如果不是加載機器碼並直接執行，而是定義自己的*虛擬*機器碼呢？
然後，在遊戲中寫個小模擬器。
這與機器碼類似——密集，線性，相對底層——但也由遊戲直接掌控，所以可以放心地將其放入沙箱。

<aside name="jit">

這就是爲什麼很多遊戲主機和iOS不允許程序在運行時生成並加載機器碼。
這是一種拖累，因爲最快的編程語言實現就是那麼做的。
它們包含了“即時（just-in-time）”編譯器，或者*JIT*，在運行時將語言翻譯成優化的機器碼。

</aside>

<span name="virtual"></span>

我們將小模擬器稱爲*虛擬機*（或簡稱“VM”），它運行的二進制機器碼叫做*字節碼*。
它有數據的靈活性和易用性，但比解釋器模式性能更好。

<aside name="virtual">

在程序語言編程圈，“虛擬機”和“解釋器”是同義詞，我在這裏交替使用。
當指代GoF的解釋器模式，我會加上“模式”來表明區別。

</aside>

這聽起來有點嚇人。
這章其餘部分的目標是爲了展示一下，如果把功能列表縮減下來，它實際上相當通俗易懂。
即使最終沒有使用這個模式，你也至少可以對Lua和其他使用了這一模式的語言有個更好的理解。

## 模式

**指令集** 定義了可執行的底層操作。
一系列的指令被編碼爲**字節序列**。
**虛擬機** 使用 **中間值棧** 依次執行這些指令。
通過組合指令，可以定義複雜的高層行爲。

## 何時使用

這是本書中最複雜的模式，無法輕易地加入遊戲中。這個模式應當用在你有許多行爲需要定義，而遊戲實現語言因爲如下原因不適用時：

* 過於底層，繁瑣易錯。
* 編譯慢或者其他工具因素導致迭代緩慢。
* 安全性依賴編程者。如果想保證行爲不會破壞遊戲，你需要將其與代碼的其他部分隔開。

當然，該列表描述了一堆特性。誰不希望有更快的迭代循環和更多的安全性？
然而，世上沒有免費的午餐。字節碼比本地代碼慢，所以不適合引擎的性能攸關的部分。

## 記住

<span name="seductive"></span>

創建自己的語言或者建立系統中的系統是很有趣的。
我在這裏做的是小演示，但在現實項目中，這些東西會像藤蔓一樣蔓延。

<aside name="seductive">

對我來說，遊戲開發也正因此而有趣。
不管哪種情況，我都創建了虛擬空間讓他人遊玩。

</aside>

<span name="template"></span>

每當我看到有人定義小語言或腳本系統，他們都說，“別擔心，它很小。”
於是，不可避免地，他們增加更多小功能，直到完成了一個完整的語言。
除了，和其它語言不同，它是定製的並擁有棚戶區的建築風格。

<aside name="template">

例如每一種模板語言。

</aside>

當然，完成完整的語言並沒有什麼*錯*。只是要確定你做得慎重。 否則，你就要小心地控制你的字節碼所能表達的範圍。在野馬脫繮之前把它拴住。

### 你需要一個前端

<span name="assembly"></span>

底層的字節碼指令性能優越，但是二進制的字節碼格式*不是*用戶能寫的。
我們將行爲移出代碼的一個原因是想要以更*高層*的形式表示它。
如果說寫C++太過底層，那麼讓用戶寫彙編可不是一個改進方案——就算是你設計的！

<aside name="assembly">

一個反例的是令人尊敬的遊戲[RoboWar](http://en.wikipedia.org/wiki/RoboWar)。
在遊戲中，*玩家* 編寫類似彙編的語言控制機器人，我們這裏也會討論這種指令集。

這是我介紹類似彙編的語言的首選。

</aside>

就像GoF的解釋器模式，它假設你有某些方法來*生成*字節碼。
通常情況下，用戶在更高層編寫行爲，再用工具將其翻譯爲虛擬機能理解的字節碼。
這裏的工具就是編譯器。

我知道，這聽起來很嚇人。醜話說在前頭，
如果沒有資源製作編輯器，那麼字節碼不適合你。
但是，接下來你會看到，也可能沒你想的那麼糟。

### 你會想念調試器

編程很難。我們知道想要機器做什麼，但並不總能正確地傳達——所以我們會寫出漏洞。
爲了查找和修復漏洞，我們已經積累了一堆工具來了解代碼做錯了什麼，以及如何修正。
我們有調試器，靜態分析器，反編譯工具等。
所有這些工具都是爲現有的語言設計的：無論是機器碼還是某些更高層次的東西。

當你定義自己的字節碼虛擬機時，你就得把這些工具拋在腦後了。
當然，可以通過調試器調試虛擬機，但它告訴你虛擬機*本身*在做什麼，而不是正在被翻譯的字節碼是幹什麼的。

它當然也不會把字節碼映射回編譯前的高層次的形式。

<span name="debugger"></span>

如果你定義的行爲很簡單，可能無需太多工具幫忙調試就能勉強堅持下來。
但隨着內容規模增長，還是應該花些時間完成些功能，讓用戶看到字節碼在做什麼。
這些功能也許不隨遊戲發佈，但它們至關重要，它們能確保你的遊戲*能*被髮布。

<aside name="debugger">

當然，如果你想要讓遊戲支持模組，那你*會*發佈這些特性，它們就更加重要了。

</aside>

## 示例代碼

經歷了前面幾個章節後，你也許會驚訝於它的實現是多麼直接。
首先需要爲虛擬機設定一套指令集。
在開始考慮字節碼之類的東西前，先像思考API一樣思考它。

### 法術的API

如果直接使用C++代碼定義法術，代碼需要調用何種API呢？
在遊戲引擎中，構成法術的基本操作是什麼樣的？

大多數法術最終改變一個巫師的狀態，因此先從這樣的代碼開始。

^code magic-api

第一個參數指定哪個巫師被影響，`0`代表玩家而`1`代表對手。
以這種方式，治癒法術可以治療玩家的巫師，而傷害法術傷害他的敵人。
這三個小方法能覆蓋的法術出人意料地多。

如果法術只是默默地調整數據，遊戲邏輯就已經完成了，
但玩這樣的遊戲會讓玩家無聊得要哭。讓我們修復這點：

^code magic-api-fx

這並不影響遊戲玩法，但它們增強了遊戲的*體驗*。
我們可以增加一些鏡頭晃動，動畫之類的，但這足夠我們開始了。

### 法術指令集

現在讓我們把這種*程序化*的API轉化爲可被數據控制的東西。
從小處開始，然後慢慢拓展到整體。
現在，要去除方法的所有參數。
假設`set__()`方法總影響玩家的巫師，總直接將狀態設爲最大值。
同樣，FX操作總是播放一個硬編碼的聲音和粒子效果。

這樣，一個法術就只是一系列指令了。
每條指令都代表了想要呈現的操作。我們可以枚舉如下：

^code instruction-enum

<span name="byte"></span>
爲了將法術編碼進數據，我們存儲了一數組`enum`值。
只有幾個不同的基本操作原語，因此`enum`值的範圍可以存儲到一個字節中。
這就意味着法術的代碼就是一系列字節——也就是“字節碼”。

<img src="images/bytecode-code.png" alt="一系列字節碼指令：0x00 HEALTH, 0x03 SOUND, 0x004 PARTICLES, ..." />

<aside name="byte">

有些字節碼虛擬機爲每條指令使用多個字節，解碼規則也更復雜。
事實上，在x86這樣的常見芯片上的機器碼更加複雜。

但單字節對於[Java虛擬機](http://en.wikipedia.org/wiki/Java_virtual_machine)和支撐了.NET平臺的[Common Language Runtime](http://en.wikipedia.org/wiki/Common_Language_Runtime)已經足夠了，對我們來說也一樣。

</aside>

爲了執行一條指令，我們看看它的基本操作原語是什麼，然後調用正確的API方法。

^code interpret-instruction

用這種方式，解釋器建立了溝通代碼世界和數據世界的橋樑。我們可以像這樣將其放進執行法術的虛擬機：

^code vm

輸入這些，你就完成了你的首個虛擬機。
不幸的是，它並不靈活。
我們不能設定攻擊對手的法術，也不能減少狀態上限。我們只能播放聲音！

爲了獲得像一個真正的語言那樣的表達能力，我們需要在這裏引入參數。

### 棧式機器

要執行復雜的嵌套表達式，得先從最裏面的子表達式開始。
計算完裏面的，將結果作爲參數向外流向包含它們的表達式，
直到得出最終結果，整個表達式就算完了。

<span name="stack-machine"></span>

解釋器模式將其明確地表現爲嵌套對象組成的樹，但我們需要指令速度達到列表的速度。我們仍然需要確保子表達式的結果正確地向外傳遞給包括它的表達式。

但由於數據是扁平的，我們得使用指令的*順序*來控制這一點。我們的做法和CPU一樣——使用棧。

<aside name="stack-machine">

這種架構不出所料地被稱爲[*棧式計算機*](http://en.wikipedia.org/wiki/Stack_machine)。像[Forth](http://en.wikipedia.org/wiki/Forth_(programming_language))，[PostScript](http://en.wikipedia.org/wiki/PostScript)，和[Factor](http://en.wikipedia.org/wiki/Factor_(programming_language)) 這些語言直接將這點暴露給用戶。

</aside>

^code stack

虛擬機用內部棧保存值。在例子中，指令交互的值只有一種，那就是數字，
所以可以使用簡單的`int`數組。
每當數據需要從一條指令傳到另一條，它就得通過棧。

顧名思義，值可以壓入棧或者從棧彈出，所以讓我們添加一對方法。

^code push-pop

當一條指令需要接受參數，就將參數從棧彈出，如下所示：

^code pop-instructions

爲了將一些值*存入*棧中，需要另一條指令：字面量。
它代表了原始的整數值。但是*它*的值又是從哪裏來的呢？
我們怎麼樣避免這樣追根溯源到無窮無盡呢？

技巧是利用指令是字節序列這一事實——我們可以直接將數值存儲在字節數組中。
如下，我們爲數值字面量定義了另一條指令類型：

^code interpret-literal

<aside name="single">

這裏，從單個字節中讀取值，從而避免瞭解碼多字節整數需要的代碼，
但在真實實現中，你會需要支持整個數域的字面量。

</aside>

<img src="images/bytecode-literal.png" alt="字面量指令的二進制編碼：0x05 (字面量) 之後是 123 (值)。 />

<span name="single"></span>

它讀取字節碼流中的字節*作爲數值*並將其壓入棧。

讓我們把一些這樣的指令串起來看看解釋器的執行，感受下棧是如何工作的。
從空棧開始，解釋器指向第一個指令：

<img src="images/bytecode-stack-1.png" alt="執行一個字節碼序列。執行指針指向第一個字面量指令，棧是空的。" />

首先，它執行第一條`INST_LITERAL`，讀取字節碼流的下一個字節(`0`)並壓入棧中。

<img src="images/bytecode-stack-2.png" alt="下一步。字面量0倍壓入到了棧中，執行指針指向了下一個字面量。" />

然後，它執行第二條`INST_LITERAL`，讀取`10`然後壓入。

<img src="images/bytecode-stack-3.png" alt="下一步。現在10倍壓入了棧中，執行指針指向了Health指令。" />

最後，執行`INST_SET_HEALTH`。這會彈出`10`存進`amount`，彈出`0`存進`wizard`。然後用這兩個參數調用`setHealth()`。

完成！我們獲得了將玩家巫師血量設爲10點的法術。
現在我們擁有了足夠的靈活度，來定義修改任一巫師的狀態到任意值的法術。
我們還可以放出不同的聲音和粒子效果。

但是……這感覺還是像*數據*格式。比如，不能將巫師的血量提升爲他智力的一半。
設計師希望法術能表達*規則*，而不僅僅是*數值*。

### 行爲 = 組合

如果我們視小虛擬機爲編程語言，現在它能支持的只有一些內置函數，以及常量參數。
爲了讓字節碼感覺像*行爲*，我們缺少的是*組合*。

設計師需要能以有趣的方式組合不同的值，來創建表達式。
舉個簡單的例子，他們想讓法術*變化*一個數值而不是*變到*一個數值。

這需要考慮到狀態的當前值。
我們有指令來*修改*狀態，現在需要添加方法*讀取*狀態：

^code read-stats

正如你所看到的，這要與棧雙向交互。
彈出一個參數來確定獲取哪個巫師的狀態，然後查找狀態的值並壓入棧中。

這允許我們創造複製狀態的法術。
我們可以創建一個法術，根據巫師的智慧設定敏捷度，或者讓巫師的血量等於對方的血量。

有所改善，但仍很受限制。接下來，我們需要算術。
是時候讓小虛擬機學習如何計算1 + 1了，我們將添加更多的指令。
現在，你可能已經知道如何去做，猜到了大概的模樣。我只展示加法：

^code add

像其他指令一樣，它彈出數值，做點工作，然後壓入結果。
直到現在，每個新指令似乎都只是有所改善而已，但其實我們已完成大飛躍。
這並不顯而易見，但現在我們可以處理各種複雜的，深層嵌套的算術表達式了。

來看個稍微複雜點的例子。
假設我們希望有個法術，能讓巫師的血量增加敏捷和智慧的平均值。
用代碼表示如下：

^code increase-health

你可能會認爲我們需要指令來處理括號造成的分組，但棧隱式支持了這一點。可以手算如下：

1. 獲取巫師當前的血量並記錄。
2. 獲取巫師敏捷並記錄。
3. 對智慧執行同樣的操作。
4. 獲取最後兩個值，加起來並記錄。
5. 除以二並記錄。
6. 回想巫師的血量，將它和這結果相加並記錄。
7. 取出結果，設置巫師的血量爲這一結果。

你看到這些“記錄”和“回想”了嗎？每個“記錄”對應一個壓入，“回想”對應彈出。
這意味着可以很容易將其轉化爲字節碼。例如，第一行獲得巫師的當前血量：

    :::text
    LITERAL 0
    GET_HEALTH

這些字節碼將巫師的血量壓入堆棧。
如果我們機械地將每行都這樣轉化，最終得到一大塊等價於原來表達式的字節碼。
爲了讓你感覺這些指令是如何組合的，我在下面給你做個示範。

爲了展示堆棧如何隨着時間推移而變化，我們舉個代碼執行的例子。
巫師目前有45點血量，7點敏捷，和11點智慧。
每條指令的右邊是棧在執行指令之後的模樣，再右邊是解釋指令意圖的註釋：

    :::text
    LITERAL 0    [0]            # 巫師索引
    LITERAL 0    [0, 0]         # 巫師索引
    GET_HEALTH   [0, 45]        # 獲取血量()
    LITERAL 0    [0, 45, 0]     # 巫師索引
    GET_AGILITY  [0, 45, 7]     # 獲取敏捷()
    LITERAL 0    [0, 45, 7, 0]  # 巫師索引
    GET_WISDOM   [0, 45, 7, 11] # 獲取智慧()
    ADD          [0, 45, 18]    # 將敏捷和智慧加起來
    LITERAL 2    [0, 45, 18, 2] # 被除數：2
    DIVIDE       [0, 45, 9]     # 計算敏捷和智慧的平均值
    ADD          [0, 54]        # 將平均值加到現有血量上。
    SET_HEALTH   []             # 將結果設爲血量

<span name="threshold"></span>

如果你注意每步的棧，你可以看到數據如何魔法一般地在其中流動。
我們最開始壓入`0`來查找巫師，然後它一直掛在棧的底部，直到最終的`SET_HEALTH`纔用到它。

<aside name="threshold">

也許“魔法”在這裏的門檻太低了。

</aside>

### 一臺虛擬機

我可以繼續下去，添加越來越多的指令，但是時候適可而止了。
如上所述，我們已經有了一個可愛的小虛擬機，可以使用簡單，緊湊的數據格式，定義開放的行爲。
雖然“字節碼”和“虛擬機”的聽起來很嚇人，但你可以看到它們往往簡單到只需棧，循環，和switch語句。

還記得我們最初的讓行爲呆在沙盒中的目標嗎？
現在，你已經看到虛擬機是如何實現的，很明顯，那個目標已經完成。
字節碼不能把惡意觸角伸到遊戲引擎的其他部分，因爲我們只定義了幾個與其他部分接觸的指令。

<span name="looping"></span>

我們通過控制棧的大小來控制內存使用量，並很小心地確保它不會溢出。
我們甚至可以控制它使用多少*時間*。
在指令循環裏，可以追蹤已經執行了多少指令，如果遇到了問題也可以擺脫困境。

<aside name="looping">

控制運行時間在例子中沒有必要，因爲沒有任何循環的指令。
可以限制字節碼的總體大小來限制運行時間。
這也意味着我們的字節碼不是圖靈完備的。

</aside>

現在就剩一個問題了：創建字節碼。
到目前爲止，我們使用僞代碼，再手工編寫爲字節碼。
除非你有*很多*的空閒時間，否則這種方式並不實用。

### 語法轉換工具

我們最初的目標是創造更*高層*的方式來控制行爲，但是，我們卻創造了比C++更*底層*的東西。
它具有我們想要的運行性能和安全性，但絕對沒有對設計師友好的可用性。

爲了填補這一空白，我們需要一些工具。
我們需要一個程序，讓用戶定義法術的高層次行爲，然後生成對應的低層棧式機字節碼。

<span name="dragon"></span>

這可能聽起來比虛擬機更難。
許多程序員都在大學參加編譯器課程，除了被龍書或者"[lex](http://en.wikipedia.org/wiki/Lex_(software))"和"[yacc](http://en.wikipedia.org/wiki/Yacc)&rdquo;引發了PTSD外，什麼也沒真正學到。

<aside name="dragon">

我指的，當然，是經典教材[*Compilers: Principles, Techniques, and Tools*](http://en.wikipedia.org/wiki/Compilers:_Principles,_Techniques,_and_Tools)。

</aside>

事實上，編譯一個基於文本的語言並不那麼糟糕，儘管把這個話題放進這裏來要牽扯的東西有*點*多。但是，你不是非得那麼做。
我說，我們需要的是*工具*——它並不一定是個輸入格式是*文本文件*的*編譯器*。

相反，我建議你考慮構建圖形界面讓用戶定義自己的行爲，
尤其是在使用它的人沒有很高的技術水平時。
沒有花幾年時間習慣編譯器怒吼的人很難寫出沒有語法錯誤的文本。

你可以建立一個應用程序，用戶通過單擊拖動小盒子，下拉菜單項，或任何有意義的行爲創建“腳本”，從而創建行爲。

<span name="text"></span>

<img src="images/bytecode-ui.png" alt="編寫行爲的樹狀結構UI" />

<aside name="text">

我爲[Henry Hatsworth in the Puzzling Adventure][hatsworth]編寫的腳本系統就是這麼工作的。

[hatsworth]: http://en.wikipedia.org/wiki/Henry_Hatsworth_in_the_Puzzling_Adventure

</aside>

<span name="errors"></span>

這樣做的好處是，你的UI可以保證用戶無法創建“無效的”程序。
與其向他們吐一大堆錯誤警告，不如主動禁用按鈕或提供默認值，
以確保他們創造的東西在任何時間點上都有效。

<aside name="errors">

我想要強調錯誤處理是多麼重要。作爲程序員，我們趨向於將人爲錯誤視爲應當極力避免的的個人恥辱。

爲了製作用戶喜歡的系統，你需要接受人性，*包括他們的失敗*。是人都會犯錯誤，但錯誤同時也是創作的固有基礎。
用撤銷這樣的特性優雅地處理它們，這能讓用戶更有創意，創作出更好的成果。

</aside>

這免去了設計語法和編寫解析器的工作。
但是我知道，你可能會發現UI設計同樣令人不快。
好吧，如果這樣，我就沒啥辦法啦。

畢竟，這種模式是關於使用對用戶友好的高層方式表達行爲。
你必須精心設計用戶體驗。
要有效地執行行爲，又需要將其轉換成底層形式。這是必做的，但如果你準備好迎接挑戰，這終會有所回報。

## 設計決策

<span name="failed"></span>

我想盡可能讓本章簡短，但我們所做的事情實際上可是創造語言啊。
那可是個寬泛的設計領域，你可以從中獲得很多樂趣，所以別沉迷於此反而忘了完成你的遊戲。

<aside name="failed">

這是本書中最長的章節，看來我失敗了。

</aside>

### 指令如何訪問堆棧？

字節碼虛擬機主要有兩種：基於棧的和基於寄存器的。
棧式虛擬機中，指令總是操作棧頂，如同我們的示例代碼所示。
例如，`INST_ADD`彈出兩個值，將它們相加，將結果壓入。

基於寄存器的虛擬機也有棧。唯一不同的是指令可以從棧的深處讀取值。
不像`INST_ADD`始終*彈出*其操作數，
它在字節碼中存儲兩個索引，指示了從棧的何處讀取操作數。

* **基於棧的虛擬機：**

    *  *指令短小。*
      由於每個指令隱式認定在棧頂尋找參數，不需要爲任何數據編碼。
      這意味着每條指令可能會非常短，一般只需一個字節。

    *  *易於生成代碼。*
      當你需要爲生成字節碼編寫編譯器或工具時，你會發現基於棧的字節碼更容易生成。
      由於每個指令隱式地在棧頂工作，你只需要以正確的順序輸出指令就可以在它們之間傳遞參數。

     *  *會生成更多的指令。*
        每條指令只能看到棧頂。這意味着，產生像`a = b + c`這樣的代碼，
         你需要單獨的指令將`b`和`c`壓入棧頂，執行操作，再將結果壓入`a`。

* **基於寄存器的虛擬機：**

    <span name="lua"></span>

     *  *指令較長。*
        由於指令需要參數記錄棧偏移量，單個指令需要更多的位。
         例如，一個Lua指令佔用完整的32位——它可能是最著名的基於寄存器的虛擬機了。
         它採用6位做指令類型，其餘的是參數。

    <aside name="lua">

    Lua作者沒有指定Lua的字節碼格式，它每個版本都會改變。現在描述的是Lua 5.1。
    要深究Lua的內部構造，
    讀讀[這個](http://luaforge.net/docman/83/98/ANoFrillsIntroToLua51VMInstructions.pdf)。

    </aside>

    * *指令較少。*
      由於每個指令可以做更多的工作，你不需要那麼多的指令。
      有人說，性能會得以提升，因爲不需要將值在棧中移來移去了。

所以，應該選一種？我的建議是堅持使用基於棧的虛擬機。
它們更容易實現，也更容易生成代碼。
Lua轉換爲基於寄存器的虛擬機從而變得更快，這爲寄存器虛擬機博得了聲譽，
但是這*強烈*依賴於實際的指令和虛擬機的其他大量細節。

### 你有什麼指令？

指令集定義了在字節碼中可以幹什麼，不能幹什麼，對虛擬機性能也有很大的影響。
這裏有個清單，記錄了你可能需要的不同種類的指令：

* **外部基本操作原語。**
  這是虛擬機與引擎其他部分交互，影響玩家所見的部分。
  它們控制了字節碼可以表達的真實行爲。
  如果沒有這些，你的虛擬機除了消耗CPU循環以外一無所得。

* **內部基本操作原語**
  這些語句在虛擬機內操作數值——文字，算術，比較操作，以及操縱棧的指令。

* **控制流。**
  我們的例子沒有包含這些，但當你需要有條件執行或循環執行，你就會需要控制流。
  在字節碼這樣底層的語言中，它們出奇地簡單：跳轉。

    在我們的指令循環中，需要索引來跟蹤執行到了字節碼的哪裏。
    跳轉指令做的是修改這個索引並改變將要執行的指令。
    換言之，這就是`goto`。你可以基於它制定各種更高級別的控制流。

* **抽象。**
  如果用戶開始在數據中定義*很多*的東西，最終要重用字節碼的部分位，而不是複製和粘貼。
  你也許會需要可調用過程這樣的東西。

    最簡單的形式中，過程並不比跳轉複雜。
    唯一不同的是，虛擬機需要管理另一個*返回*棧。
    當執行“call”指令時，將當前指令索引壓入棧中，然後跳轉到被調用的字節碼。
    當它到了“return”，虛擬機從堆棧彈出索引，然後跳回索引指示的位置。

### 數值是如何表示的？

我們的虛擬機示例只與一種數值打交道：整數。
回答這個問題很簡單——棧只是一棧的`int`。
更加完整的虛擬機支持不同的數據類型：字符串，對象，列表等。
你必須決定在內部如何存儲這些值。

* **單一數據類型：**

    * *簡單易用*
      你不必擔心標記，轉換，或類型檢查。

    * *無法使用不同的數據類型。*
      這是明顯的缺點。將不同類型成塞進單一的表示方式——比如將數字存儲爲字符串——這是自找麻煩。

* **帶標記的類型：**

    這是動態類型語言中常見的表示法。
    所有的值有兩部分。
    第一部分是類型標識——一個存儲了數據的類型的`enum`。其餘部分會被解釋爲這種類型：

    ^code tagged-value

    * *數值知道其類型。*
      這個表示法的好處是可在運行時檢查值的類型。
      這對動態調用很重要，可以確保沒有在類型上面執行其不支持的操作。

    * *消耗更多內存。*
      每個值都要帶一些額外的位來標識類型。在像虛擬機這樣的底層，這裏幾位，那裏幾位，總量就會快速增加。

* **無標識的union：**

    像前面一樣使用union，但是*沒有*類型標識。
    你可以將這些位表示爲不同的類型，由你確保沒有搞錯值的類型。

    <span name="untyped"></span>

    這是靜態類型語言在內存中表示事物的方式。
    由於類型系統在編譯時保證沒弄錯值的類型，不需要在運行時對其進行驗證。

    <aside name="untyped">

    這也是*無類型*語言，像彙編和Forth存儲值的方式。
    這些語言讓*用戶*保證不會寫出誤認值的類型的代碼。毫無服務態度！

    </aside>

    * *結構緊湊。*
      找不到比只存儲需要的值更加有效率的存儲方式。

    * *速度快。*
      沒有類型標識意味着在運行時無需消耗週期檢查它們的類型。這是靜態類型語言往往比動態類型語言快的原因之一。

    <span name="unsafe"></span>

    * *不安全。* 這是真正的代價。一塊錯誤的字節碼，會讓你誤解一個值，把數字誤解爲指針，會破壞遊戲安全性從而導致崩潰。

        <aside name="unsafe">

        如果你的字節碼是由靜態類型語言編譯而來，你也許認爲它是安全的，因爲編譯不會生成不安全的字節碼。
        那也許是真的，但記住惡意用戶也許會手寫惡意代碼而不經過你的編譯器。

        舉個例子，這就是爲什麼Java虛擬機在加載程序時要做*字節碼驗證*。

        </aside>

* **接口：**

    多種類型值的面向對象解決方案是通過多態。接口爲不同的類型的測試和轉換提供虛方法，如下：

    ^code value-interface

    然後你爲每個特定的數據類型設計特定的類，如：

    ^code int-value

    * *開放。*
      可在虛擬機的核心之外定義新的值類型，只要它們實現了基本接口就行。

    *  *面向對象。*
      如果你堅持OOP原則，這是“正確”的做法，爲特定類型使用多態分配行爲，而不是在標籤上做switch之類的。

    * *冗長。*
      必須定義單獨的類，包含了每個數據類型的相關行爲。
      注意在前面的例子中，這樣的類定義了*所有*的類型。在這裏，只包含了一個！

    * *低效。*
      爲了使用多態，必須使用指針，這意味着即使是短小的值，如布爾和數字，也得裹在堆中分配的對象裏。
      每使用一個值，你就得做一次虛方法調用。

        在虛擬機核心之類的地方，像這樣的性能影響會迅速疊加。
        事實上，這引起了許多我們試圖在解釋器模式中避免的問題。
        只是現在的問題不在*代碼*中，而是在*值*中。

我的建議是：如果可以，只用單一數據類型。
除此以外，使用帶標識的union。這是世界上幾乎每個語言解釋器的選擇。

### 如何生成字節碼？

我將最重要的問題留到最後。我們已經完成了*消耗*和*解釋*字節碼的部分，
但需你要寫*製造*字節碼的工具。
典型的解決方案是寫個編譯器，但它不是唯一的選擇。

* **如果你定義了基於文本的語言：**

    * *必須定義語法。*
      業餘和專業的語言設計師小看這件事情的難度。讓解析器高興很簡單，讓*用戶*快樂很*難*。

        語法設計是用戶界面設計，當你將用戶界面限制到字符構成的字符串，這可沒把事情變簡單。

    * *必須實現解析器。*
      不管名聲如何，這部分其實非常簡單。無論使用ANTLR或Bison，還是——像我一樣——手寫遞歸下降，都可以完成。

    * *必須處理語法錯誤。*
      這是最重要和最困難的部分。
      當用戶製造了語法和語義錯誤——他們總會這麼幹——引導他們返回到正確的道路是你的任務。
      解析器只知道接到了意外的符號，給予有用的的反饋並不容易。

    * *可能會對非技術用戶關上大門。*
      我們程序員喜歡文本文件。結合強大的命令行工具，我們把它們當作計算機的樂高積木——簡單，有百萬種方式組合。

        大部分非程序員不這樣想。
        對他們來說，輸入文本文件就像爲憤怒機器人審覈員填寫稅表，如果忘記了一個分號就會遭到痛斥。

* **如果你定義了一個圖形化創作工具：**

    *  *必須實現用戶界面。*
      按鈕，點擊，拖動，諸如此類。
      有些人畏懼它，但我喜歡它。
      如果沿着這條路走下去，設計用戶界面和工作核心部分同等重要——而不是硬着頭皮完成的亂七八糟工作。

        每點額外工作都會讓工具更容易更舒適地使用，並直接導致了遊戲中更好的內容。
        如果你看看很多遊戲製作過程的內部解密，經常會發現製作有趣的創造工具是祕訣之一。

    * *有較少的錯誤情況。*
      由於用戶通過交互式一步一步地設計行爲，應用程序可以儘快引導他們走出錯誤。

        而使用基於文本的語言時，直到用戶輸完整個文件*才能*看到用戶的內容，預防和處理錯誤更加困難。

    <span name="lines"></span>

    * *更難移植。*
      文本編譯器的好處是，文本文件是通用的。編譯器簡單地讀入文件並寫出。跨平臺移植的工作實在微不足道。

        <aside name="lines">

        除了換行符。還有編碼。

        </aside>

        當你構建用戶界面，你必須選擇要使用的架構，其中很多是基於某個操作系統。
        也有跨平臺的用戶界面工具包，但他們往往要爲對所有平臺同樣適用付出代價——它們在不同的平臺上同樣差異很大。

## 參見

* 這一章節的近親是GoF的<a href="http://en.wikipedia.org/wiki/Interpreter_pattern" class="gof-pattern">解釋器模式</a>。兩種方式都能讓你用數據組合行爲。

    事實上，最終你兩種模式*都*會使用。你用來構造字節碼的工具會有內部的對象樹。這也是解釋器模式所能做的。

    爲了編譯到字節碼，你需要遞歸回溯整棵樹，就像用解釋器模式去解釋它一樣。
    *唯一的* 不同在於，不是立即執行一段行爲，而是生成整個字節碼再執行。

* [Lua](http://www.lua.org/)是遊戲中最廣泛應用的腳本語言。
  它的內部被實現爲一個非常緊湊的，基於寄存器的字節碼虛擬機。

* [Kismet](http://en.wikipedia.org/wiki/UnrealEd#Kismet)是個可視化腳本編輯工具，應用於Unreal引擎的編輯器UnrealEd。

* 我的腳本語言[Wren](https://github.com/munificent/wren)，是一個簡單的，基於棧的字節碼解釋器。

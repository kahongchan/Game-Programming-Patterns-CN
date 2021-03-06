^title 更新方法 Update Method
^section Sequencing Patterns

## 意圖

*通過每次處理一幀的行爲模擬一系列獨立對象。*

## 動機

玩家操作強大的女武神完成考驗：從死亡巫王的棲骨之處偷走華麗的珠寶。
她嘗試接近巫王華麗的地宮門口，然後遇到了……*啥也沒遇到*。
沒有詛咒雕像向她發射閃電，沒有不死戰士巡邏入口。
她直搗黃龍，拿走了珠寶。遊戲結束。你贏了。

好吧，這可不行。

<span name="brains"></span>
地宮需要守衛——一些英雄可以殺死的敵人。
首先，我們需要一個骷髏戰士在門口前後移動巡邏。
如果無視任何關於遊戲編程的知識，
讓骷髏蹣跚着來回移動的最簡單的代碼大概是這樣的：

<aside name="brains">

如果巫王想表現得更加智慧，它應創造一些仍有腦子的東西。

</aside>

^code just-patrol

這裏的問題，當然，是骷髏來回打轉，可玩家永遠看不到。
程序鎖死在一個無限循環，那可不是有趣的遊戲體驗。
我們事實上想要的是骷髏*每幀*移動一步。

<span name="game-loop1"></span>
我們得移除這些循環，依賴外層遊戲循環來迭代。
這保證了在衛士來回巡邏時，遊戲能響應玩家的輸入並進行渲染。如下：

<aside name="game-loop1">

當然，<a href="game-loop.html" class="pattern">遊戲循環</a>是本書的另一個章節。

</aside>

^code patrol-in-loop

在這裏前後兩個版本展示了代碼是如何變得複雜的。
左右巡邏需要兩個簡單的`for`循環。
通過指定哪個循環在執行，我們追蹤了骷髏在移向哪個方向。
現在我們每幀跳出到外層的遊戲循環，然後再跳回繼續我們之前所做的，我們使用`patrollingLeft`顯式地追蹤了方向。

但或多或少這能行，所以我們繼續。
一堆無腦的骨頭不會對你的女武神提出太多挑戰，
我們下一個添加的是魔法雕像。它們一直會向她發射閃電球，這樣可讓她保持移動。

繼續我們的“用最簡單的方式編碼”的風格，我們得到了：

^code statues

<span name="mush"></span>

你會發現這代碼漸漸滑向失控。
變量數目不斷增長，代碼都在遊戲循環中，每段代碼處理一個特殊的遊戲實體。
爲了同時訪問並運行它們，我們將它們的代碼混雜在了一起。

<aside name="mush">

一旦能用“混雜”一詞描述你的架構，你就有麻煩了。

</aside>

你也許已經猜到了修復這個所用的簡單模式了：
*每個遊戲實體應該封裝它自己的行爲。*
這保持了遊戲循環的整潔，便於添加和移除實體。

爲了做到這點需要*抽象層*，我們通過定義抽象的`update()`方法來完成。
遊戲循環管理對象的集合，但是不知道對象的具體類型。
它只知道這些對象可以被更新。
這樣，每個對象的行爲與遊戲循環分離，與其他對象分離。

<span name="simultaneously"></span>

每一幀，遊戲循環遍歷集合，在每個對象上調用`update()`。
這給了我們在每幀上更新一次行爲的機會。
在所有對象上每幀調用它，對象就能同時行動。

<aside name="simultaneously">

死摳細節的人會在這點上揪着我不放，是的，它們沒有*真的同步*。
當一個對象更新時，其他的都不在更新中。
我們等會兒再說這點。

</aside>

遊戲循環維護動態的對象集合，所以從關卡添加和移除對象是很容易的——只需要將它們從集合中添加和移除。
不必再用硬編碼，我們甚至可以用數據文件構成這個關卡，那正是我們的關卡設計者需要的。

## 模式

**遊戲世界**管理**對象集合**。
每個對象實現一個**更新方法**模擬對象在**一幀**內的行爲。每一幀，遊戲循環更新集合中的每一個對象。

## 何時使用

如果<a href="game-loop.html" class="pattern">遊戲循環</a>模式是切片面包，
那麼更新方法模式就是它的奶油。
很多玩家交互的遊戲實體都以這樣或那樣的方式實現了這個模式。
如果遊戲有太空陸戰隊，火龍，火星人，鬼魂或者運動員，很有可能它使用了這個模式。

<span name="pawn"></span>
但是如果遊戲更加抽象，移動部分不太像活動的角色而更加像棋盤上的棋子，
這個模式通常就不適用了。
在棋類遊戲中，你不需要同時模擬所有的部分，
你可能也不需要告訴棋子每幀都更新它們自己。

<aside name="pawn">

你也許不需要每幀更新它們的*行爲*，但即使是棋類遊戲，
你可能也需要每幀更新*動畫*。
這個設計模式也可以幫到你。

</aside>

更新方法適應以下情況：

* 你的遊戲有很多對象或系統需要同時運行。

* 每個對象的行爲都與其他的大部分獨立。

* 對象需要跟着時間進行模擬。

## 記住

這個模式很簡單，所以沒有太多值得發現的驚喜。當然，每行代碼還是有利有弊。

## 將代碼劃分到一幀幀中會讓它更復雜

當你比較前面兩塊代碼時，第二塊看上去更加複雜。
兩者都只是讓骷髏守衛來回移動，但與此同時，第二塊代碼將控制權交給了遊戲循環的一幀幀中。

<span name="coroutine">幾乎</span>
這個改變是遊戲循環處理用戶輸入，渲染等幾乎必須要注意的事項，所以第一個例子不大實用。
但是很有必要記住，將你的行爲切片會增加很高的複雜性。

<aside name="coroutine">

我在這裏說幾乎是因爲有時候魚和熊掌可以兼得。
你可以直接爲對象編碼而不進行返回，
保持很多對象同時運行並與遊戲循環保持協調。

你需要的是允許你同時擁有多個“線程”執行的系統。
如果對象的代碼可以在執行中暫停和繼續，而不是總得*返回*，
你可以用更加命令式的方式編碼。

真實的線程太過重量級而不能這麼做，
但如果你的語言支持輕量協同架構比如generators，coroutines或者fibers，那你也許可以使用它們。

<a href="bytecode.html" class="pattern">字節碼</a>模式是另一個在應用層創建多個線程執行的方法。

</aside>

### 當離開每幀時，你需要存儲狀態，以備將來繼續。

在第一個示例代碼中，我們不需要用任何變量表明守衛在向左還是向右移動。
這顯式的依賴於哪塊代碼正在運行。

當我們將其變爲一次一幀的形式，我們需要創建`patrollingLeft`變量來追蹤行走的方向。
當從代碼中返回時，就丟失了行走的方向，所以爲了下幀繼續，我們需要顯式存儲足夠的信息。

<a href="state.html" class="pattern">狀態模式</a>通常可以在這裏幫忙。
狀態機在遊戲中頻繁出現的部分原因是（就像名字暗示的），它能在你離開時爲你存儲各種你需要的狀態。

### 對象逐幀模擬，但並非真的同步

在這個模式中，遊戲遍歷對象集合，更新每一個對象。
在`update()`調用中，大多數對象都能夠接觸到遊戲世界的其他部分，
包括現在正在更新的其他對象。這就意味着你更新對象的*順序*至關重要。

<span name="double-buffer"></span>

如果對象更新列表中，A在B之前，當A更新時，它會看到B之前的狀態。
但是當B更新時，由於A已經在這幀更新了，它會看見A的*新*狀態。
哪怕按照玩家的視角，所有對象都是同時運轉的，遊戲的核心還是回合制的。
只是完整的“回合”只有一幀那麼長。

<aside name="double-buffer">

如果，由於某些原因，你決定*不*讓遊戲按這樣的順序更新，你需要<a href="double-buffer.html" class="pattern">雙緩衝</a>模式。
那麼AB更新的順序就沒有關係了，因爲*雙方*都會看對方之前那幀的狀態。

</aside>

當關注遊戲邏輯時，這通常是件好事。
同時更新所有對象將把你帶到一些不愉快的語義角落。
想象如果國際象棋中，黑白雙方同時移動會發生什麼。
雙方都試圖同時往同一個空格子中放置棋子。這怎麼解決？

<span name="sequential"></span>

序列更新解決了這點——每次更新都讓遊戲世界從一個合法狀態增量更新到下一個，不會出現引發歧義而需要協調的部分。

<aside name="sequential">

這對在線遊戲也有用，因爲你有了可以在網上發送的行動指令序列。

</aside>

### 在更新時修改對象列表需小心

當你使用這個模式時，很多遊戲行爲在更新方法中糾纏在一起。
這些行爲通常包括增加和刪除可更新對象。

舉個例子，假設骷髏守衛被殺死時掉落物品。
使用新對象，你通常可以將其增加到列表尾部，而不引起任何問題。
你會繼續遍歷這張鏈表，最終找到新的那個，然後也更新了它。

但這確實表明新對象在它產生的那幀就有機會活動，甚至有可能在玩家看到它之前。
如果你不想發生那種情況，簡單的修復方法就是在遊戲循環中緩存列表對象的數目，然後只更新那麼多數目的對象就停止：

^code skip-added

這裏，`objects_`是可更新遊戲對象的數組，而`numObjects_`是數組的長度。
當添加新對象時，這個數組長度變量就增加。
在循環的一開始，我們在`numObjectsThisTurn`中存儲數組的長度，
這樣這幀的遍歷循環會停在新添加的對象之前。

一個更麻煩的問題是在遍歷時*移除*對象。
你擊敗了邪惡的野獸，現在它需要被移出對象列表。
如果它正好位於你當前更新對象之前，你會意外地跳過一個對象：

^code skip-removed

這個簡單的循環通過增加索引值來遍歷每個對象。
下圖的左側展示了在我們更新英雄時，數組看上去是什麼樣的：

<img src="images/update-method-remove.png" alt="在一次移除中的對象實體列表。一個指針指向第二個實體，英雄。排在第一個的怪獸被移除後，英雄向上移動一格，與此同時，指針向下移動一格。" />

<span name="backwards"></span>

我們在更新她時，索引值`i`是1。
邪惡野獸被她殺了，因此需要從數組移除。
英雄移到了位置0，倒黴的鄉下人移到了位置1。
在更新英雄之後，`i`增加到了2。
就像你在右圖看到的，倒黴的鄉下人被跳過了，沒有更新。

<aside name="backwards">

一種簡單的解決方案是在更新時*從後往前*遍歷列表。
這種方式只會移動已經被更新的對象。

</aside>

<span name="defer"></span>

一種解決方案是小心地移除對象，任何對象被移除時，更新索引。
另一種是在遍歷完列表後再移除對象。
將對象標爲“死亡”，但是把它放在那裏。
在更新時跳過任何死亡的對象。然後，在完成遍歷後，遍歷列表並刪除屍體。

<aside name="defer">

如果在更新循環中有多個線程處理對象，
那麼你可能更喜歡推遲任何修改，避免更新時同步線程的開銷。

</aside>

## 示例代碼

這個模式太直觀了，代碼幾乎只是在重複說明要點。
這不意味着這個模式沒*有用*。它*因爲*簡單而有用：這是一個無需裝飾的乾淨解決方案。

但是爲了讓事情更具體些，讓我們看看一個基礎的實現。
我們會從代表骷髏和雕像的`Entity`類開始：

^code entity-class

我在這裏只呈現了我們後面所需東西的最小集合。
可以推斷在真實代碼中，會有很多圖形和物理這樣的其他東西。
上面這部分代碼最重要的部分是它有抽象的`update()`方法。

遊戲管理實體的集合。在我們的示例中，我會把它放在一個代表遊戲世界的類中。

<span name="array"></span>

^code game-world

<aside name="array">

在真實的世界程序中，你可能真的要使用集合類，我在這裏使用數組來保持簡單

</aside>

現在，萬事俱備，遊戲通過每幀更新每個實體來實現模式：

<span name="game-loop"></span>

^code game-loop

<aside name="game-loop">

正如其名，這是<a href="game-loop.html" class="pattern">遊戲循環</a>模式的一個例子。

</aside>

### 子類化實體？！

有很多讀者剛剛起了雞皮疙瘩，因爲我在`Entity`主類中使用繼承來定義不同的行爲。
如果你在這裏還沒有看出問題，我會提供一些線索。

當遊戲業界從6502彙編代碼和VBLANKs轉向面向對象的語言時，
開發者陷入了對軟件架構的狂熱之中。
其中之一就是使用繼承。他們建立了遮天蔽日的高聳的拜占庭式對象層次。

<span name="subclass">

最終證明這是個糟點子，沒人可以不拆解它們來管理龐雜的對象層次。
哪怕在1994年的GoF都知道這點，並寫道：

> 多用“對象組合”，而非“類繼承”。

<aside name="subclass">

只在你我間聊聊，我認爲這已經是一朝被蛇咬十年怕井繩了。
我通常避免使用它，但教條地不用和教條地使用一樣糟。
你可以適度使用，不必完全禁用。

</aside>

當遊戲業界都明白了這一點，解決方案是使用<a href="component.html" class="pattern">組件</a>模式。
使用它，`update()`是實體的*組件*而不是在`Entity`中。
這讓你避開了爲了定義和重用行爲而創建實體所需的複雜類繼承層次。相反，你只需混合和組裝組件。

<span name="chapter"></span>

如果我真正在做遊戲，我也許也會那麼做。
但是這章不是關於組件的，
而是關於`update()`方法，最簡單，最少牽連其他部分的介紹方法，
就是把更新方法放在`Entity`中然後創建一些子類。

<aside name="chapter">

組件模式在<a href="component.html" class="pattern">這裏</a>。

</aside>

### 定義實體

好了，回到任務中。
我們原先的動機是定義巡邏的骷髏守衛和釋放閃電的魔法雕像。
讓我們從我們的骷髏朋友開始吧。
爲了定義它的巡邏行爲，我們定義恰當地實現了`update()`的新實體：

^code skeleton

如你所見，幾乎就是從早先的遊戲循環中剪切代碼，然後粘貼到`Skeleton`的`update()`方法中。
唯一的小小不同是`patrollingLeft_`被定義爲字段而不是本地變量。
通過這種方式，它的值在`update()`兩次調用間保持不變。

讓我們對雕像如法炮製：

^code statue

又一次，大部分改動是將代碼從遊戲循環中移動到類中，然後重命名一些東西。
但是，在這個例子中，我們真的讓代碼庫變簡單了。
先前討厭的命令式代碼中，存在存儲每個雕像的幀計數器和開火的速率的分散的本地變量。

現在那些都被移動到了`Statue`類中，你可以想創建多少就創建多少實例了，
每個實例都有它自己的小計時器。
這是這章背後的真實動機——現在爲遊戲世界增加新實體會更加簡單，
因爲每個實體都帶來了它需要的全部東西。

這個模式讓我們分離了遊戲世界的*構建*和*實現*。
這同樣能讓我們靈活地使用分散的數據文件或關卡編輯器來構建遊戲世界。

<span name="uml"></span>

<img src="images/update-method-uml.png" alt="一個UML圖。世界有一系列實體組成，每實體都有update()方法。可鏤衛士和魔法雕像都繼承實體。" />

<aside name="uml">

還有人關心UML嗎？如果還有，那就是我們剛剛建的。

</aside>

### 傳遞時間

這是模式的關鍵，但是我只對常用的部分進行了細化。
到目前爲止，我們假設每次對`update()`的調用都推動遊戲世界前進一個固定的時間。

<span name="variable"></span>

我更喜歡那樣，但是很多遊戲使用*可變時間步長*。
在那種情況下，每次遊戲循環推進的時間長度或長或短，
具體取決於它需要多長時間處理和渲染前一幀。

<aside name="variable">

<a href="game-loop.html" class="pattern">遊戲循環</a>一章討論了更多關於固定和可變時間步長的優劣。

</aside>

這意味着每次`update()`調用都需要知道虛擬的時鐘轉動了多少，
所以你經常可以看到傳入消逝的時間。
舉個例子，我們可以讓骷髏衛士像這樣處理變化的時間步長：

^code variable

現在，骷髏衛士移動的距離隨着消逝時間的增長而增長。
也可以看出，處理變化時間步長需要的額外複雜度。
如果一次需要更新的時間步長過長，骷髏衛士也許就超過了其巡邏的範圍，因此需要小心的處理。

## 設計決策

在這樣簡單的模式中，沒有太多的調控之處，但是這裏仍有兩個你需要決策的地方：

### 更新方法在哪個類中？

最明顯和最重要的決策就是決定將`update()`放在哪個類中。

* **實體類中：**

    如果你已經有實體類了，這是最簡單的選項，
    因爲這不會帶來額外的類。如果你需要的實體種類不多，這也許可行，但是業界已經逐漸遠離這種做法了。

    當類的種類很多時，一有新行爲就建`Entity`子類來實現是痛苦的。
    當你最終發現你想要用單一繼承的方法重用代碼時，你就卡住了。

* **組件類：**

    如果你已經使用了<a href="component.html" class="pattern">組件</a>模式，你知道這個該怎麼做。
    這讓每個組件獨立更新它自己。
    更新方法用了同樣的方法解耦遊戲中的實體，組件讓你進一步解耦了*單一實體中的各部分*。
    渲染，物理，AI都可以自顧自了。

* **委託類：**

    還可將類的部分行爲委託給其他的對象。
    <a href="state.html" class="pattern">狀態</a>模式可以這樣做，你可以通過改變它委託的對象來改變它的行爲。
    <a href="type-object.html" class="pattern">類型對象</a>模式也這樣做了，這樣你可以在同“種”實體間分享行爲。

    如果你使用了這些模式，將`update()`放在委託類中是很自然的。
    在那種情況下，也許主類中仍有`update()`方法，但是它不是虛方法，可以簡單地委託給委託對象。就像這樣：

    ^code forward

    這樣做允許你改變委託對象來定義新行爲。就像使用組件，這給了你無須定義全新的子類就能改變行爲的靈活性。

### 如何處理隱藏對象？

遊戲中的對象，不管什麼原因，可能暫時無需更新。
它們可能是停用了，或者超出了屏幕，或者還沒有解鎖。
如果狀態中的這種對象很多，每幀遍歷它們卻什麼都不做是在浪費CPU循環。

一種方法是管理單獨的“活動”對象集合，它存儲真正需要更新的對象。
當一個對象停用時，從那個集合中移除它。當它啓用時，再把它添加回來。
用這種方式，你只需要迭代那些真正需要更新的東西：

* **如果你使用單個包括了所有不活躍對象的集合：**

    <span name="cache"></span>

    * *浪費時間*。對於不活躍對象，你要麼檢查一些“是否啓用”的標識，要麼調用一些啥都不做的方法。

    <aside name="cache">

    檢查對象啓用與否然後跳過它，不但消耗了CPU循環，還報銷了你的數據緩存。
    CPU通過從RAM上讀取數據到緩存上來優化讀取。
    這樣做是基於剛剛讀取內存之後的內存部分很可能等會兒也會被讀取到這個假設。

    當你跳過對象，你可能越過了緩存的尾部，強迫它從緩慢的主存中再取一塊。

    </aside>

* **如果你使用單獨的集合保存活動對象：**

    * *使用了額外的內存管理第二個集合。*
      當你需要所有實體時，通常又需要一個巨大的集合。在那種情況下，這集合是多餘的。
      在速度比內存要求更高的時候（通常如此），這取捨仍是值得的。

    另一個權衡後的選擇是使用兩個集合，除了活動對象集合的另一個集合只包含*不活躍*實體而不是全部實體。

     * *得保持集合同步。*
        當對象創建或完全銷燬時（不是暫時停用），你得修改全部對象集合和活躍對象集合。

方法選擇的度量標準是不活躍對象的可能數量。
數量越多，用分離的集合避免在覈心遊戲循環中用到它們就更有用。

## 參見

* 這個模式，以及<a href="game-loop.html" class="pattern">遊戲循環</a>模式和<a href="component.html" class="pattern">組件</a>模式，是構建遊戲引擎核心的三位一體。

* 當你關注在每幀中更新實體或組件的緩存性能時，<a href="data-locality.html" class="pattern">數據局部性</a>模式可以讓它跑到更快。

* [Unity](http://unity3d.com)框架在多個類中使用了這個模式，包括
  [`MonoBehaviour`](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)。

* 微軟的[XNA](http://creators.xna.com/en-US/)平臺在
  [`Game`](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.game.update.aspx)
  和
  [`GameComponent`](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.gamecomponent.update.aspx)
  類中使用了這個模式。

* [Quintus](http://html5quintus.com/)，一個JavaScript遊戲引擎在它的主[`Sprite`](http://html5quintus.com/guide/sprites.md)類中使用了這個模式。

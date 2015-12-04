^title Sequencing Patterns

电子游戏之所有有趣，很大程度上归功于它们会将我们带到别的地方。
几分钟后（或者，诚实点，可能会更长），我们栖息在一个虚拟的世界。
创造那样一个世界是游戏程序员至上的欢愉。

大多数游戏世界都有的特性是*时间*——虚构世界以其特定的昼夜生活。
作为世界的架构师，我们必须发明时间，制造推动我们游戏时间运作的齿轮。

这一部分的模式是建构这些方面的工具。
[游戏循环](game-loop.html)是时钟的中间轴。
对象通过[更新模式](update-method.html)来听时钟的滴答声。
我们可以用[双重缓存](double-buffer.html)存储快照来隐藏计算机的顺序执行，这样看起来世界是同步运行的。

## 模式

* [双缓冲](double-buffer.html)
* [游戏循环](game-loop.html)
* [更新方法](update-method.html)
# A Philosophy of Software Design 读书笔记

## 作者介绍

[John Ousterhout](https://web.stanford.edu/~ouster/cgi-bin/home.php), Professor of Computer Science at Stanford University

你也许在 RAMCloud, Raft, Tcl, [Why Threads Are A Bad Idea (for most purposes)](https://web.stanford.edu/~ouster/cgi-bin/papers/threads.pdf) 相关的话题中看到过这个名字

## 书籍介绍

你认为计算机科学领域中最重要的东西是什么？

* abstraction?
    
* testing?
    
* composition?
    
* complexity?
    
* layers of abstraction( [by Don Knuth](https://youtu.be/bmSAYlu0NcY?t=215) )?
    

作者的答案是 **problem decomposition**

> The most fundamental problem in computer science is problem decomposition: how to take a complex problem and divide it up into pieces that can be solved independently

既然 problem decomposition 是如此重要，但是学校里教了 `for` 循环和 object-oriented programming, 却没有专门的课程来教软件设计 (Software Design)。多年来作者一直为此感到困惑和沮丧。

最终作者决定自己开设一门课程 (CS 190, Stanford) 来教学软件设计。教学的方式是：

1. 学生写代码
    
2. 作者进行 code review
    
3. 学生进行改进
    

这本书便是作者基于几次的教学经验写成的，当然作者本身也有充足的软件开发经验，参与过多个领域的软件的设计与开发，有足够的资历在哲学层面上谈论软件设计。

相比起 *The Pragmatic Programmer*, 以及 Bob 大叔的 *Clean Code* 系列，本书提及的设计原则 (Design Principle) 比较抽象，比如

* *Define Errors Out Of Existence*
    
* *Complexity is incremental: you have to sweat the small stuff*
    
* *Modules should be deep*
    

好在作者给出了足够的例子来论证自己的观点。

如果你打算看这本书，建议先花一个小时看看作者以本书为主题的 [talk](https://www.youtube.com/watch?v=bmSAYlu0NcY)。另外，这本书只有 200 页，短小精悍，非常值得一读。

## Complexity

> This book is about one thing: complexity. Dealing with complexity is the most important challenge in software design.

Complexity is anything related to the structure of a software system that makes it hard to understand and modify the system.

![complexity.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1675016322058/b4562f0e-6b9c-4dca-8e4b-54ea25d7c54f.png align="center")

* Change amplification: a seemingly simple change requires code modifications in many places.
    
* Cognitive load: in order to make a change, the developer must accumulate a large amount of information.
    
* Unknown unknowns: it is unclear what code needs to be modified, or what information must be considered in order to make those modifications.
    

## 令我印象深刻的设计原则/哲学

### Working Code Isn’t Enough

> If you’re not making the design better, you are probably making it worse.

* **Tactical programming**: 把快速完成当前的任务作为主要目标，比如实现需求，修复 bug。
    
* **Strategic programming**: 相比起完成当前的任务，更重视设计。
    

Tactical programming 是短视的，忽视设计使得系统的复杂性陡增，很快我们就会注意到复杂性带来的问题，这会使得我们把从设计中节省的时间加倍奉还。我们应该把 Strategic programming 视为一种投资 (investment mindset)，很多时候这个投资的回报来得比预期的更快。

### Zero tolerance

> Complexity is incremental

为了控制复杂性，必须采取零容忍 (zero tolerance) 哲学，警惕 [破窗效应](https://zh.wikipedia.org/zh-hans/%E7%A0%B4%E7%AA%97%E6%95%88%E5%BA%94) 与 [Slippery slope](https://en.wikipedia.org/wiki/Slippery_slope)

正如作者在 *Working Code Isn’t Enough(Strategic vs. Tactical Programming)* 这一章总结的

> It’s crucial to be consistent in applying the strategic approach and to think of investment as something to do today, not tomorrow. When you get in a crunch it will be tempting to put off cleanups until after the crunch is over. However, this is a slippery slope; after the current crunch there will almost certainly be another one, and another after that. Once you start delaying design improvements, it’s easy for the delays to become permanent and for your culture to slip into the tactical approach. The longer you wait to address design problems, the bigger they become; the solutions become more intimidating, which makes it easy to put them off even more. The most effective approach is one where every engineer makes continuous small investments in good design.

### Define errors out of existence

> Throwing exceptions is easy; handling them is hard. Thus, the complexity of exceptions comes from the exception handling code.

这里的异常泛指不常见的情况，比如

* 方法被传入了非法的参数
    
* 调用方法不一定会成功，比如遇到了 IO 异常，或者需要的资源不可用
    
* 分布式系统中的丢包，超时，乱序问题
    

抛出异常是简单的，处理异常是困难的。在没有尽自己最大努力之前就把处理异常这个脏活丢给调用方是不负责任的表现。

减少处理异常对系统复杂性的的影响的最好办法是**减少必须处理异常的地方** (reduce the number of places where exceptions have to be handled)。具体的方法有

* 不产生异常 (Define errors out of existence)
    
* 隐瞒异常 (Mask exceptions)
    
* 聚集异常 (Exception aggregation)
    
* Just crash?
    

### Modules Should Be Deep

> The best modules are those that provide powerful functionality yet have simple interfaces.

![Modules-Should-Be-Deep.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1675083728308/1a211913-4044-41eb-bdf5-4fc68170aac7.png align="center")

在上图中，矩形的面积与模块的功能成正比。矩形的顶部表示模块的接口，其长度与模块的复杂性成正比。当为系统实现同样多的功能时，deep module 与 shallow module 给系统的复杂性造成的影响不同。

> For the purposes of this book, a module is any unit of code that has an interface and an implementation.

**Deep modules examples: Unix I/O interface**

```c
int open(const char* path, int flags, mode_t permissions);
ssize_t read(int fd, void* buffer, size_t count);
ssize_t write(int fd, const void* buffer, size_t count);
off_t lseek(int fd, off_t offset, int referencePosition);
int close(int fd);
```

**Deep modules examples:** Java 和 Go 的垃圾回收器甚至不需要接口

**shallow modules** **examples:** Java 社区有 *classes should be small* 的文化，作者将这种现象称为 *Classitis* 。下面是用 Java 读取文件中的序列化后的对象的例子。为了创建 `objectStream` 对象，不得不手动创建 `fileStream`, `bufferedStream`.

```java
FileInputStream fileStream = new FileInputStream(fileName);
BufferedInputStream bufferedStream = new BufferedInputStream(fileStream);
ObjectInputStream objectStream = new ObjectInputStream(bufferedStream);
```

**shallow modules examples: CS 190 中的 project 代码**。该模块的实现完全被接口暴露出来了，接口没有起到抽象的作用，反而增加了认知负担。

```java
private void addNullValueForAttribute(String attribute) {
    data.put(attribute, null);
}
```

## 一些有意思的观点

### tactical tornado

指代将 tactical programming 发挥到极致的人，其特点是写起代码来糙快猛，乍一看上去像是 10x 程序员。

tactical tornado 把代码弄得一团糟，其他人要为他收拾烂摊子，降低了其他人的工作效率，从而导致团队整体的效率下降，但是对比之下自己的效率显得更高了。

有些不明真相的管理人员可能将 tactical tornado 视为团队的英雄。

### Facebook vs Google

Facebook 是 tactical approach 的代表，而 Google 是 strategic approach 的代表。从结果来看，无疑这两家公司都取得了巨大的成功。

strategic 与 tactical 不是公司是否成功的决定性因素，但是它们能决定我们的工作体验。毕竟，在一家重视软件设计，代码整洁的公司工作更有趣。

Facebook 的格言从 *Move fast and break things* 改为 *Move fast with solid infrastructure*，看上去是在为 tactical programming 还技术债。

### 注释 (Comments)

对不写注释的四个借口分别进行了驳斥。

#### Good code is self-documenting

注释作为接口抽象的一部分，其目的是为了让使用者不用关心接口实现。接口方法的声明（包括参数名，参数类型，返回值类型）并不足以建立接口抽象。

#### I don’t have time to write comments

写注释的时间占比不会超过 10%

> Ask yourself how much of your development time you spend typing in code (as opposed to designing, compiling, testing, etc.), assuming you don’t include any comments; I doubt that the answer is more than 10%. Now suppose that you spend as much time typing comments as typing code; this should be a safe upper bound. With these assumptions, writing good comments won’t add more than about 10% to your development time. The benefits of having good documentation will quickly offset this cost.

#### Comments get out of date and become misleading

一般只有代码变化大时注释变化才会大，无论如何，相比起改代码，改注释工作量不大。

#### The comments I have seen are all worthless; why bother?

这是事实，但是基于前面的讨论，我们认为注释是必不可少的，这应该作为你写好注释的理由，而不是不写注释的理由。

## 我的感受

作者是学术大牛，这本书也写得很好。我认同本书的绝大部分观点。

作者主要是在学术界工作，研究领域偏向 system，这些设计原则是成立的。但是在工业界未必成立，比如

* 需求还没有实现就被产品经理改了，导致前期的设计时间被浪费了。这时 Working Code Isn’t Enough 原则不成立
    
* 假设 A 部门有一个供外部使用的模块 (module-a)，其他部门的模块 (module-b, module-c, etc.) 依赖了 module-a。module-a 没有提供好的抽象，导致 module-b, module-c 开发困难。但是 module-a 比较稳定，很少需要改动代码，这时 module-a 欠下的技术债要由其他模块 (module-b, module-c, etc.) 的开发人员进行偿还。对于 module-a 的开发人员，很多设计原则都不成立。
    

最后引用作者的总结作为本文的总结

> The reward for being a good designer is that you get to spend a larger fraction of your time in the design phase, which is fun. Poor designers spend most of their time chasing bugs in complicated and brittle code. If you improve your design skills, not only will you produce higher quality software more quickly, but the software development process will be more enjoyable.
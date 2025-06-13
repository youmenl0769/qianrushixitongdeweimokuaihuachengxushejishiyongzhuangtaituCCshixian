# 嵌入式系统的微模块化程序设计－实用状态图C/C++实现

## 资源描述

状态模式是GoF23个模式中最常用的之一，这篇小文不打算涉及方方面面的内容，只想在状态模式的高效运用方面谈一下自己的心得体会。

状态模式是用来设计状态机的，因此下面的叙述中将它们等同理解。有关状态机设计方面的书籍，我这里隆重推荐一本：《Practical Statecharts in C/C++ Quantum Programming for Embedded Systems》，中文名叫做《嵌入式系统的微模块化程序设计－实用状态图C/C++实现》，北航出版的，作者是Miro Samek博士，长期从事嵌入式实时系统的开发，具有丰富的经验。如果你想对状态机领域进行比较深入的研究，这本书绝对不容错过。

让我们先来看看比较“古老”的状态机实现，假设你还是用C语言。一般而言，我们用得到状态机系统都可以称为事件（消息）驱动系统，系统往往处于某个状态，等待外部的激励。这些激励可以是外部的事件、定时器超时等等，系统收到这些事件后，进行相应的处理，然后跃迁到新的状态（状态也可能不变）继续等待下一个激励的到来，最后直到相应的事务处理完毕为止。

典型的状态机实现中需要考虑几个要素：状态、消息（及其内容）、消息处理函数以及系统上下文等。系统处于某个状态，收到某个消息后，解析出消息内容，然后调用相应的消息处理函数进行处理，而消息处理函数往往会用到状态机的上下文数据，消息处理完毕系统会跃迁到新的状态。

典型代码大致如下：

```c
switch (state)
{
    case STATE1:
        switch (msg)
        {
            case MSG1:
                HandleMsg1(msgparacontext);
                nextstate(STATE2);
                break;
            case MSG2:
                HandleMsg2(msgparacontext);
                nextstate(STATE3);
                break;
            /*......*/
        }
    case STATE2:
        switch (msg)
        {
            case MSG3:
                HandleMsg3(msgparacontext);
                nextstate(STATE3);
                break;
            /*......*/
        }
    /*......*/
}
```

可以看到这就是所谓的平面状态机，特点就是先枚举状态，然后再枚举消息，如果找不到的话，就将消息丢弃。

为了使状态机更高效的运行，这里有几个小技巧，稍为总结一下。

（1）把接收概率大的消息放在前面

把同一个状态下最有可能收到的消息放在前面。一个状态下可能要处理很多消息，这视乎你状态划分的粒度大小。每个消息收到的机会并不是均等的，有些消息系统收到的概率很大，有些很小，因此把接收概率大的消息放在前面，这样可以减少case消息时的比较次数，相应的执行效率就提高了。对于一个状态机的运行而言，这样的节省当然微乎其微，但假如你的系统同时运行成千上万个这种状态机时，那么就有必要考虑一下这种优化了。

（2）查表法

第（1）种方法再怎么优化，也需要枚举状态和消息，假如能把这方面的开销变成零，那么效率自然可以进一步提升。我们可以想象把消息处理函数指针放在一个二维数组（表）中，其中一维代表状态，另外一维代表消息序号，那么通过p[state][msg]就可以定位到当前状态下当前消息的处理函数。对一些简单的应用，甚至可以把新状态也存放在这张二维表中，这样的好处是用户不需要显示调用状态跃迁函数。当然对于一些状态有不同执行路径的情况，状态的跃迁可能就要放在消息处理函数之中。

（3）消息先分段再查表

一般而言，一个状态机的状态数目不会很多，当然接收的消息数目也是有限的。但一般来说，消息是不连续的，这样应用查表法可能内存的开销就比较大，尤其是消息序号比较稀疏的时候，内存更加浪费。

在一般的嵌入式软件开发中，我发现往往可以将消息进行归类分段，比方说一个接口的消息定义成一段。这样虽然消息不连续，但通过分段后可以将消息放在一个较紧凑的内存空间中，在这个空间里再运用查表法，就有可能达到效率和空间开销的平衡。注意，我是说有可能，并不是一定，这取决于具体情况。系统收到消息后，先判断消息处于哪个分段，然后调用p[state][msg - offset]来进行处理。

## 资源下载

本仓库提供《嵌入式系统的微模块化程序设计－实用状态图C/C++实现》的资源文件下载，欢迎有兴趣的开发者下载学习。

## 下载链接
[嵌入式系统的微模块化程序设计实用状态图CC实现](https://pan.quark.cn/s/6a5517646ac3) 

(备用: [备用下载](https://pan.baidu.com/s/1uU8FpxGoQ7JCYIee-PhbWQ?pwd=1234))

## 说明

该仓库仅用于学习交流，请勿用于商业用途。

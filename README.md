# flink 文档整理
本人只是收集了 这些文档，并不是翻译者
文档中有些作者翻译的时间可能较早版本较早，有些内容缺失或者被标记为废弃

**注意**请英文中文 对比阅读，否则翻译错误导致您理解错误，我可负不了责。。

本人只是总结了 文档，如果有侵权，联系我，马上删除，感谢下面所有提供链接的人！~！~

参考：
1. https://github.com/apachecn/flink-doc-zh
2. https://www.jianshu.com/u/12d5c7b605d1
3. https://www.jianshu.com/u/e7c0da812dd9

目录：

1. Application Development
2. Basic Api Concepts
    1. [Overview](./BasicApIConcepts/Overview.md)
3. Streaming(DataStream API)
    1. [Overview](https://www.jianshu.com/p/ea80d15e9b5e)
    2. Event Time
        1. [Overview](https://www.jianshu.com/p/68ab40c7f347)
        2. [Generating TimeStamps](https://www.jianshu.com/p/8c4a1861e49f)
        3. [Pre-defined Timestamp Extractors / Watermark Emitters](https://www.jianshu.com/p/00bb57459ef4)
    3. State & Fault Tolerance
        1. [Overview](https://www.jianshu.com/p/9a53c1792c4a)
        2. [Working width State](https://www.jianshu.com/p/14c6f0e70efc)
        3. [The BroadCast State Pattern](https://www.jianshu.com/p/e475504480f9)
        4. [Checkpointing](https://www.jianshu.com/p/e475504480f9)
        5. [Queryable State]()
        6. [State Backends](https://www.jianshu.com/p/9fac80afff2c)
        7. [State Schema Evolution]()
        8. [Custom Serialization for Managed State](https://flink-china.org/doc/dev/stream/state/custom_serialization.html)
    4. Operators
        1. [Overview](https://www.jianshu.com/p/ea80d15e9b5e)
        1. [Overview](https://www.jianshu.com/p/f13e860cafbe)
        2. [windows 1.3版本缺失和淘汰了一些内容](https://www.jianshu.com/p/a883262241ef)
        3. [Joining]()
        4. [Process Function](https://www.jianshu.com/p/fbcb7cf7f863)
4. Batch(DataStream API)
    1. [Overview]()
    2. [Concepts & Common API](https://www.jianshu.com/p/6bc5b4e6f163)
    3. Streaming Concepts
        1. [Overview](./TableApi&SQL/StreamingConcepts/Overview.md)
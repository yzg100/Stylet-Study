It's hard to compare the performance of different IoC containers. Added complexity usually comes at a performance cost, so the more featureful contains are almost always going to be slower.

However, the numbers are quite interesting to look at, so here's Munq's benchmark, modified to add StyletIoC.

---
><font color="#63aebb" face="微软雅黑">很难比较不同IoC容器的性能。增加的复杂性通常是以性能为代价的，因此包含更多特性的内容几乎总是会比较慢。

>但是，这些数字非常有趣，所以这里是Munq的基准测试，经过修改后添加了StyletIoC。</font>

```
Running 500000 iterations for each use case. - 为每个用例运行500000次迭代。
                          Test:        Ticks -         mSec -   Normalized
              No IOC Container:      153,169 -        92.47 -         1.00
                     StyletIoC:      418,840 -       252.86 -         2.73
                          Munq:    3,009,272 -     1,816.74 -        19.65
                         Unity:   48,533,016 -    29,300.02 -       316.86
                       Autofac:   69,065,954 -    41,696.02 -       450.91
                  StructureMap:   15,356,998 -     9,271.22 -       100.26
                      Ninject2:  269,115,738 -   162,468.69 -     1,756.99
                       Windsor:  191,559,858 -   115,647.19 -     1,250.64
              StyletIoCFactory:    3,082,670 -     1,861.05 -        20.13
                   MunqFactory:    2,777,705 -     1,676.94 -        18.13
                AutofacFactory:   18,526,411 -    11,184.64 -       120.95
           StructureMapFactory:   15,708,059 -     9,483.16 -       102.55
               Ninject2Factory:  197,354,786 -   119,145.67 -     1,288.48
    No IOC Container Singleton:       27,382 -        16.53 -         0.18
            StyletIoCSingleton:      183,638 -       110.86 -         1.20
                 MunqSingleton:      411,133 -       248.21 -         2.68
                UnitySingleton:    8,048,317 -     4,858.87 -        52.55
              AutofacSingleton:    3,205,066 -     1,934.94 -        20.93
         StructureMapSingleton:    5,007,675 -     3,023.20 -        32.69
             Ninject2Singleton:   30,795,175 -    18,591.45 -       201.05
              WindsorSingleton:    2,379,411 -     1,436.48 -        15.53
                          Hiro:      252,305 -       152.32 -         1.65
```

The entries at the top, with no suffix, are there the container's responsible for figuring out how to instantiate the type - e.g. `builder.Bind<ISomeType>().To<SomeType>()`. The entries ending with 'Factory' are where the bindings were created using e.g. `builder.Bind<ISomeType>().To(container => new SomeType(container.Get<ISomeDependency>()))`, while those ending with 'Singleton' are where the container was configured to always return the same instance for each type.

An interesting observation is that StyletIoC isn't *much* slower than simply instantiating the type - in fact, StyletIoC actually instantiates the type as quickly as native code, it's figuring out which type to instantiate which adds the extra.

Another is that, in the Factory and Singleton cases, StyletIoC is pretty much the same speed as Munq. When it comes to the container being responsible for figuring out how to instantiate an instance of the type itself, however, StyletIoC is quicker by more than an order of magnitude. This is thanks for some clever use of C#'s expressions, covered in [StyletIoC Technical](./StyletIoC-Technical.md).

---
><font color="#63aebb" face="微软雅黑">顶部的记录，没有后缀，容器确定如何实例化类型 - 例如：`builder.Bind<ISomeType>().To<SomeType>()`。以 “Factory” 结尾的记录是使用例如 ：`builder.Bind<ISomeType>().To(container => new SomeType(container.Get<ISomeDependency>()))`,，而那些以“Singleton”结尾的地方，容器配置总是为每种类型返回相同的实例。

>一个有趣的现象是，StyletIoC 并不比简单地实例化类型慢多少，StyletIoC 实例化类型的速度与本机代码一样快，它是在确定要实例化的类型，这会增加额外的开销。

>另一个原因是，在 Factory 和 Singleton 情况下，StyletIoC 与 Munq 的速度几乎相同。当容器确定如何实例化类型本身时，StyletIoC 的速度要快一个数量级以上。这要归功于巧妙使用C＃的表达式，包含在 [StyletIoC Technical](./StyletIoC-Technical.md) 中。</font>

[目录](./../Index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[StyletIoC-Technical - StyletIoC 技术](./StyletIoC-Technical.md)
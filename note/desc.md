# desc 解读


##  名词解释

- desc

    desc 是每个prometheus metric所实用的描述符。它的本质是metric 不可变得元数据。这个包中实现的常规metric的实现使用当前文件中的方法管理
它们的desc。使用者只有在使用高级特性（例如ExpvarCollector，custom Collectors或Metrics）的时候才需要处理desc。
    
    在统一注册机构中注册的描述符，如果有相同的名字，则必须满足唯一和一致性的要求：有完全相同的 help 描述， 相同的标签名称, 常量标签和变量标签
常量标签有完全不同的值。 

    如果一些描述符，有完全相同的名称和相同的常量标签值，则它们被认为是相等的。

```


Desc is the descriptor used by every Prometheus Metric. It is essentially
the immutable meta-data of a Metric. The normal Metric implementations
included in this package manage their Desc under the hood. Users only have to
deal with Desc if they use advanced features like the ExpvarCollector or
custom Collectors and Metrics.

Descriptors registered with the same registry have to fulfill certain
consistency and uniqueness criteria if they share the same fully-qualified
name: They must have the same help string and the same label names (aka label
dimensions) in each, constLabels and variableLabels, but they must differ in
the values of the constLabels.

Descriptors that share the same fully-qualified names and the same label
values of their constLabels are considered equal.

Use NewDesc to create new Desc instances.


```


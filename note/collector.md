# collector 收集器接口

## 
- metric 指标，度量标准


### type  Collector interface


    Collector 是一个接口，需要收集的指标都要实现该接口。收集器使用前必须先注册。详见Registerer.Register。

    由Gauge, Counter, Summary,Histogram, Untyped 等包提供的存储指标也是收集器， 它只收集一个度量标准本身。然而，收集器的实现者可以以协调
    的方式收集多个指标或者动态的创建指标。在这个库中已经实现的例子是指标向量【相同的指标多个实例的集合，但是有不同的标签值 】， 例如 GaugeVec，
    SummaryVec 和  ExpvarCollector

### method  (Collector)Describe(chan<- *Desc)

    Describe 发送收集到的指标集合到预先提供的channel中， 最后一个描述符发送完成立刻返回。发送的描述符完全满足desc文档中的一致性（consistency）和
    和唯一性（uniqueness） 要求
    
    同一个收集器发送重复的描述符是允许的，这些重复的描述符被简单的忽略掉。然而两个不同的收集器不得发送重复的描述符。
    
    完全不发送描述符的收集器会被标记为未选中，注册是不会被检测并且收集器会在collect方法中生成任何它认为合适的指标
    
    这个方法在收集器的整个生命周期中，都会有意识的发送描述符， 它可以同事调用， 因此必须以并发安全的方式去实现。
    
    如果收集器在执行此方法是遇到错误， 它必须发送无效的描述符(created with NewInvalidDesc)的错误信号到register

### method 	(Collector)Collect(chan<- Metric)
    
    收集指标时， prometheus 会调用 Collect 方法，接口实现者发送每个收集到的指标到提供的channel中， 一旦最后一个指标发送立即返回。
    每个指标描述符都是有Describe返回中的一个。共享相同描述符的指标， 标签必须有不同的值

### 接口源码

```
package prometheus

// Collector is the interface implemented by anything that can be used by
// Prometheus to collect metrics. A Collector has to be registered for
// collection. See Registerer.Register.
//
// The stock metrics provided by this package (Gauge, Counter, Summary,
// Histogram, Untyped) are also Collectors (which only ever collect one metric,
// namely itself). An implementer of Collector may, however, collect multiple
// metrics in a coordinated fashion and/or create metrics on the fly. Examples
// for collectors already implemented in this library are the metric vectors
// (i.e. collection of multiple instances of the same Metric but with different
// label values) like GaugeVec or SummaryVec, and the ExpvarCollector.
type Collector interface {
	// Describe sends the super-set of all possible descriptors of metrics
	// collected by this Collector to the provided channel and returns once
	// the last descriptor has been sent. The sent descriptors fulfill the
	// consistency and uniqueness requirements described in the Desc
	// documentation.
	//
	// It is valid if one and the same Collector sends duplicate
	// descriptors. Those duplicates are simply ignored. However, two
	// different Collectors must not send duplicate descriptors.
	//
	// Sending no descriptor at all marks the Collector as “unchecked”,
	// i.e. no checks will be performed at registration time, and the
	// Collector may yield any Metric it sees fit in its Collect method.
	//
	// This method idempotently sends the same descriptors throughout the
	// lifetime of the Collector. It may be called concurrently and
	// therefore must be implemented in a concurrency safe way.
	//
	// If a Collector encounters an error while executing this method, it
	// must send an invalid descriptor (created with NewInvalidDesc) to
	// signal the error to the registry.
	Describe(chan<- *Desc)
	// Collect is called by the Prometheus registry when collecting
	// metrics. The implementation sends each collected metric via the
	// provided channel and returns once the last metric has been sent. The
	// descriptor of each sent metric is one of those returned by Describe
	// (unless the Collector is unchecked, see above). Returned metrics that
	// share the same descriptor must differ in their variable label
	// values.
	//
	// This method may be called concurrently and must therefore be
	// implemented in a concurrency safe way. Blocking occurs at the expense
	// of total performance of rendering all registered metrics. Ideally,
	// Collector implementations support concurrent readers.
	Collect(chan<- Metric)
}


```
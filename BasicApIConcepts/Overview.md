#特别声明
这篇文章转载自 https://www.jianshu.com/p/79ace3190819 只是原作者排版过于难以阅读，所以修改了排版，方便阅读,如有侵权立删

# Basic API Concepts
Flink程序是实现基于分布式采集的转换程序（如：过滤器，映射，更新状态，连接，分组，定义窗口，聚合计算），集合最初从源头创建（读取文件，kafka, 本地，内存），最终的计算结果通过sinks返回，例如：把数据写入分布式文件，标准的输出（如：终端的命令行输出），flink运行在各种各样的环境中，独立运行，或嵌入到其他程序中。可以运行在本地的JVM中，或者包含有许多机器的集群中。

根据数据源的类型（有边界数据源与无边界数据源），你可以写批处理（batch）程序或者数据流程序	，写批处理程序用DataAPI，写数据流程序用DataStream程序，这些指南将介绍两种API常见的基本概念，但请参阅我们的数据流指南和批处理指南，以了解关于每个API编写程序的具体信息。

注意：当在实际情况中如何使用API的例子中，我们将使用流数据执行环境（StreamingExecutionEnvironment）和流数据API。在使用DataSetAPI中，概念是相同的，只是需要执行环境（ExecutionEnvironment）和接口（DataSet）变化下。

# DataSet and DataStream
Flink在处理数据是由有两个特殊的类DataSet、DataStream，你可以把他们看作不能改变并且包含重复的数据，在DataSet的情况下数据是有边界的，而对于DataStream来说数据的数量是无边界的。

这些集合（collections）不同于传统的JAVA的集合（collections）在某些关键方面。首先他们是不可变的，一旦创建就不能添加和移除，你也不能简单的检查里面的元素。

集合最初是通过在Flink的程序中添加一个源来创建的，通过使用如map、filter之类的API方法对源集合进行转换，从而派生出新的集合。

# Anatomy of a Flink Program
Flink的程序程序看起来像普通的程序，可以改变数据的收集。每个程序有相同的基础部分组成：
1. 创建一个执行环境。
2. 加载（load）/创建初始数据
3. 指定对数据的转换方法。
4. 指定在什么地方放置计算的结果
5. 触发程序执行

现在我们将对每一个步骤做一个概述，请参考相应的部分以获得更多的详细信息。
注意所有的Java DataSet API的核心类能够被找到在`org.apache.flink.api.java`，同时Java DataStream API 的核心类在能被找到在`org.apache.flink.streaming.api`

`StreamExecutionEnvironment`是所有Flink程序的基础。你可以获得使用`StreamExecutionEnvironment`上的这些静态方法获得`StreamExecutionEnvironment`:
```Java
getExecutionEnvironment()
createLocalEnvironment()
createRemoteEnvironment(String host,intport,String...jarFiles)
```
通常情况下，你仅会用到`getExecutionEnvironment()`，因为该方法会根据环境做一些正确的事情：如果您在IDE中或常规Java程序中执行你编写的程序，该将创建一个本地环境，该环境将在本地机器上执行你所编写的程序。如果你创建的是JAR程序，并且调用jar程序在命令行，Flink集群将执行你的main方法并且getExecutionEnvironment()将返回一个执行环境来执行你的程序在集群上。

对于指定数据源，执行环境有几个方法可以使用各种各样的方式读取文件：可以逐行读取 CVS文件，使用完全自定义的数据输入格式。只需将文本文件作为序列读取，你可以使用：
```java
final StreamExecutionEnvironment env=StreamExecutionEnvironment.getExecutionEnvironment();
DataStreamtext=env.readTextFile("file:///path/to/file");
```
这将为您提供一个数据流，然后您可以应用转换来创建新的派生数据流。

您可以使用转换函数调用数据流的方法来实现转换。例如：下面这样Map 转换：
```java
DataStreaminput=...;DataStreamparsed=input.map(newMapFunction(){
    @Override
    public Integer map(String value){
      returnInteger.parseInt(value);
    }
});
```
这将创建新的数据流，把原始数据中每个String类型转化为Integer。

一旦你获得一个最终结构的数据流，你能够写出结果到外部通过创建一个sink,下面是一个创建Sink的例子：
```java
writeAsText(String path)
print()
```
一旦您指定了完整的程序，您就需要通过调用StreamExecutionEnvironment方法的上的execute()`触发程序执行`，基于执行环境的类型，判断是在本地执行，还是提交到集群中执行。

execute()方法返回一个JobExecutionResult结果，它包括一个执行时间和累加结果.

请看[Streaming 指南](https://ci.apache.org/projects/flink/flink-docs-release-1.8/dev/datastream_api.html)，了解关于数据流和Sink信息，以及关于数据流支持转换的更多深度信息。

请参阅[Batch指南](https://ci.apache.org/projects/flink/flink-docs-release-1.8/dev/batch/index.html)，了解关于批处理数据来源和Sink信息，并了解关于数据集的支持转换的更多深度信息


# Lazy Evaluation
所有的Flink程序都是被惰性地执行的，当程序的main方法被执行时，数据加载和转换不会立即发生。而是每个操作被创建和添加到程序计划中，只有在执行环境中的execute()被显式调用时，操作才会实际上是执行。然而程序是在本地执行还是在集群上执行是基于执行环境的类型。

lazy evaluation能够让你创建基于Flink作为整体的执行单元复杂的程序。


# Specifying Keys
一些转换（join，coGroup，keyBy，groupBy）需要一个Key被定义基于数据集的项。其他的转换（Reduce，GroupReduce，Aggregate，Windows）允许在基于Key进行分组在被应用前
一个DataSet被分组的代码：
```java
DataSet<...>input=// [...]
DataSet<...>reduced=input
  .groupBy(/*define key here*/)
  .reduceGroup(/*do something*/);
```
在使用数据流时可以指定一个键
```java
DataStream<...>input=// [...]
DataStream<...>windowed=input
  .keyBy(/*define key here*/)
  .window(/*window specification*/);
```
Flink的数据模型不是基于键值对的。因此你不需要物理的把数据弄成键值对，Keys是抽象的，它们被定义为用于指导分组操作符的实际数据的函数。

*NOTD:*在接下来的讨论中，我们将用到DataStreamAPI和keyBy，对于DataSet API你仅需要替换DataSet和groupBy

# Define keys for Tuples
最简单的实例进行分组Tuples基于一个或多个字段上分组Tuple:
```java
DataStream<Tuple3<Integer,String,Long>> input = // [...]
KeyedStream<Tuple3<Integer,String,Long>,Tuple> keyed = input.keyBy(0)
```
这些tuples被分组基于第一个字段
```java
DataStream>input=// [...]
KeyedStream,Tuple>keyed=input.keyBy(0,1)
```
我们分组Tuples基于第一个字段和第二个字段组合的Key,。

关于嵌套Tuples说明：如果你有一个数据流的嵌套tuple像下面一样
```java
DataStream<Tuple3<Tuple2<Integer, Float>,String,Long>> ds;
```
指定keyBy(0)将导致系统使用整体Tuple2作为key(使用Integer和Float作为Key),如果你想"navigate"到内嵌的Tuple2您必须使用下面的字段表达式进行解释

# Define keys using Field Expressions

可以使用String-based字段表达式来引用嵌套的字段并且定义Key用来grouping，sorting，joining，coGrouping。

字段表达式可以很容易地选择(嵌套)复合类型的字段，比如Tuple和POJO类型。

在下面的示例中，我们有一个WC POJO，它有两个字段“word”和“count”。按照字段词来分组，我们只是将它的名称传递给keyBy()函数。
```java
// some ordinary POJO (Plain old Java Object)
public class WC{
  public String word;
  public int count;
}
DataStreamwords=// [...]
DataStreamwordCounts=words.keyBy("word").window(/*window specification*/);
```
字段表达式语法:
1. 选择POJO字段根据字段的名字。例如，“user”指的是POJO类型的“user”字段。
2. 通过字段名或0-偏移字段索引来选择Tuple字段，例如，“f0”和“5”分别指的是Java Tuple类型的第一个和第六个字段。
3. 您可以在pojo和Tuples中选择嵌套的字段。例如："user.zip"指的是zip被存在一个POJO类型为"user"字段中，任何内嵌和混合的POJO和Tuples 支持 如："f1.user.zip"和"user.f3.1.zip".这种形式。
4. 您可以使用“*”选择完整的类型，这也适用于不是Tuples或POJO类型的类型。

字段表达式例子：
``` java
public static class WC {
  public ComplexNestedClass complex; //nested POJO
  private int count;
  // getter / setter for private field (count)
  public int getCount() {
    return count;
  }
  public void setCount(int c) {
    this.count = c;
  }
}
public static class ComplexNestedClass {
  public Integer someNumber;
  public float someFloat;
  public Tuple3<Long, Long, String> word;
  public IntWritable hadoopCitizen;
}
```
这些是上面示例代码的有效字段表达式:
  * "count": 在WC类中的count字段。
  * "complex":递归地选择POJO类型复杂的字段的所有字段。
  * "complex.word.f2":选择嵌套的Tuple3的最后一个字段。
  * "complex.hadoopCitize":选择Hadoop IntWritable类型。


# Define keys using Key Selector Functions
定义键的另一种方法是“key selector”函数。一个key选择器函数将一个元素作为输入，并返回元素的key。key可以是任意类型的，可以从任意计算中派生出来。

下面的示例显示了一个键选择函数，它简单地返回一个对象的字段:
```java
// some ordinary POJO
public class WC {public String word; public int count;}
DataStream<WC> words = // [...]
KeyedStream<WC> keyed = words
  .keyBy(new KeySelector<WC, String>() {
     public String getKey(WC wc) { return wc.word; }
   });
```
# Specifying Transformation Functions
大多数转换需要用户自定义方法。本节列出了如何指定它们的不同方式

## Implementing an interface
最基本的方法是实现一个提供的接口。
```java
class MyMapFunction implements MapFunction<String, Integer> {
  public Integer map(String value) { return Integer.parseInt(value); }
};
data.map(new MyMapFunction());
```

## Anonymous classes
您可以作为匿名类传递一个函数:
```java
data.map(new MapFunction<String, Integer> () {
  public Integer map(String value) { return Integer.parseInt(value); }
});
```

## Java 8 Lambdas
Flink还在Java API中支持Java 8。请参阅完整的Java 8指南。
```java
data.filter(s->s.startsWith("http://"));
data.reduce((i1,i2)->i1+i2);
```

## Rich functions
所有需要用户定义函数的转换都可以将其作为一个富函数。，例如： 替换
```java
class MyMapFunction implements MapFunction<String, Integer> {
  public Integer map(String value) { return Integer.parseInt(value); }
};
```

你可以写的
```java
class MyMapFunction extends RichMapFunction<String, Integer> {
  public Integer map(String value) { return Integer.parseInt(value); }
};
```
然后像往常一样将方法传递给映射转换:
```java
data.map(newMyMapFunction());
```
富函数也可以定义为一个匿名类:
```java
data.map (new RichMapFunction<String, Integer>() {
  public Integer map(String value) { return Integer.parseInt(value); }
});
```
Rich function除了提供用户自定义的方法（map, reduce等）富函数还提供了四个方法:open，close，getRuntimeContext，setRuntimeContext.这些对于参数化函数(seePassing Parameters to Functions)、创建和最终的局部状态，访问broadcast变量，存储如累加器和计数器之类的运行时信息(seeAccumulators and Counters))，以及迭代的信息(seeIterations)，都是很有用的。

# Supported Data Type
Flink对DataSet或DataStream中的元素类型进行了一些限制。这样做的原因是系统分析需要利用这些类型来决定有效的执行策略

有六种不同类型的数据类型:
1. Java Tuples和Scala Case Classes
2. Java POJOs
3. Primitive Types
4. Regular Classes
5. Values
6. Hadoop Writables
7. Special Types

## Tuples and Case Classes
Tuples包含有不同类型的固定数量的字段的复合类型。Java API提供了从Tuple1到Tuple25的类。Tuples的每个字段都可以是任意的Flink类型，包括进一步的Tuple，结果是嵌套Tuple。可以使用字段的名称直接访问像tuple.f4，或者使用通用的getter方法tuple.getField(int position)，字段开始于0，这与Scala的Tuples形成了对比，但是它更符合Java的一般索引。
```java
DataStream<Tuple2<String, Integer>> wordCounts = env.fromElements(
    new Tuple2<String, Integer>("hello", 1),
    new Tuple2<String, Integer>("world", 2));

wordCounts.map(new MapFunction<Tuple2<String, Integer>, Integer>() {
    @Override
    public Integer map(Tuple2<String, Integer> value) throws Exception {
        return value.f1;
    }
});
wordCounts.keyBy(0); // also valid .keyBy("f0")
```

### POJOs
Java和Scala类被Flink当作一种特殊的POJO数据类型，如果它们满足以下要求:
  * 类必须是public
  * 它必须有一个没有参数的公共构造函数(默认构造函数)
  * 所有字段要么是public，要么必须通过getter和setter函数访问。对于一个名为foo的字段，getter和setter方法必须命名为getFoo()和setFoo()。
  * 一个Field类型必须由Flink支持。目前，Flink使用Avro来序列化任意对象(比如日期)。

Flink分析了POJO类型的结构。它学习了一个POJO的字段。因此，POJO类型比一般类型更容易使用。此外，Flink可以比一般类型更有效地处理POJO。
下面的例子展示了一个简单带有两个共有字段的POJO对象
```java
public class WordWithCount {

    public String word;
    public int count;

    public WordWithCount() {}

    public WordWithCount(String word, int count) {
        this.word = word;
        this.count = count;
    }
}

DataStream<WordWithCount> wordCounts = env.fromElements(
    new WordWithCount("hello", 1),
    new WordWithCount("world", 2));

wordCounts.keyBy("word"); // key by field expression "word"
```

### Primitive Types
Flink支持所有的Java和Scala基本类型，比如Integer,String, andDouble.

## General Class Types
Flink支持大多数Java和Scala类(API和自定义)。限制使用包含不能序列化字段的类，比如文件指针、输入/输出流或其他本地资源，遵循Java bean约定的类通常工作得很好。所有没有被确定为POJO类型的类(参见上面的POJO需求)都是由Flink作为一般类类型处理的。Flink将这些数据类型视为黑盒子，无法访问他们的内容(即:有效的排序)。一般类型使用Kryo的序列化框架来进行/序列化。

### Values
值类型(Value Type)可以手动描述它们的序列化和反序列化。他们没有使用通用的序列化框架，而是通过实现org.apache.flinktype来为这些操作提供定制代码。使用读取和写入的方法的值接口，使用值类型是合理的，因为一般的序列化是非常低效的。例如，一个数据类型实现了作为数组的元素的稀疏向量。由于知道数组大部分为零，所以可以为非零元素使用特殊的编码，而一般的序列化则只需要编写所有的数组元素。org.apache.flinktypes.CopyableValue接口以类似的方式支持手动的内部克隆逻辑。Flink带着与基本数据类型对应的预定义值类型。(ByteValue,ShortValue,IntValue,LongValue,FloatValue,DoubleValue,StringValue,CharValue,BooleanValue)。这些值类型充当基本数据类型的可变变体:它们的值可以被修改，允许程序员重用对象并从垃圾收集器中释放压力。

### Hadoop Writables
您可以使用实现org.apache.hadoop的类型。可写接口。在write()和readFields()方法中定义的序列化逻辑将用于序列化。

### Special Types
您可以使用特殊类型，包括Scala的选项、选项和尝试。Java API也有自己的自定义实现。与Scala类似，它代表了一种两种可能类型的值，左或右。对于需要输出两种不同类型记录的错误处理或操作符，这两种方法都很有用。

### Type Erasure & Type Inference
注意:本节只与Java相关。

在编译之后，他的Java编译器会抛出大量的泛型类型信息。这在Java中被称为类型擦除。这意味着在运行时，对象的实例不再知道它的泛型类型。例如，对于JVM来说，`DataStream<String>`和`DataStream<Long>`看起来是一样的。

Flink在准备执行程序的时候需要类型信息(当程序的主要方法被调用时)。Flink Java API试图重构以各种方式抛出的类型信息，并将其显式地存储在数据集和操作符中。您可以通过数据ream.gettype()检索类型。这个方法返回一个`TypeInformation`的实例，这是Flink的内部方式来表示类型。

类型推断有其局限性，在某些情况下需要程序员的“合作”。示例用collections创建data set的方法,如ExecutionEnvironment.fromCollection(),在那里你可以传递一个参数描述类型。但是，像MapFunction<I, O>这样的泛型函数可能需要额外的类型信息。

通过实现[ResultTypeQueryable](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/java/typeutils/ResultTypeQueryable.java)接口的input formats 和 functions来明确api的返回类型。函数调用的`input types`通常可以通过前一个操作的结果类型来推断。

# Accumulators & Counters
累加器是一个简单的构造，有一个`添加操作`和`一个最终积累的结果`，在作业结束后可用。

最简单的累加器是一个`counter`:你可以增加值通过`Accumulator.add(V value)`方法。在工作结束时，Flink将sum(merge)所有的部分结果，并将结果发送给客户。在调试过程中，积累器是有用的，或者如果您想要了解更多关于您的数据的信息。

Flink目前有以下`内置的累加器`。每个都实现了 Accumulator interface。
* `IntCounter`,`LongCounter` and `DoubleCounter`:下面是使用计数器的示例
* `Histogram`:一个用于离散数量的容器的直方图实现。在内部，它只是一个从整数到整数的映射。您可以使用它来计算值的分布，例如，一个单词计数程序的每一行字的分布。

### How to use accumulators:
首先，您必须在用户定义的转换函数中创建一个累加器对象(这里是一个counter)，您想要使用它。
```java
private IntCounter numLines=newIntCounter();
```
其次，必须注册累加器对象，通常在富函数的open()方法中，这里还定义了名称。
```java 
getRuntimeContext().addAccumulator("num-lines",this.numLines);
```
您现在可以在操作符函数的任何地方使用累加器，包括open()和close()方法。
```java 
this.numLines.add(1);
```
整个结果将存储在JobExecutionResult对象中，该对象从执行环境的execute()方法中返回(当前只有当执行等待作业完成时才会工作)。
```java
myJobExecutionResult.getAccumulatorResult("num-lines")
```
所有的Accumulator都在每个作业中共享一个名称空间。因此，您可以在不同的操作符函数中使用相同的Accumulator。Flink将在内部将所有 name 相同的Accumulator都合并在一起。

关于accumulators和iterations的注意:目前，accumulators的结果只有在整个job结束后才可用。我们还计划在下一次迭代中使用前一个迭代的结果。您可以使用Aggregators来计算每次迭代的统计数据，并根据这些统计数据来终止迭代。

### Custom accumulators:
如果你要实现一个自己的accumulators，你只需你实现 Accumulator interface。如果你认为你的定制累加器应该和Flink一起运送，就可以创建一个拉请求（Feel free to create a pull request if you think your custom accumulator should be shipped with Flink）。

您可以选择实现 Accumulator or SimpleAccumulator。

Accumulator<V,R>是最灵活的:它定义一个类型V的值来添加值，一个结果类型R用于最终的结果。对于histogram，V是一个数字，而R是一个histogram。SimpleAccumulator适用于两种类型相同的情况，例如用于计数器。

作者：MiyoungCheng
链接：https://www.jianshu.com/p/79ace3190819
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

---
title: Java8新特性之Stream
date: 2018-02-27 11:29:44
tags: [java]
categories: java
---

这个包主要提供元素的streams函数操作，比如对collections的map,reduce.   
例如：
```java
int sum = widgets.stream()
                      .filter(b -> b.getColor() == RED)
                      .mapToInt(b -> b.getWeight())
                      .sum();
```
本例中的widgets是Stream的源，类型为Collection，通过filter->map->reduce操作获取red widget的sum weight. 
除了基本的Stream,JAVA8还专门为基础类型int,long,double提供了`IntStream`,`LongStream`,`DoubleStream`.

### Streams和collections的不同之处

1. Stream没有存储。即不是一个存储结构，而是通过管道操作从Array,IO channel等转换而来。
2. Stream本质上是泛函。Streams上的操作不会修改源数据，比如filter操作只是将符合要求的数据放在一个新的Stream,而不是真的删除源数据。
3. Laziness-seeking，懒查询。
4. 大小不受限制，collections 有大小限制，streams没有。比如，limit或者findfirst等Short-circuiting操作会在有限的时间内完成无限的大小的streams操作。
5. 不可逆的，streams中的元素只能被操作一次。就像流水一样，一去不复返。如果你希望在源数据中获取一个新的Stream,必须在源数据上重新开始。

### Stream创建
怎样创建一个Stream呢，有下面几种方法：

- Collection的Stream()和paralleStream()方法。
- Arrays.stream(Object[])
- Stream 类的静态方法，比如:  

> `Stream.of(Object[]),IntStream.range(int, int)`   
> `Stream.iterate(Object, UnaryOperator)`

- File的lines操作， BufferedReader.lines();
- Files中的一些方法获取file path的stream.例如:

> `static Stream<Path> list(Path dir)`  
> `static Stream<Path>  walk(Path start, FileVisitOption... options)`

- Random.ints()返回IntStream 
- JDK中也有很多方法产生Stream,比如

> `BitSet.stream()`   
`Pattern.splitAsStream(java.lang.CharSequence)`   
`JarFile.stream()`   


##### 1.接口
> `BaseStream<T,S extends BaseStream<T,S>>`  

Streams的Base interface,支持各种并行和线性聚合操作。
`Collector<T,A,R>`，可以将元素聚集在一个容器中，容器类型可变，比如List,Map. 是一个Stream处理完毕之后的结束操作。
```
DoubleStream ，处理double类型的stream 
DoubleStream.Builder, 可变的builder。 
IntStream，处理int类型的stream. 
IntStream.Builder, 可变的builder 
LongStream,处理Long类型的Stream. 
LongStream.Builder,可变的builder

Stream<T> -–数据流，支持并行和线性聚合操作。 
Stream.Builder<T>—-Stream的可变builder.
```
##### 2.类
执行Collector reduce操作的类，比如将元素聚合为collection,list等。 
StreamSupport，创建和处理Stream的低级工具。

##### 3.Enum
```
Collector.Characteristics—Collector的属性，可以用于优化reduce 操作。 
Collector.Characteristics CONCURRENT：表示collector的result container可以被多个线程同时处理。 
Collector.Characteristics UNORDERED：collection操作不保证操作元素的原有顺序 
Collector.Characteristics IDENTITY_FINISH：类型转换时不会检查
```

### Stream的操作和管道

Stream操作是管道操作，操作分为中间操作和结束操作两种。整个管道操作的流程为数据源->中间操作->结束操作。   
**中间操作**比如：`Stream.filter`、`Stream.map`,可以有0个或者多个中间操作。  
**结束操作**比如：`Sream.foreach`、`Stream.reduce`，`Stream.collect`,结束操作是必须的。  
中间操作返回的仍然是stream,这些操作经常是“懒惰的”。比如filter操作，并非真的对源数据进行过滤，而是重新创建一个Stream。而且当多个中间操作时，不是真的每次都去遍历源数据，而是在结束操作时，才会去遍历源数据，从而执行这些中间操作。在遍历中，操作的懒惰性表现在，并不是每次都是遍历所有的数据，比如“找出长度大于1000的第一个String”,找到之后就会返回。

结束操作的目的是将Stream转换为想要的结果或者结构。当结束操作之后，**当前管道结束，不可以再重复使用。如果你仍然想操作源数据，那你必须创建一个新的Stream。**

中间操作又分为**无状态**和**有状态**两种。比如map和filter是无状态操作，也就是各元素间的操作是无关的，可单独进行。而distinct和sorted 是有状态操作，在操作时需要用到前元素，也就是元素之间的处理是相关的。 
有状态的操作必须全部执行完毕的时候才能装入结果中。

##### 1.并行操作
JDK的默认操作都是线性的，或者说单线程的。并行操作可以通过Collection.parallelStream()和BaseStream.parallel()实现。
```java
int sumOfWeights = widgets.parallelStream()
                               .filter(b -> b.getColor() == RED)
                               .mapToInt(b -> b.getWeight())
                               .sum();
```

检查一个Stream的执行方式可以调用 isParallel() 方法。

但并不是所有的数据都是可以使用并行操作的。要执行的操作必须是可以并行执行的，也就是多个线程操作之后，和顺序执行的结果是一致的，即线程安全。

```java
int[] wordLength = new int[12];

Stream.of("It", "is", "your", "responsibility").parallel().forEach(s -> {
    if (s.length() < 12) wordLength[s.length()]++;
});
```

比如上面的wordlength操作就是非线程安全的。

##### 2.非干扰性
通常在结束操作之前，源数据都是可以修改的。除了一些Concurrent数据，还有iterator,spliterator操作。 
下面的例子就是正确的：

```java
List<String> l = new ArrayList(Arrays.asList("one", "two"));
     Stream<String> sl = l.stream();
     l.add("three");
     String s = sl.collect(joining(" "));
```

##### 3.无状态的行为
在streams操作中使用无状态的行为，即操作的结果不受其他状态的影响，就比如上面提高的parallel操作要考虑多线程的行为，是否有线程安全问题。

##### 4.collect,reduce操作通常是线程安全的.

##### 5.顺序
对于有顺序的源数据，比如list,set,Stream操作之后仍然会按照顺序排（线性操作）。   
如果是非固定顺序的数据源，则不保证其最后处理的顺序。   
对于线性Streams,源的固定顺序对性能没有特别影响。   
对于并行Streams,不强调顺序的话显然性能会更好。比如filter,distinct,groupingBy操作，非顺序操作的效率就更好.  
另外，如果数据本身是固定顺序的，但是开发者本身不在意是否按顺序处理，那么可以通过unordered()处理之后再操作，会提高某些有状态的并行操作和结束操作的效率。

下面穿插一些线程安全的操作例子：
```java
List<Integer> datalist=new ArrayList<>(1000);
List<Integer> datalist_collect=new ArrayList<>();
//无状态的并行操作，即元素的操作和其他元素状态无关，线程安全
        long sum = IntStream.range(0,1000).parallel().filter(s->s%2==0).count();
        System.out.println("并行:"+sum);
//datalist为非线程安全的，因此属于有状态并行操作，非线程安全     IntStream.range(0,1000).parallel().forEach(data->datalist.add(data));
        System.out.println("并行foreach:"+datalist.size());

//collect操作线程安全
datalist_collect=IntStream.range(0,1000).parallel().collect(ArrayList::new, (c,e)->c.add(e), (c1,c2)->c1.addAll(c2));
        System.out.println("并行collect:"+datalist_collect.size());

        long sum2 = IntStream.range(0,1000).filter(s->s%2==0).count();
            System.out.println("串行:"+sum2);


        List<Integer> intlist=IntStream.range(0,10).collect(ArrayList::new, (c,e)->c.add(e), (c1,c2)->c1.addAll(c2));
        System.out.println("并行数据开始：");
  //并行操作时无顺序    intlist.parallelStream().map(e->e+1).forEach(System.out::println);

        System.out.println("串行数据开始：");
//串行操作保证顺序
        intlist.stream().forEach(System.out::println);
        System.out.println("串行数据结束：");
```

运行结果如下：

```
并行:500
并行foreach:980
并行collect:1000
串行:500
并行数据开始：
7
8
10
6
5
4
1
9
3
2
串行数据开始：
0
1
2
3
4
5
6
7
8
9
串行数据结束
```

##### 6.Reduction 操作 
Reduction操作将元素通过合并操作还原为一个result,比如总数，最大值，元素存入list等。   
Reduction操作的函数有很多，比如 reduce(),collect(),sum(),max(),count()等。   
Reduction操作不仅看起来简洁，而且可以使用并行来实现，效率高，不存在效率安全问题，比如  
```java
int sum = 0;
    for (int x : numbers) {
       sum += x;
    }
//改写为
int sum = numbers.parallelStream().reduce(0, Integer::sum);

```

Reduce()定义1：
```java
T reduce(T identity,
         BinaryOperator<T> accumulator)
```
该方法使用提供的identity和accumulator来获取result.等同于下面的写法，其实identity就是计算的初始值：
```java
T result = identity;
     for (T element : this stream)
         result = accumulator.apply(result, element)
     return result;
```

但于for循环不同的是，reduce操作不局限于线性运行。但对于同一个对象T,其accumulator的运算结果必须相同。

Reduce()定义2：
```java
Optional<T> reduce(BinaryOperator<T> accumulator)
//这个操作等同于：
boolean foundAny = false;
     T result = null;
     for (T element : this stream) {
         if (!foundAny) {
             foundAny = true;
             result = element;
         }
         else
             result = accumulator.apply(result, element);
     }
     return foundAny ? Optional.of(result) : Optional.empty();
```

Reduce()定义3：

```java
<U> U reduce(U identity,
             BiFunction<U,? super T,U> accumulator,
             BinaryOperator<U> combiner)
//这个操作等同于：
 U result = identity;
     for (T element : this stream)
         result = accumulator.apply(result, element)
     return result;
```
此方法通常用于将map和reduce操作的结合来使用。



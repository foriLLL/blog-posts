Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。Stream API（java.util.stream） 把真正的函数式编程风格引入到Java中，使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

## 什么是stream
Stream（流）是一个来自数据源的元素队列并支持聚合操作。  
* **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
* **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

他不是一种数据结构，只是某种数据源的一个视图，数据源可以是一个数组，Java容器或者I/O channel等。对stream的任何修改都不会修改背后的数据源，比如对stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新stream。

### 特征
和以前的Collection操作不同， Stream操作还有两个基础的特征：

1. **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
2. **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

## 生成流
通过 `stream()` 方法为集合创建流。
```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
```
## 聚合操作
### map + collect用法
获取List列表的所有id
```java
List<Long> ids = demos.stream().map(Demo::getId).collect(toList());
```
list变成map
```java
 Map<Long, Demo> map = demos.stream().collect(toMap(Demo::getId, o -> o));
```
### filter
```java
Demo target = demos.stream()
            .filter(d -> targetName.equals.equals(d.getName()))
            .findFirst()
            .orElse(null);
```
注意`find()`返回的结果是Optional。
### match
match()是filter()的缩写版本，返回结果只有boolean类型，返回是否匹配。
```java
boolean flag = demos.stream()
            .map(Demo::getName)
            .anyMatch(condition::contains);
```
### distinct
去重
```java
List<Long> ids = demos.map(Demo::getId).distinct().collect(Collectors.toList());
```
***
Stream还有一些有用的API比如 `groupby`、`sorted` 等。可在有需要时查询使用。
### flatMap
flatMap方法的作用就和他的名字flat一样，对于调用flatMap的流的每一个元素，执行flatMap入参中的函数式方法，由于该函数式方法必须返回一个stream<T>类型的流，这样对于调用flatMap的操作来说，就收集了另一种类型(<T>)的流，并在后续的操作中将<T>类型进行合并，最终产生一个stream<T>的流，而不是一个stream<stream<T>>类型的流。

 
## 参考：
* https://blog.csdn.net/qq_37131111/article/details/99546357  
* https://www.runoob.com/java/java8-streams.html
* https://www.cnblogs.com/xfyy-2020/p/13289066.html
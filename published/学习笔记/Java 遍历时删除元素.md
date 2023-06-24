---
description: "之前写代码时遇到过在遍历时需要删除对象元素的问题，也记得看到过好多解决方案，这里做一个整理。"
time: 2022-03-04
---

之前写代码时遇到过在遍历时需要删除对象元素的问题，也记得看到过好多解决方案，这里做一个整理。  

## 两种常见错误方法
```java
@Test void concurrentTest(){
    List<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    list.add(3);
    list.add(4);
    list.add(5);
    for(Integer i: list){
        if(i.equals(3)){
            list.remove(i);
        }
    }
}
```
在以上代码中，需要在遍历集合的同时删除其中的一个元素，但是这样的删除方式会抛出`ConcurrentModificationException`的运行时异常，使用IDEA也会同时提醒The loop can be replaced with 'Collection.removeIf'。

JDK的API中对该异常的描述是：  
当方法检测到对象的并发修改，但不允许这种修改时，抛出此异常。
例如，**某个线程在 Collection 上进行迭代时，通常不允许另一个线性修改该 Collection**。通常在这些情况下，迭代的结果是不确定的。如果检测到这种行为，一些迭代器实现（包括 JRE 提供的所有通用 collection 实现）可能选择抛出此异常。执行该操作的迭代器称为快速失败迭代器，因为迭代器很快就完全失败，而不会冒着在将来某个时间任意发生不确定行为的风险。
注意，此异常不会始终指出对象已经由不同 线程并发修改。如果单线程发出违反对象协定的方法调用序列，则该对象可能抛出此异常。例如，如果线程使用快速失败迭代器在 collection 上迭代时直接修改该 collection，则迭代器将抛出此异常。
注意，迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败操作会尽最大努力抛出 ConcurrentModificationException。因此，为提高此类操作的正确性而编写一个依赖于此异常的程序是错误的做法，正确做法是：ConcurrentModificationException 应该仅用于检测 bug。

若使用索引来删除元素，如以下代码
```java
 @Test void concurrentTest(){
    List<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    list.add(2);
    list.add(4);
    list.add(5);
    for(int i = 0;i<list.size();i++){
        if(2== list.get(i)){
            list.remove(i);
        }
    }
    System.out.println(list);
}
```
得到的结果是
```
[1, 2, 4, 5]
```
只删除了一个2，是因为list删除了一个元素后，其他元素向前补齐，第二个2被遗漏。

## 常见解决方案
在对Java集合进行遍历同时删除必须要使用迭代器。  
第一种解决方案就是使用迭代器：
```java
Iterator<Integer> it = list.iterator();
while(it.hasNext()){
    if(2==it.next()){
        it.remove();
    }
}
```
使用迭代器遍历且使用迭代器自己的`remove`方法，这个迭代器仍然是合法的。这里需要注意的是每次调用迭代器的`next`只能调用一次`remove`方法，连续调用会抛出`UnsupportedOperationException`，切调用`remove`前必须调用过一次`remove`。
第二种解决方案就是利用IDEA的推荐，修改为
```java
list.removeIf(i -> i.equals(3));
```
调用接口来删除集合中的元素，也是一种解决方案。



## 参考
* https://www.cnblogs.com/jinglecode/p/5603545.html
* https://www.cnblogs.com/goody9807/p/6432904.html
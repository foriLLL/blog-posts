---
description: "在 Java 中，Queue 接口常用的两种实现分别是 LinkedList 以及 ArrayDeque，之前在写代码中，我一般都是用 ArrayDeque，而今天突然发现原来 ArrayDeque 不能在队列中保存 null 值，而 LinkedList 可以。"
time: 2023-09-04T10:05:55+08:00
heroImage: ""
---

在 Java 中，`Queue` 接口常用的两种实现分别是 `LinkedList` 以及 `ArrayDeque`，之前在写代码中，我一般都是用 `ArrayDeque`，而今天突然发现原来 `ArrayDeque` 不能在队列中保存 `null` 值，而 `LinkedList` 可以。  
在这里记录一些关于这两种实现的差异。

## ArrayDeque

`ArrayDeque` 内部维护了一个循环数组，是一个基于数组实现的非常高效的双端队列，它优于 LinkedList，因为它的数组实现比链表实现在大多数情况下都有更好的缓存局部性。  
其默认的容量为 `16`，当队列中的元素个数超过容量时，会进行扩容，扩容的策略是将容量扩大为原来的两倍。

它使用 `null` 值来检测队列中的元素是否为空，因此不能在队列中保存 `null` 值，否则会抛出 `NullPointerException`。 这一点也在文档中显式的说明了。

> Resizable-array implementation of the Deque interface. Array deques have no capacity restrictions; they grow as necessary to support usage. They are not thread-safe; in the absence of external synchronization, they do not support concurrent access by multiple threads. Null elements are prohibited. This class is likely to be faster than Stack when used as a stack, and faster than LinkedList when used as a queue.  
>

从这里可以看出，`ArrayDeque` 不是线程安全的，禁止 null 值传入。  
在作为栈使用时，比 `Stack` 更快，而在作为队列使用时，比 `LinkedList` 更快。

## LinkedList

`LinkedList` 是一个典型的双向链表实现，具有链表的所有基本特性，也实现了 `Deque` 接口。虽然它在插入和删除元素时有 $O(1)$ 的时间复杂度（假设已经有了节点的引用），但在访问元素时，尤其是中间的元素，它比基于数组的列表（如 ArrayList）慢，需要 $O(n)$ 的时间复杂度。

> Doubly-linked list implementation of the List and Deque interfaces. Implements all optional list operations, and permits all elements (including null).

`LinkedList` 内部有一个私有的静态节点类，通常称为 `Node`。这个 `Node` 类有三个主要的属性：

* item：存储元素的值。
* next：指向下一个节点的引用。
* prev：指向前一个节点的引用。

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

`LinkedList` 维护两个特定的节点引用：`first` 和 `last`，分别指向链表的第一个节点和最后一个节点。当链表为空时，这两个引用都是 null。

`LinkedList` 允许存储 null 元素，因为每个节点都有明确的前后引用，所以可以容易地区分一个包含 null 的节点和一个真正不存在的节点。

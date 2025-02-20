---
description: "协同编辑是指多个用户同时编辑同一文档，并确保所有用户的内容保持一致。本文介绍了两种常见的协同编辑实现方式：基于操作转换（OT）和无冲突复制数据类型（CRDT），并深入探讨了 CRDT 在去中心化协同编辑中的潜在应用及其未来发展。"
time: 2024-09-12T10:18:54+08:00
tags: 
heroImage: 
---

## 协同编辑的概念

协同编辑指的是多个用户可以同时对同一文档进行编辑，并确保所有用户看到的内容保持一致。它的核心挑战是如何处理多个用户的并发操作，避免冲突并确保最终一致性。

## 协同编辑的关键挑战

- **并发冲突**：当多个用户同时修改相同内容时，如何协调这些修改以避免内容不一致。
- **最终一致性**：无论操作顺序如何变化，系统需要确保所有用户最终都能看到相同的编辑结果。

## 实现协同编辑的常用方法

### OT (Operational Transformation)

OT 通过对操作进行转换来处理并发编辑。它提供了一种操作转换器，能够在出现“脏路径”（多个用户的操作依赖于编辑的路径，同样的编辑操作如果顺序不同会导致结果不同）时，将并发操作进行调整。OT 的主要步骤包括：
- 检测冲突操作：识别多个用户对同一部分内容的编辑冲突。
- 修正冲突操作：**调整冲突的操作，如修正文本的插入/删除位置，或调整属性修改，使其与其他操作兼容**。
- 通过转换确保最终一致性：每个用户的操作在应用时**经过转换**，确保所有用户的视图最终一致。

下图是 OT 解决「脏路径」和「并发冲突」的直观理解。

<div style="display: flex; flex-wrap: wrap; justify-content: center; align-items: center;">
  <img alt="OT 解决脏路径" src="https://img.foril.space/OT 解决脏路径.png" style="max-width: 600px; width: 350px; margin: 10px;">
  <img alt="OT 解决并发冲突" src="https://img.foril.space/OT 解决并发冲突.png" style="max-width: 600px; width: 350px; margin: 10px;">
</div>


### CRDT (Conflict-free Replicated Data Types)

CRDT 即 「无冲突复制数据类型」，它主要被应用在分布式系统中，保证分布式应用的数据一致性，文档协同编辑可以理解为分布式应用的一种。**它的本质是数据结构，通过数据结构的设计保证并发操作数据的最终一致性**。

与 OT 不同，CRDT 不需要依赖复杂的操作转换机制，而是通过数学上的设计确保所有用户对数据的修改可以无冲突地合并，最终达成一致性。

大多数的 CRDT **为在文档中创建的每个字符分配一个唯一的标识符**。为了确保文档始终能够收敛，CRDT 模型即使在删除字符时也会保留元数据。

CRDT 最初是为了解决分布式系统最终数据一致性而提出的，它支持各个主机副本之间数据修改的直接同步，而且<span style="color: red">数据修改的同步顺序以及同步的次数不影响最终结果，只要修改操作一致，数据的最终状态就是一致的，也就是通常大家说的 CRDT 数据的满足交换性和幂等性。</span>

下面用一个例子来说明 CRDT 的工作原理，主要体现修改的顺序对最终结果没有影响。

<img alt="CRDT 修改例子" src="https://img.foril.space/CRDT 修改例子.png" width=300px style="display: block; margin:10px auto"/>

我们设计一种编辑的表达方式 $XXX_{a,b}$ 作为一个操作的标识，$a$ 表示操作的作者编号，$b$ 表示操作的序号。对于每个编辑，都有一个全局唯一的 id，这个 id 由作者编号和序号组成。对同样的编辑 id 集合，无论他们的顺序如何，最终的结果都是一样的。

在上面的图片中，原本的文本是 `AB`，用户 0 在 AB 中间插入一个字符 `C`，接着在 `C` 后面插入一个字符 `D`，得到 `ACDB`。但同时用户 1 并发的在 `AB` 中间插入一个字符 `E`，此时，在拿到这三个编辑的集合后，我们通过一些规则来合并这些编辑操作，比如用户 1 的用户编号比用户 0 的大，那么可以把所有用户 1 的操作排在用户 0 的操作后面，最终的结果是 `ACDEB`。

> CRDT 有着极佳的去中心化性质，它在 Web 3.0 时代或许有机会成为某种形式的前端基础设施。并且相对于经典的 OT，近年来 CRDT 的流行或许也属于一次潜在的范式转移（paradigm shift），这对前端开发者们意味着全新的机遇。
同时，CRDT比较适合来记录用户产生的各种事件的集合，因此将这些事件直接用于推荐系统的计算也是自然而然的事情。特别是对于很多大规模的企业，需要将用户的数据分布在多个数据中心，必须要实现读写两方面的高可用，特别是一些边缘计算的场景，可以利用CRDT的去中心化、离线、无冲突等特性实现用户操作数据的收集和同步。

### OT 和 CRDT 的区别

- **OT**: 需要在客户端之间进行操作的同步和转换，确保操作以合适的顺序应用。
- **CRDT**: 依赖于数据结构的设计，天然具备冲突合并的能力，因此不需要像 OT 那样的操作转换。

## 开源解决方案

- **ShareDB**: ShareDB 是一个基于 OT 的实时协作编辑库，它提供了一个简单的 API，用于在 Node.js 服务器上实现实时协作编辑功能。
- **Yjs**: Yjs 是一个基于 CRDT 的实时协作编辑库，它提供了一种简单的方法来实现实时协作编辑功能，支持多种数据结构，如文本、数组、对象等。

## 参考

- [多人协同编辑技术的演进](https://juejin.cn/post/7030327005665034247)
- [多人协同编辑技术初探](https://www.ctyun.cn/developer/article/430592717156421)
- [CRDT与协同编辑](https://www.herui.club/archives/1066)
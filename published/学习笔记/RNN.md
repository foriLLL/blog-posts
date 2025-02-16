---
description: 最近模型又用到了 RNN 模型，发现之前对于 RNN 的理解已经相当模糊了，这里从李沐老师的《动手学深度学习》出发，对 RNN 模型做一个简单的总结。
time: 2023-12-06T15:02:04+08:00
tags: 
heroImage: https://img.foril.space/20231206150254.png
---

循环神经网络（RNN）是为了更好地处理序列信息的一种神经网络。通过引入状态变量来记录过去的信息，从而更好地处理当前的输入。接下来我会从序列任务出发，介绍 RNN 的引入背景并介绍 RNN 的基本结构，然后介绍 RNN 的一些变种。本文并不会涉及有关 RNN 训练时的具体梯度计算，而是从一个更加直观的角度来介绍 RNN 的基本结构和一些变种，以便之后回顾翻阅。

## 序列预测任务

现实生活中有许多问题其实都是序列预测问题，以股票价格为例，我们需要基于之前时间的价格预测时间步 $t$ 的股票价格 $x_t$，即需要估计

$$
x_t \sim P(x_t \mid x_{t-1}, \ldots, x_1).
$$

### 策略一：固定时间跨度（自回归模型）

这种方式就是找一个固定的时间长度 $\tau$，然后用过去 $\tau$ 个时间步的数据来预测当前时间步的数据，即

$$
x_t \sim P(x_t \mid x_{t-1}, \ldots, x_{t-\tau}).
$$

这种方式的好处是参数数量不随时间步的增加而增加，但是这种方式的缺点也很明显，就是无法利用过去的所有信息，因此我们需要引入隐状态变量来保存过去的信息。

### 策略二：隐状态保存过去信息（隐变量自回归模型）

这种方式的思路就是通过引入一个隐状态变量 $h_t$ 来保存过去的信息，然后用这个隐状态变量来预测当前时间步的数据，即

$$
x_t \sim P(x_t \mid h_t), \quad h_t = g(h_{t-1}, x_{t-1}).
$$

通过一个状态变量来保存之前遇到过的所有数据的信息。

## RNN

固定长度的 $n$ 元模型在预测时间步 $t$ 的数据时，只能利用过去 $n-1$ 个时间步的数据，如果想要考虑更长的时间跨度，就需要增加 $n$，但是这样会导致参数数量的增加，比如一个 $n$ 元语法模型，预测时间步 $t$ 的单词时，需要考虑之前 $n-1$ 个单词，那么参数数量就是 $O(|\mathcal{V}|^n)$，其中 $\mathcal{V}$ 是词典的大小。参数数量是随着 $n$ 的增加而指数增加的，这样就会导致模型的训练变得非常困难。  

因此，我们需要引入一个隐状态变量 $h_t$ 来保存过去的信息，用于存储过去所有时间步的信息，这个隐状态变量可以用来预测下一时间步的数据，从当前时间数据和之前时间步的隐状态得到，即

$$
h_t = f(x_{t}, h_{t-1}).
$$

这种方法与上面说的隐变量自回归模型的思路是一致的，这里的 $f$ 函数是一个神经网络，这样的话，我们就可以通过神经网络来学习到更加复杂的模式，而不是像 $n$ 元模型那样只能学习到固定长度的模式。

<img alt="RNN" src="https://img.foril.space/RNN.svg" width=600px style="display: block; margin:10px auto"/>

上面图片展示的就是一个最基础的 RNN 模型，每个时间步的输入 $X_t$ 和隐状态 $H_t$ 都是一个向量，考虑 batch 的情况，$X_t$ 是一个 $n \times d$ 的矩阵，$H_t$ 是一个 $n \times h$ 的矩阵，其中 $n$ 是 batch 的大小，$d$ 是输入的维度，$h$ 是隐状态的维度。  
模型的参数包括 $\mathbf{W}_{xh} \in \mathbb{R}^{d \times h}, \mathbf{W}_{hh} \in \mathbb{R}^{h \times h}$ 和 $\mathbf{b}_q \in \mathbb{R}^{1 \times h}$，其中 $W_{xh}$ 是输入到隐状态的权重矩阵，$W_{hh}$ 是隐状态到隐状态的权重矩阵，$b_q$ 是隐状态的偏置。

### RNN 的计算

在时间步 $t$，输入上一个时间步的隐状态 $H_{t-1}$ 和当前时间步的输入 $X_t$，RNN 会利用全连接层和激活函数来计算当前时间步的隐状态，即

$$
\mathbf{H}_t = \phi(\mathbf{X}_t \mathbf{W}_{xh} + \mathbf{H}_{t-1} \mathbf{W}_{hh}  + \mathbf{b}_q).
$$

其中 $\mathbf{W}_{xh}$ 是输入到隐状态的权重矩阵，$\mathbf{W}_{hh}$ 是隐状态到隐状态的权重矩阵，$\mathbf{b}_h$ 是隐状态的偏置向量，$\phi$ 是激活函数。

> 实际运算时可以分别拼接 $\mathbf{X}_t$ 和 $\mathbf{H}_{t-1}$ 以及 $\mathbf{W}_{xh}$ 和 $\mathbf{W}_{hh}$，然后一次性完成计算，这样可以提高计算效率。即：  
> $$
> \mathbf{H}_t = \phi([\mathbf{X}_t, \mathbf{H}_{t-1}] [\mathbf{W}_{xh}, \mathbf{W}_{hh}] + \mathbf{b}_q).
> $$
> 其中 $[\mathbf{X}_t, \mathbf{H}_{t-1}]$ 表示将 $\mathbf{X}_t$ 和 $\mathbf{H}_{t-1}$ 拼接起来，$[\mathbf{W}_{xh}, \mathbf{W}_{hh}]$ 表示将 $\mathbf{W}_{xh}$ 和 $\mathbf{W}_{hh}$ 拼接起来。得到一个 $n \times (d+h)$ 的矩阵与一个 $(d+h) \times h$ 的矩阵相乘，得到一个 $n \times h$ 的矩阵，然后再加上一个 $1 \times h$ 的偏置向量，最后再通过激活函数 $\phi$ 得到一个 $n \times h$ 的隐状态矩阵。
>
> 这不仅减少了计算步骤，而且在实际运算中往往可以更高效地利用硬件资源。
> 为什么可以这样做的原因是矩阵乘法的分配律和合并操作在数学上是等价的。通过合并输入矩阵和权重矩阵，可以在保持计算结果不变的前提下，简化计算过程，提高效率。这种方法在许多神经网络的实现中都非常常见，它可以显著提升大规模矩阵运算的效率。


然后，我们可以利用隐状态来计算当前时间步的输出，即

$$
\mathbf{O}_t = \mathbf{H}_t \mathbf{W}_{hq} + \mathbf{b}_q.
$$


## GRU（门控循环单元）

引入 GRU 的一个原因是早期的观测值可能对所有未来的预测都有用，但是随着时间的推移，这种记忆会被逐渐淡化，我们需要一个机制来
1. 存储重要的早期信息
2. 控制重置内部状态表示，跳过无关时间步信息

<img alt="GRU" src="https://img.foril.space/GRU.svg" width=600px style="display: block; margin:10px auto"/>

上图是一个完整的 GRU 模型，其中 $R_t$ 是重置门，$Z_t$ 是更新门，$H_t$ 是隐状态，$X_t$ 是输入，这两个门的输出经过 sigmoid 函数后得到的值在 $[0, 1]$ 之间，形状和隐状态 $H_t$ 保持一致，所以可以理解为门的开关：  

| 门类   | 功能描述                           |备注  |
| ------ | ---------------------------------- | ---- |
| 重置门 $R_t$ | 允许我们控制可能还想记住的过去状态的多少 |<span style="color: red;">控制保留程度 [0, 1]</span>|
| 更新门 $Z_t$ | 允许我们控制新状态中有多少个是旧状态的副本 |<span style="color: red;">控制隐状态来自两部分的加权</span>|


### GRU 的计算

具体来说，重置门会在我们计算候选隐状态时发挥作用，与普通 RNN 计算隐状态的方式相同，GRU 中计算候选隐状态的方式相同，也是通过这个时间步的输入 $X_t$ 和上一个时间步的隐状态 $H_{t-1}$，利用全连接层和激活函数来计算当前时间步的隐状态，唯一的区别就是会使用重置门给上一个时间步的隐状态一个权重，即

$$
\tilde{\mathbf{H}}_t = \tanh(\mathbf{X}_t \mathbf{W}_{xh} + \left(\mathbf{R}_t \odot \mathbf{H}_{t-1}\right) \mathbf{W}_{hh} + \mathbf{b}_h),
$$

其中 $\mathbf{W}_{xh} \in \mathbb{R}^{d \times h}$ 和 $\mathbf{W}_{hh} \in \mathbb{R}^{h \times h}$ 是权重矩阵，$\mathbf{b}_h \in \mathbb{R}^{1 \times h}$ 是偏置向量，$\odot$ 表示按元素相乘（Hadamard 积）。同时这里使用 tanh 作为激活函数，确保候选隐状态的值在 $[-1, 1]$ 之间。

有了候选隐状态后，更新门就可以发挥作用，通过给当前候选隐状态和上一个时间步的隐状态做一个加权平均，来得到当前时间步的隐状态，即

$$
\mathbf{H}_t = \mathbf{Z}_t \odot \mathbf{H}_{t-1} + (1 - \mathbf{Z}_t) \odot \tilde{\mathbf{H}}_t.
$$

总之，门控循环单元具有以下两个显著特征：
- 重置门有助于捕获序列中的短期依赖关系；
- 更新门有助于捕获序列中的长期依赖关系。

## LSTM（长短期记忆网络）

LSTM 有许多比 GRU 一样的属性，比 GRU 的设计稍复杂一些，但却比 GRU 早诞生了近 20 年。  
其关键的概念是 **记忆元**（memory cell），他与隐状态 $H_t$ 形状相同（因为隐状态其实就是记忆元的按元素加权变换），用于长期保存信息。它可以被看作是网络的“记忆”，负责存储长期依赖的信息。

同时，由于 RNN 每个时间步的输出都是隐状态 $H_t$ 经过全连接层的输出，因此我们引入一个输出门 $O_t$ 来控制得到 $H_t$。  
所以大体上看计算流程可以理解为通过记忆元传递整个序列的记忆信息，然后通过输出门来控制 $H_t$ 输出。

<img alt="LSTM" src="https://img.foril.space/LSTM.svg" width=600px style="display: block; margin:10px auto"/>

上图给出的是 LSTM 的计算流程，其中 $C$ 是记忆元，$\tilde{C_t}$ 是候选记忆元，$H$ 是隐状态，$X_t$ 是输入。  
$I_t$ 是输入门，$F_t$ 是遗忘门，$O_t$ 是输出门，这三个门的输出经过 sigmoid 函数后得到的值在 $[0, 1]$ 之间，形状和隐状态 $H_t$ 保持一致，所以可以理解为门的开关：

| 门类   | 功能描述                           |备注  |
| ------ | ---------------------------------- | ---- |
| 输入门 $I_t$ | 允许我们控制新信息的流入程度 |<span style="color: red;">采用多少 $\tilde{C_t}$</span>|
| 遗忘门 $F_t$ | 允许我们控制旧信息的流出程度 |<span style="color: red;">采用多少 $C_{t-1}$</span>|
| 输出门 $O_t$ | 允许我们控制隐状态的输出程度 |<span style="color: red;">$H_t$ 来自 $C_t$ 的多少</span>|

### LSTM 的计算

结合图片，我们可以看到 LSTM 的计算流程如下：

- 首先，我们需要计算输入门 $I_t$、遗忘门 $F_t$ 和输出门 $O_t$，这三个门的计算方式都是一样的，都是通过当前时间步的输入 $X_t$ 和上一个时间步的隐状态 $H_{t-1}$，利用全连接层和激活函数来计算当前时间步的隐状态，然后再通过 sigmoid 函数来得到门的开关，即

$$
\begin{split}\begin{aligned}
\mathbf{I}_t &= \sigma(\mathbf{X}_t \mathbf{W}_{xi} + \mathbf{H}_{t-1} \mathbf{W}_{hi} + \mathbf{b}_i),\\
\mathbf{F}_t &= \sigma(\mathbf{X}_t \mathbf{W}_{xf} + \mathbf{H}_{t-1} \mathbf{W}_{hf} + \mathbf{b}_f),\\
\mathbf{O}_t &= \sigma(\mathbf{X}_t \mathbf{W}_{xo} + \mathbf{H}_{t-1} \mathbf{W}_{ho} + \mathbf{b}_o),
\end{aligned}\end{split}
$$

- 然后，我们需要计算候选记忆元 $\tilde{C}_t$，他的计算和三个门的计算类似，但是使用 tanh 函数作为激活函数，函数的值范围为 $(-1, 1)$，即
  $$
    \tilde{\mathbf{C}}_t = \tanh(\mathbf{X}_t \mathbf{W}_{xc} + \mathbf{H}_{t-1} \mathbf{W}_{hc} + \mathbf{b}_c).
  $$
- 接着，我们需要计算记忆元 $C_t$，通过遗忘门 $F_t$ 控制上一个时间步的记忆元 $C_{t-1}$ 的流出程度，通过输入门 $I_t$ 控制当前时间步的候选记忆元 $\tilde{C}_t$ 的流入程度，即
  $$
    \mathbf{C}_t = \mathbf{F}_t \odot \mathbf{C}_{t-1} + \mathbf{I}_t \odot \tilde{\mathbf{C}}_t.
  $$
- 最后，我们需要计算隐状态 $H_t$，通过输出门 $O_t$ 控制记忆元 $C_t$ 的流出程度，即
  $$
    \mathbf{H}_t = \mathbf{O}_t \odot \tanh(\mathbf{C}_t).
  $$

### LSTM 与 GRU 的比较

| 特征/模型         | LSTM（长短期记忆网络） | GRU（门控循环单元） |
|----------------|---------------------|-------------------|
| **结构**          | 包含三个门：输入门、遗忘门和输出门 | 包含两个门：更新门和重置门 |
| **长期记忆能力**  | 较强，适合处理长序列 | 较LSTM略弱，但通常仍足够强大 |
| **参数量**       | 较多，导致更长的训练时间和更高的计算资源需求 | 较少，提高了计算效率 |
| **性能**         | 在处理非常长的序列时表现可能更好 | 在许多任务中与LSTM相似，但计算更高效 |
| **训练时间和计算成本** | 通常需要更多的训练时间和计算资源 | 训练更快，计算成本较低 |
| **适用场景**     | 适合需要复杂模型和长期依赖的任务 | 适合对计算效率有更高要求的场景 |

## 深度 RNN

我们可以将多个 RNN 模型堆叠起来，就可以得到一个深度 RNN 模型，通过对几个简单层的组合，产生了一个灵活的机制。 特别是，数据可能与不同层的堆叠有关。不同层的 RNN 可以捕捉数据中的不同层面和时间尺度的信息。例如，我们可以在第一层中学习短期依赖关系，而在第二层中学习长期依赖关系。

如下图所示：

<img alt="深度RNN" src="https://img.foril.space/深度RNN.svg" width=350px style="display: block; margin:10px auto"/>

### 深度 RNN 的计算

我们可以将深度架构中的函数依赖关系形式化，后续的讨论主要集中在经典的循环神经网络模型上，但是这些讨论也适应于其他序列模型。

假设在时间步 $t$ 有一个小批量的输入数据 $\mathbf{X}_t \in \mathbb{R}^{n \times d}$（batch_size：$n$，每个样本中的输入数：$d$）。同时，将 $l^\mathrm{th}$ 隐藏层（$l=1,\ldots,L$）
的隐状态设为 $\mathbf{H}_t^{(l)}  \in \mathbb{R}^{n \times h}$，输出层变量设为$\mathbf{O}_t \in \mathbb{R}^{n \times q}$。

第一层的隐状态输入用时间步的输入，即设置 $\mathbf{H}_t^{(0)} = \mathbf{X}_t$，第 $l$ 个隐藏层的隐状态使用激活函数 $\phi_l$，则：

$$
\mathbf{H}_t^{(l)} = \phi_l(\mathbf{H}_t^{(l-1)} \mathbf{W}_{xh}^{(l)} + \mathbf{H}_{t-1}^{(l)} \mathbf{W}_{hh}^{(l)}  + \mathbf{b}_h^{(l)}),
$$

其中，权重 $\mathbf{W}_{xh}^{(l)} \in \mathbb{R}^{h \times h}$，$\mathbf{W}_{hh}^{(l)} \in \mathbb{R}^{h \times h}$ 和 偏置 $\mathbf{b}_h^{(l)} \in \mathbb{R}^{1 \times h}$ 都是第 $l$ 个隐藏层的模型参数。

最后，输出层的计算仅基于第 $l$ 个隐藏层最终的隐状态：

$$\mathbf{O}_t = \mathbf{H}_t^{(L)} \mathbf{W}_{hq} + \mathbf{b}_q,$$

其中，权重 $\mathbf{W}_{hq} \in \mathbb{R}^{h \times q}$ 和偏置 $\mathbf{b}_q \in \mathbb{R}^{1 \times q}$ 都是输出层的模型参数。

与多层感知机一样，隐藏层数目 $L$ 和隐藏单元数目 $h$ 都是超参数，可以由我们指定。

## 双向 RNN

在循环神经网络中，每个时间步的输出只取决于该时间步及之前的输入序列，这种结构称为单向循环神经网络。类似地，我们也可以设计双向循环神经网络，捕捉每个时间步的输入序列中，过去和未来的数据信息。这在某些特定任务中是很有用的，例如，当我们需要根据某些序列的过去和未来来预测当前的单词时，双向循环神经网络可以提供更好的特征。

> 我___饿了，我可以吃半头猪。

<img alt="双向RNN" src="https://img.foril.space/双向RNN.svg" width=350px style="display: block; margin:10px auto"/>

对于任意时间步 $t$，给定一个小批量的输入数据 $\mathbf{X}_t \in \mathbb{R}^{n \times d}$，并且令隐藏层激活函数为 $\phi$。  
在双向架构中，我们设该时间步的前向和反向隐状态分别为 $\overrightarrow{\mathbf{H}}_t  \in \mathbb{R}^{n \times h}$ 和 $\overleftarrow{\mathbf{H}}_t  \in \mathbb{R}^{n \times h}$，
其中 $h$ 是隐藏单元的数目。
前向和反向隐状态的更新如下：

$$
\begin{aligned}
\overrightarrow{\mathbf{H}}_t &= \phi(\mathbf{X}_t \mathbf{W}_{xh}^{(f)} + \overrightarrow{\mathbf{H}}_{t-1} \mathbf{W}_{hh}^{(f)}  + \mathbf{b}_h^{(f)}),\\
\overleftarrow{\mathbf{H}}_t &= \phi(\mathbf{X}_t \mathbf{W}_{xh}^{(b)} + \overleftarrow{\mathbf{H}}_{t+1} \mathbf{W}_{hh}^{(b)}  + \mathbf{b}_h^{(b)}),
\end{aligned}
$$

其中，权重 $\mathbf{W}_{xh}^{(f)} \in \mathbb{R}^{d \times h}, \mathbf{W}_{hh}^{(f)} \in \mathbb{R}^{h \times h}, \mathbf{W}_{xh}^{(b)} \in \mathbb{R}^{d \times h}, \mathbf{W}_{hh}^{(b)} \in \mathbb{R}^{h \times h}$ 和偏置 $\mathbf{b}_h^{(f)} \in \mathbb{R}^{1 \times h}, \mathbf{b}_h^{(b)} \in \mathbb{R}^{1 \times h}$ 都是模型参数。

**接下来，将前向隐状态 $\overrightarrow{\mathbf{H}}_t$ 和反向隐状态 $\overleftarrow{\mathbf{H}}_t$ 连接起来**，获得需要送入输出层的隐状态 $\mathbf{H}_t \in \mathbb{R}^{n \times 2h}$。

在具有多个隐藏层的深度双向循环神经网络中，
该信息作为输入传递到下一个双向层。
最后，输出层计算得到的输出为
$\mathbf{O}_t \in \mathbb{R}^{n \times q}$（$q$是输出单元的数目）：

$$
\mathbf{O}_t = \mathbf{H}_t \mathbf{W}_{hq} + \mathbf{b}_q.
$$

这里，权重矩阵 $\mathbf{W}_{hq} \in \mathbb{R}^{2h \times q}$ 和偏置 $\mathbf{b}_q \in \mathbb{R}^{1 \times q}$ 是输出层的模型参数。

事实上，这两个方向也可以拥有不同数量的隐藏单元，只需要最后将它们连接起来即可。

## Encoder-Decoder

Encoder-Decoder 架构想要解决的问题是，当输入和输出都是变长序列时（例如机器翻译任务），如何将它们映射到一个固定维度的向量上。

<img alt="Encoder-Decoder" src="https://img.foril.space/Encoder-Decoder.svg" width=400px style="display: block; margin:10px auto"/>

第一个组件是 **编码器**（encoder），它可以是一个循环神经网络，接受一个可变长度的序列作为输入，输出一个固定长度的向量作为状态。

第二个组件是 **解码器**（decoder），它也可以是一个循环神经网络，接受一个固定长度的向量作为输入，输出一个可变长度的序列。

## seq2seq

利用 Encoder-Decoder 架构，我们可以利用 RNN，设计一个序列到序列（seq2seq）模型，用于处理输入和输出都是可变长度序列的任务，例如机器翻译。  
输入序列的信息被编码到隐状态中。为了连续生成输出序列的词元，解码器基于 **输入序列的编码信息** 和 **输出序列已经看见的或者生成的词元** 来预测下一个词元。

<img alt="seq2seq" src="https://img.foril.space/seq2seq.svg" width=600px style="display: block; margin:10px auto"/>

如图所示的机器翻译任务，编码器每个时间步的输入是一个英语词元，输出是一个固定长度的向量，解码器每个时间步的输入是一个法语词元或者一个特殊的开始词元（例如`<bos>`），输出是一个法语词元或者一个特殊的结束词元（例如`<eos>`）。

### Encoder

假设有一个样本，输入序列是 $x_1, \ldots, x_T$，其中 $x_t$ 是输入序列的第 $t$ 个词元。  

#### 对每个时间步

在时间步 $t$，编码器将 $x_t$ 的输入向量 $\mathbf{x}_t$ 和上一个时间步的隐状态 $\mathbf{h}_{t-1}$ 作为输入，然后计算当前时间步的隐状态 $\mathbf{h}_t$，即

$$
\mathbf{h}_t = f(\mathbf{x}_t, \mathbf{h}_{t-1}).
$$

上图中使用 RNN 作为编码器，所以这里的 $f$ 函数就是 RNN 的计算方式。

#### 上下文变量

得到所有 $T$ 个时间步的隐状态后，我们需要得到一个能表示整个输入序列的向量，这个向量就是上下文变量 $c$，即

$$
\mathbf{c} = q(\mathbf{h}_1, \ldots, \mathbf{h}_T).
$$

这里 $q$ 函数可以是任意函数，例如可以直接选择最后一个时间步的隐状态，即 $\mathbf{c} = \mathbf{h}_T$，正如上图所示，最后一个时间步的输出作为上下文变量被送入解码器的每个时间步。

### Decoder


来自训练数据集的输出序列$y_1, y_2, \ldots, y_{T'}$，
对于每个时间步$t'$（与输入序列或编码器的时间步$t$不同），
解码器输出$y_{t'}$的概率取决于先前的输出子序列
$y_1, \ldots, y_{t'-1}$和上下文变量$\mathbf{c}$，
即$P(y_{t'} \mid y_1, \ldots, y_{t'-1}, \mathbf{c})$。

根据上面 seq2seq 的图片，我们直接使用 Encoder 的最后一个时间步的隐状态作为上下文变量，Decoder 也选用 RNN，**这就要求编码器和解码器具有相同数量的层和隐藏单元**。

#### 对每个时间步

在时间步 $t'$，解码器将 $y_{t'-1}$ 的输入向量 $\mathbf{y}_{t'-1}$、上一个时间步的隐状态 $\mathbf{s}_{t'-1}$ 和上下文变量 $\mathbf{c}$ 作为输入，然后计算当前时间步的隐状态 $\mathbf{s}_{t'}$，即

$$
\mathbf{s}_{t'} = g(\mathbf{y}_{t'-1}, \mathbf{s}_{t'-1}, \mathbf{c}),
$$

得到隐状态后，我们可以通过全连接层和 softmax 函数来计算当前时间步输出的概率分布，得到输出的词元 $y_{t'}$。

直到输出一个特殊的结束词元（例如`<eos>`）或者达到一个最大的输出长度，这样就完成了解码过程。

### 一些训练细节

在训练时，不同长度的序列经过 padding 填充后可以以相同形状的小批量加载，但是在训练时，我们应将填充词元的预测排除在损失计算之外，我们可以使用遮蔽变量（mask variable）来消除填充词元的预测。

```py
def sequence_mask(X, valid_len, value=0):
    """在序列中屏蔽不相关的项"""
    maxlen = X.size(1)
    mask = torch.arange((maxlen), dtype=torch.float32,
                        device=X.device)[None, :] < valid_len[:, None]  # 这里 None 是一种广播机制的小技巧
    X[~mask] = value
    return X

X = torch.tensor([[1, 2, 3], [4, 5, 6]])
sequence_mask(X, torch.tensor([1, 2]))
# tensor([[1, 0, 0],
#         [4, 5, 0]])

```

这里不得不感叹一下 PyTorch 的计算图机制，居然直接这么遮蔽也可以正确计算梯度。

同时在训练时，我们可以使用 **强制教学**（teacher forcing）的方法，即使用真实的输出序列，经过嵌入层后作为 Decoder 的输入，而不是使用上一个时间步的输出作为解码器的输入，这样可以加速训练过程，但是在预测时，我们需要使用上一个时间步的输出经过嵌入层后作为解码器的输入。

<img alt="encoder-decoder details" src="https://img.foril.space/encoder-decoder details.svg" width=400px style="display: block; margin:10px auto"/>

此外，对于一个 batch 批次中序列长度不同的数据，我们需要通过 `pad_sequence`、`pack_padded_sequence` 和 `pad_packed_sequence` 来进行处理：

1. **pad_sequence**: 这是处理一批不同长度的序列的第一步。在这一步中，您对每个序列进行填充（padding），使得这一批次中的所有序列长度相同。通常，这是通过在序列的末尾添加特殊的填充标记（如0）来实现的。在 PyTorch 里面一般是使用 DataLoader 进行数据加载，返回 mini-batch 形式的数据，再将此数据喂给网络进行训练。我们一般会自定义一个 collate_fn 函数，完成对数据的填充。

2. **pack_padded_sequence**: 在进行了填充之后，接下来使用`pack_padded_sequence`对这些填充后的序列进行打包。这一步是关键，因为它告诉LSTM网络哪些是序列的实际数据部分，哪些是填充部分。LSTM处理时将只关注非填充部分，这有助于提高效率，并且防止模型学习到与任务无关的信息。

3. **LSTM处理**: 经过打包的数据送入LSTM层进行处理。

4. **pad_packed_sequence**: 在LSTM处理之后，使用`pad_packed_sequence`对输出进行解包。这将输出恢复到填充前的维度，包括填充部分。这一步是为了使得数据能够以统一的格式进行后续处理，例如通过一个全连接层。

5. **后续处理**: 对LSTM的输出进行进一步的处理，比如通过全连接层和激活函数等。

这个流程确保了即使是在处理不同长度的序列时，神经网络也能有效地工作，同时避免了在模型训练时学习到无关的填充数据。

***

以上就是我个人对于 RNN 的一些理解的总结，以便日后查阅，个人理解比较浅薄，对于深层次的梯度计算和训练细节没有太多涉及，难免会有错误的地方，欢迎指正。

## 参考

- [循环神经网络](https://zh.d2l.ai/chapter_recurrent-neural-networks/index.html)
- [现代循环神经网络](https://zh.d2l.ai/chapter_recurrent-modern/index.html)
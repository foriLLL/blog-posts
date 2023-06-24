---
description: "PyTorch 中有很多操作需要记忆，这里对 repeat 和 repeat_interleave 做一个简单的记录。"
time: 2023-05-06
---

```py
x = torch.tensor([1,2,3])
# x.shape = torch.Size([3])

x.repeat(2, 1)
# tensor([[1, 2, 3],
#         [1, 2, 3]])
# repeat后的 shape = torch.Size([2, 3])

x.repeat(4,2,1)
# tensor([[[1, 2, 3],
#          [1, 2, 3]],
#         [[1, 2, 3],
#          [1, 2, 3]],
#         [[1, 2, 3],
#          [1, 2, 3]],
#         [[1, 2, 3],
#          [1, 2, 3]]])
# repeat后的 shape = torch.Size([4, 2, 3])
```
可以看出 `torch.repeat(*sizes)` 可以理解为先从最低维repeat，然后逐维度向上repeat，直到repeat到最高维度。这样的话，`torch.repeat(*sizes)` 的参数 `*sizes` 就可以理解为一个从最低维度开始的repeat次数的数组。

```py
x = torch.rand(3,2)
# x.shape = torch.Size([3, 2])
x.repeat(4,2,1) # *sizes 的维数不能小于 x 的维数

#   [3,2]   x.shape
#    | |
# [4,2,1]   x.repeat(4,2,1) 
#    | |
# [4,6,2]   x.repeat(4,2,1).shape
```

## repeat_interleave

在 PyTorch 中，`repeat()` 和 `repeat_interleave()` 函数都用于在张量中重复元素，但是它们的行为有所不同。

`repeat()` 方法会将整个张量重复指定的次数，返回一个新的张量。例如，如果我们有一个形状为 `(2, 3)` 的张量 `x`，并且我们调用 `x.repeat(2, 1)`，那么将得到一个形状为 `(4, 3)` 的新张量，其中原始的 `x` 张量被按行方向重复了两次。

`repeat_interleave()` 方法与 `repeat()` 不同，它将 **沿着指定的维度重复张量**，并将重复的元素插入到一个新的轴中。例如，如果我们有一个形状为 `(2, 3)` 的张量 `x`，并且我们调用 `x.repeat_interleave(2, dim=0)`，那么将得到一个形状为 `(4, 3)` 的新张量，其中沿着行方向的元素被重复了两次，并插入到新的第一维度中。

因此，`repeat()` 方法重复整个张量，而 `repeat_interleave()` 方法将重复的元素插入到一个新的轴中。具体使用哪种方法取决于你的具体需求。

```py
x torch.tensor([[1, 2, 3],
                [4, 5, 6]])

x.repeat_interleave(torch.tensor([2,3]),dim=0) # repeat_interleave 的 repeats参数可以是int或者tensor
# 在指定的维度上重复，如果是int，则重复次数为int，如果是tensor，则对应每个元素重复的次数为tensor的值，tensor的长度必须与指定的维度长度一致（可以广播）

# tensor([[1, 2, 3],
#         [1, 2, 3],    第0维第一个元素重复了 2 次
#         [4, 5, 6],
#         [4, 5, 6],
#         [4, 5, 6]])   第0维第二个元素重复了 3 次

x.repeat_interleave(torch.tensor([2]),dim=1)
# tensor([[1, 1, 2, 2, 3, 3],
#         [4, 4, 5, 5, 6, 6]])

x.repeat_interleave(torch.tensor([2,1,3]),dim=1)
# tensor([[1, 1, 2, 3, 3, 3],
#         [4, 4, 5, 6, 6, 6]])
```


## 区别

我们可以从 `repeat` 和 `repeat_interleave` 这两个函数名本身就能够看出它们的不同之处。

`repeat` 的意思是“重复”，它强调的是对整个张量进行重复操作。这与该函数的行为相符，因为它重复整个张量而不插入新的轴。

而 `repeat_interleave` 的意思是“重复插入”，它强调的是将元素插入到新的轴中。这也与该函数的行为相符，因为它在重复元素时会将它们插入到一个新的轴中。

因此，从函数名上看，`repeat` 和 `repeat_interleave` 的区别很明显，它们的行为也与它们的名字相符。
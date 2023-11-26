---
description: 有关使用 pip install 安装 Pypi 上的 wheel 包的一些问题，包括什么是 wheel、source distribution 与 built distribution 的区别、wheel 的优势等内容。
time: 2023-11-21T18:20:19+08:00
tags: 
heroImage: "https://img.foril.fun/Python wheel.png"
---

## sdist 与 bdist

在使用 pip 安装某些包时，我们往往会看到这样的提示：

```sh
$ python -m pip install 'uwsgi==2.0.*'
Collecting uwsgi==2.0.*
  Downloading uwsgi-2.0.18.tar.gz (801 kB)
     |████████████████████████████████| 801 kB 1.1 MB/s
Building wheels for collected packages: uwsgi
  Building wheel for uwsgi (setup.py) ... done
  Created wheel for uwsgi ... uWSGI-2.0.18-cp38-cp38-macosx_10_15_x86_64.whl
  Stored in directory: /private/var/folders/jc/8_hqsz0x1tdbp05 ...
Successfully built uwsgi
Installing collected packages: uwsgi
Successfully installed uwsgi-2.0.18
```

- 在第 3 行，它下载了一个名为 `uwsgi-2.0.18.tar.gz` 的 TAR 文件（tarball），该文件是用 gzip 压缩过的。
- 在第 6 行，它接受 tarball 并通过调用 `setup.py` 构建一个 `.whl` 文件。
- 在第 7 行，它将 wheel 标记为 `uWSGI-2.0.18-cp38-cp38-macosx_10_15_x86_64.whl`。
- 在第 10 行，它在构建 wheel 之后安装实际的包。

这里的 `.tar.gz` 文件就是 Source Distribution（sdist）。

如果我们查看 Pypi 上的 [uWSGI 下载文件](https://pypi.org/project/uWSGI/#files) ，我们可以看到它只有 Source Distribution：

<img alt="uWSGI_pypi" src="https://img.foril.fun/uWSGI_pypi.png" width=600px style="display: block; margin:10px auto"/>

Source Distribution 包含源代码，不仅包括 Python 代码，还包括与包捆绑在一起的任何扩展模块（通常是 C 或 C++ 语言）的源代码。对于 sdist，一些扩展模块是在用户端而不是开发人员端编译的。

而对于一些有 wheel 包的包，我们可以看到它有 Source Distribution 和 Built Distribution：

<img alt="chardet_pypi" src="https://img.foril.fun/chardet_pypi.png" width=600px style="display: block; margin:10px auto"/>

这里的 `.whl` 文件就是 Built Distribution（bdist），如果使用 pip 安装也可以看出明显不同：如果有合适的 wheel 包，pip 会直接安装 wheel 包，而不是下载源代码分发包（source distribution）然后在本地编译它。

```sh
$ python -m pip install 'chardet==3.*'
Collecting chardet
  Downloading chardet-3.0.4-py2.py3-none-any.whl (133 kB)
     |████████████████████████████████| 133 kB 1.5 MB/s
Installing collected packages: chardet
Successfully installed chardet-3.0.4
```

如果使用 pip 安装 Python 包时没有找到对应平台的预编译 wheel 文件，pip 通常会回退到下载源代码分发包（source distribution），然后在本地编译。

## 什么是 wheel

Python `.whl` 文件本质上是一个 `ZIP (.ZIP)` 归档文件，带有一个特制的文件名，告诉安装程序 wheel 将支持哪些 `Python` 版本和平台。

wheel 是一种 bdist。意味着 wheel 可以随时被安装，并允许跳过 sdist 所需的构建阶段。

### wheel 的命名规则

wheel文件名被分解成用连字符分隔的部分:

```
{dist}-{version}(-{build})?-{python}-{abi}-{platform}.whl

比如 cryptography-2.9.2-cp35-abi3-macosx_10_9_x86_64.whl
```

- `Cryptography` 是包名。

- `2.9.2` 是密码学的包版本。版本是符合 PEP 440 的字符串，如 `2.9.2`、`3.4` 或 `3.9.0.3.a3`。
- `cp35` 是 Python 标签，表示 wheel 所需的 Python 实现和版本。cp 代表 CPython, Python 的参考实现，而 35 表示 Python 3.5。例如，这个轮子与 `Jython` 不兼容。
- `abi3` 是 ABI 标记。ABI 代表应用程序二进制接口。实际上不需要担心它需要什么，但是 abi3 是 Python C API 的二进制兼容性的单独版本。
- `Macosx_10_9_x86_64` 是平台标记，它很拗口。在这种情况下，它可以进一步细分为子部分:
    - `macosx` 是 macOS 操作系统。
    - `10_9` 是 macOS developer tools SDK 版本，用于编译 Python，从而构建此 wheel。
    - `x86_64` 是对 x86-64 指令集体系结构的引用。


## wheel 的优势

- 对于纯 Python 包和扩展模块，wheels 的安装速度都比源代码发行版快。
- 比 sdist小。
- 车轮排除了 `setup.py` 的影响，不需要运行 `setup.py`。
- 不需要编译器。
- pip 会自动在 wheel 中生成匹配正确 Python 解释器的 `.pyc` 文件。
- 车轮提供了一致性（比如排除了变量的影响）。

## 参考

- [real python](https://realpython.com/python-wheels/)
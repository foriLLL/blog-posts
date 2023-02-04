> Go is expressive, concise, clean, and efficient. Its concurrency mechanisms make it easy to write programs that get the most out of multicore and networked machines, while its novel type system enables flexible and modular program construction. Go compiles quickly to machine code yet has the convenience of garbage collection and the power of run-time reflection. It's a fast, statically typed, compiled language that feels like a dynamically typed, interpreted language.

不同于其他语言，go中没有项目的说法，只有包, 其中有两个重要的路径，GOROOT 和 GOPATH

Go开发相关的环境变量如下：

GOROOT：GOROOT就是Go的安装目录，（类似于java的JDK）
GOPATH：GOPATH是我们的工作空间,保存go项目代码和第三方依赖包
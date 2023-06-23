当我们自己开发的Java程序希望通过命令行(CLI)调用时，可以自己在main函数的参数args中解析，但这样的做法极其复杂且常常缺乏鲁棒性。为了不在这里浪费时间，JCommander应运而生。  
> "Because life is too short to parse command line parameters" 

## 什么是JCommander
JCommander是Java解析命令行参数的工具，能够帮助我们快速开发CLI程序，Java在命令行交互的应用，还有很多工具。另一个使用比较广泛的是[Apache Commons CLI](http://commons.apache.org/proper/commons-cli/index.html)，它比JCommander支持更多的命令行风格，但是扩展能力不够。  

JCommander特点主要有：
* **注解驱动**  
它的核心功能**命令行参数定义**是基于注解的，这也是选择用它的主要原因。我们可以轻松做到命令行参数与属性的映射，属性除了是String类型，还可以是Integer、Boolean，甚至是File、集合类型。
* **功能丰富**  
它同时支持 `POSIX` 和 `Java` 两种风格的两种命令行风格，并且提供了输出帮助文档的能力(`usage()`)，还提供了国际化的支持。
* **高度扩展**  
可高度自定义所需要的各类选项以及对域的限制。

## 基本使用
一般来说，你只需要给你的fields加入注解，就可以使用JCommander来解析命令行：定义一个类，给有需要的field加入`@Parameter`注解。
```java
import com.beust.jcommander.Parameter;

public class Args {
  @Parameter
  private List<String> parameters = new ArrayList<>();

  @Parameter(names = { "-log", "-verbose" }, description = "Level of verbosity")
  private Integer verbose = 1;

  @Parameter(names = "-groups", description = "Comma-separated list of group names to be run")
  private String groups;

  @Parameter(names = "-debug", description = "Debug mode")
  private boolean debug = false;
}
```
然后只需要让JCommander去解析：
```java
class Main {
    @Parameter(names={"--length", "-l"})
    int length;
    @Parameter(names={"--pattern", "-p"})
    int pattern;

    public static void main(String ... argv) {
        Main main = new Main();
        JCommander.newBuilder()
            .addObject(main)
            .build()
            .parse(argv);
        main.run();
    }

    public void run() {
        System.out.printf("%d %d", length, pattern);
    }
}
```

## 参数类型
代表参数的字段可以是任何类型。基本类型（Integer、Boolean等……）默认支持，您可以编写**类型转换器**来支持任何其他类型（File等……）。
### Boolean
对于Boolean类型的域，JCommander默认其参数数量（arity）为0，就是说只要有这个参数就默认为true，没有就为false。
```java
@Parameter(names = "-debug", description = "Debug mode")
private boolean debug = false;
```
如果向指定这个参数默认为出，可以将他的参数数量设置为1，这样就需要明确指出这个参数值的值才可以运行。
```java
@Parameter(names = "-debug", description = "Debug mode", arity = 1)
private boolean debug = true;
```
需要使用`program -debug true`或`program -debug false`来调用。

> 当被装饰的域的类型是String、Integer、int、Long或者long时，JCommander会自动将值转化为对应的类型。当传入的参数不能被转化为对应类型时，会抛出异常。

### Lists
当域的类型是List时，JCommander会将其转为一个可出现多次的选项。比如：
```java
@Parameter(names = "-host", description = "The host")
private List<String> hosts = new ArrayList<>();
```
可以使用以下命令来向hosts传入两个值：
```java
java Main -host host1 -verbose -host host2
```
### 密码
如果您的参数之一是密码或您不希望在命令行历史记录中显示的其他值，则可以将其声明为密码类型，然后 JCommander 将要求您在控制台中输入它。
```java
public class ArgsPassword {
  @Parameter(names = "-password", description = "Connection password", password = true)
  private String password;
}
```
当你在程序中运行时，你会得到如下的提示：
```sh
Value for -password (Connection password):
```
需要在输入密码后才能继续，在Java5及以前版本中，密码会被回显，需要设置`echoInput=false`来取消回显；而在Java6及之后版本中密码默认不会显示。

## 自定义类型
### Single value
JCommander提供IStringConverter和IParameterSplitter两个接口来将参数绑定到自定义类型或更改JCommander拆分参数的方式（默认为逗号拆分）。  
很多时候，我们的应用程序需要通过命令行解析更复杂的类型（例如文件、主机名、列表等）。为此，您可以通过实现以下接口来编写类型转换器：
```java
public interface IStringConverter<T> {
  T convert(String value);
}
// 例如
public class FileConverter implements IStringConverter<File> {
  @Override
  public File convert(String value) {
    return new File(value);
  }
}
```
然后只需要在对应的域上加上注解，并加入converter属性。
```java
@Parameter(names = "-file", converter = FileConverter.class)
File file;
```
### List value
如果是一个List域，默认会调用List<>中的泛型类的converter，并以逗号为分隔符，将字符串拆分后一个个作为输入得到List。
```java
@Parameter(names = "-files", converter = FileConverter.class)
List<File> files;
```
应该使用如下命令调用。
```sh
$ java App -files file1,file2,file3
```
也可以IStringConverter中的泛型改为List，例如`IStringConverter<List<File>>`，然后手动解析字符串中的逗号等分隔符，然后加入List中返回。

如果觉得在每一个域上都需要加入converter很麻烦，可以用ConverterFactory在JCommander实例上加入一个实现了指定接口的工厂类实例，详细看[这里](http://jcommander.org/#single-value)。

### 分隔符
给`@Parameter`的`splitter=`属性传入一个实现了`IParameterSplitter`接口的类，可以自定义分隔符。
```java
public interface IParameterSplitter {
  List<String> split(String value);
}
```
如下是一个以分号为分隔符的接口实现：
```java
public static class SemiColonSplitter implements IParameterSplitter {
    public List<String> split(String value) {
      return Arrays.asList(value.split(";"));
    }
}

/************/

@Parameter(names = "-files", converter = FileConverter.class, splitter = SemiColonSplitter.class)
List<File> files;
```
## 参数校验
### 单个参数校验
单个参数校验用于那些指定参数有一定限制的情况，比如用Integer记录年龄必须为非负数，String的长度必须小于10等限制。  
只需要在`@Paramter`注解中加入`validateWith=`属性，传入一个实现了`IParameterValidator`接口的类即可。
```java
// 接口
public interface IParameterValidator {
 /**
   * Validate the parameter.
   *
   * @param name The name of the parameter (e.g. "-host").
   * @param value The value of the parameter that we need to validate
   *
   * @throws ParameterException Thrown if the value of the parameter is invalid.
   */
  void validate(String name, String value) throws ParameterException;
}

/************/

// 实现
public class PositiveInteger implements IParameterValidator {
 public void validate(String name, String value)
      throws ParameterException {
    int n = Integer.parseInt(value);
    if (n < 0) {
      throw new ParameterException("Parameter " + name + " should be positive (found " + value +")");
    }
  }
}

/************/

// 调用
@Parameter(names = "-age", validateWith = PositiveInteger.class)
private Integer age;

@Parameter(names = "-count", validateWith = { PositiveInteger.class, CustomOddNumberValidator.class })
private Integer value;
```
### 全局参数验证
全局参数验证用于多个参数之间的限制关系，比如不能同时出现的参数等情况。这种情况下就不能单纯利用注解中的参数验证来实现，需要自行通过获得参数后在代码中加入限制。

## 主参数
到目前为止，我们看到的所有`@Parameter`注解都定义了一个名为`names`的属性。您可以定义一个（**最多一个**）参数而不使用任何此类属性。此参数可以是列表或单个字段（例如字符串或具有转换器的类型，例如文件），在这种情况下，只需要一个主要参数。
```java
@Parameter(description = "Files")
private List<String> files = new ArrayList<>();

@Parameter(names = "-debug", description = "Debugging level")
private Integer debug = 1;
```
这样可以允许你解析以下命令：
```bash
$ java Main -debug file1 file2
```
files域就会得到`file1`和`file2`两个字符串。

## 参数分隔符
默认情况下，参数由空格分隔，但您可以更改此设置以允许使用不同的分隔符：
```java
@Parameters(separators = "=")
public class SeparatorEqual {
  @Parameter(names = "-level")
  private Integer level = 2;
}
```
之后便可以使用
```bash
$ java Main -log:3
$ java Main -level=42
```
## 帮助属性
如果您的参数之一用于显示一些帮助或用法，则需要使用`help`属性：
```java
@Parameter(names = "--help", help = true)
private boolean help;
```
如果您省略此布尔值，JCommander将在尝试验证您的命令并发现您未指定某些必需参数时发出错误消息。

## 使用文档
您可以在用于解析命令行的 JCommander 实例上调用`.usage()`以生成程序理解的所有选项的摘要：
```bash
Usage: <main class> [options]
  Options:
    -debug          Debug mode (default: false)
    -groups         Comma-separated list of group names to be run
  * -log, -verbose  Level of verbosity (default: 1)
    -long           A long number (default: 0)
```
您可以通过在 JCommander 对象上调用`.setProgramName()`来自定义程序的名称。前面有星号的选项是必需的。您还可以通过设置`@Parameter`注解的`order`属性来指定调用`.usage()`时每个选项的显示顺序：
```java
class Parameters {
    @Parameter(names = "--importantOption", order = 0)
    private boolean a;

    @Parameter(names = "--lessImportantOption", order = 3)
    private boolean b;
```

## 参考
* https://blog.csdn.net/adalf90/article/details/80492795
* http://jcommander.org/#_overview
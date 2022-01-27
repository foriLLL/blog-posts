Java中并没有内置JSON的解析，因此使用JSON需要借助第三方类库。

下面是几个常用的 JSON 解析类库：

* Gson: 谷歌开发的 JSON 库，功能十分全面。
* FastJson: 阿里巴巴开发的 JSON 库，性能十分优秀。
* Jackson: 社区十分活跃且更新速度很快。

以下内容基于 `Jackson` 。

## Jackson简介
Jackson 是当前用的比较广泛的，用来序列化和反序列化 json 的 Java 的开源框架。Jackson 社 区相对比较活跃，更新速度也比较快， 从 Github 中的统计来看，Jackson 是最流行的 json 解析器之一 。 Spring MVC 的默认 json 解析器便是 Jackson。 Jackson 优点很多。 Jackson 所依赖的 jar 包较少 ，简单易用。与其他 Java 的 json 的框架 Gson 等相比， Jackson 解析大的 json 文件速度比较快；Jackson 运行时占用内存比较低，性能比较好；Jackson 有灵活的 API，可以很容易进行扩展和定制。

Jackson 的核心模块由三部分组成：
1. jackson-core，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。 Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。
2. jackson-annotations，注解包，提供标准注解功能。
3. jackson-databind ，数据绑定包， 提供基于"对象绑定" 解析的相关 API （ ObjectMapper ） 和"树模型" 解析的相关 API （JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。

> jackson-databind 依赖 jackson-core 和 jackson-annotations，当添加 jackson-databind 之后， jackson-core 和 jackson-annotations 也随之添加到 Java 项目工程中。在添加相关依赖包之后，就可以使用 Jackson。

## 依赖安装
在 Maven 构建的项目中，在 pom.xml 文件中加入以下依赖即可。（在 `gradle.build` 中 `dependencies` 内直接粘贴Mavan依赖xml可自动转化为gradle形式）
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.5</version>
</dependency>
```
## 序列化：Java对象转换为JSON
Jackson 最常用的 API 就是基于"对象绑定" 的 `ObjectMapper`。

ObjectMapper 通过 writeValue 系列方法将 java 对象序列化为 json，并将 json 存储成不同的格式：String（writeValueAsString）、Byte Array（writeValueAsBytes）、Writer、File、OutStream、DataOutput。  
ObjectMapper 通过 readValue 系列方法从不同的数据源： String、Byte Array、Reader、File、URL、InputStream 中反序列化为 java 对象。

常用序列化API：
```java
writeValue()
writeValueAsString()
writeValueAsBytes()
```
以下是一个简单的例子：
```java
@Test
@DisplayName("jacksonTest")
public void jsonTest() throws IOException {
    ObjectMapper mapper = new ObjectMapper();
    List<Person> list = new ArrayList<>();
    list.add(new Person("foril", 22));

    String s = mapper.writeValueAsString(list);     //转换为String
    s = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(list);    //prettyPrint
    mapper.writeValue(new FileOutputStream("out/output.json"), list);   //写入文件
    byte[] bytes = mapper.writeValueAsBytes(list);  //转换为Bytes

    System.out.printf(s);
}
/**
* 实例所用类
*/
class Person {
    private String name;
    private Integer age;
    private Date birthday;

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public Person() {
    }

    public Person(String name, Integer age, Date date) {
        this.name = name;
        this.age = age;
        this.birthday = date;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

//output
// [ {
//   "name" : "foril",
//   "age" : 22
// } ]
```
## 反序列化：JSON转换为Java对象
使用 `readValue()` 方法来从JSON转换为Java对象：
```java
Person p = new Person("xk", 22, new Date());
s = mapper.writeValueAsString(p);
Person p2 = mapper.readValue(s, Person.class);  //从字符串转换

Reader reader = new StringReader(s);
p2 = mapper.readValue(reader, Person.class);    //从Reader转换

File f = new File("data/p.json");
p2 = mapper.readValue(f, Person.class);         //从文件转换

URL url = new URL("file:data/p.json");
p2 = mapper.readValue(url, Person.class);       //从网络文件地址转换

InputStream input = new FileInputStream("data/p.json");
p2 = mapper.readValue(input, Person.class);     //从流中转换

byte[] bytes = mapper.writeValueAsBytes(p);
p2 = mapper.readValue(bytes, Person.class);     //从字节数组中读取
```
> * 需要注意的是反序列化时需要对应的类有默认构造函数，Jackson通过默认构造函数以及setter来构造新的对象。  
> * 目前Jackson似乎不能正确处理内部类以及匿名类，详见[这里](https://stackoverflow.com/questions/28418564/jackson-deserialization-with-anonymous-classes)

### 反序列化为不同类型
```java
String s = mapper.writeValueAsString(list);

//转换为数组
Person[] people = mapper.readValue(s, Person[].class);

//转换为集合
List<Person> people  = mapper.readValue(s, new TypeReference<List<Person>>(){});

//转换为Map
Map<String, Object> jsonMap = mapper.readValue(s, new TypeReference<Map<String,Object>>(){});
```

## ObjectMapper配置
在调用 writeValue 或调用 readValue 方法之前，往往需要设置 ObjectMapper 的相关配置信息。这些配置信息应用 java 对象的所有属性上。示例如下：
```java
//在反序列化时忽略在 json 中存在但 Java 对象不存在的属性 
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES,
   false); 
//在序列化时日期格式默认为 yyyy-MM-dd'T'HH:mm:ss.SSSZ 
mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,false) 
//在序列化时忽略值为 null 的属性 
mapper.setSerializationInclusion(Include.NON_NULL); 
//忽略值为默认值的属性 
mapper.setDefaultPropertyInclusion(Include.NON_DEFAULT);
```
### 时间类型格式化
Jackson 默认会将java.util.Date对象转换成long值（时间戳）,同时也支持将时间转换成格式化的字符串。
```java
SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
objectMapper.setDateFormat(dateFormat);

String json = objectMapper.writeValueAsString(对象);
```

## 参考
* https://blog.csdn.net/psh18513234633/article/details/88599509
* https://blog.csdn.net/wangxuelei036/article/details/107360975/
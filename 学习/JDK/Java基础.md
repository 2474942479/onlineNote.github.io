# Java 基础

## 重写和重载

| 区别点     |  重载方法  | 重写方法                                       |
| :--------- | :--------: | :--------------------------------------------- |
| 发生范围   | 同一个类中 | 子类中                                         |
| 参数类型   |  必须不同  | 必须相同                                       |
| 返回类型   |    随意    | 必须相同                                       |
| 访问修饰符 |    随意    | 范围不能小于父类                               |
| 发生阶段   |   编译期   | 运行期                                         |
| 异常       |    随意    | 可以减少或删除，⼀定不能抛出新的或者更⼴的异常 |

**方法的重写要遵循“两同两小一大”**（以下内容摘录自《疯狂 Java 讲义》,[issue#892](https://github.com/Snailclimb/JavaGuide/issues/892) ）：

+ “两同”即方法名相同、形参列表相同；
+ “两小”指的是子类方法返回值类型应比父类方法返回值类型更小或相等，子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等；
+ “一大”指的是子类方法的访问权限应比父类方法的访问权限更大或相等。



##  深拷贝 vs 浅拷贝

1. **浅拷贝**：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝(增加了一个指针指向已存在的内存地址)

    仅仅是指向被复制的内存地址，如果原地址发生改变，那么浅复制出来的对象也会相应的改变

2. **深拷贝**：对基本数据类型进行值传递，对引用数据类型，增加了一个指针并且申请了一个新的内存，使这个增加的指针指向这个新的内存。

    在计算机中开辟一块**新的内存地址**用于存放复制的对象。

    使用深拷贝的情况下，释放内存的时候不会因为出现浅拷贝时释放同一个内存的错误

![deep and shallow copy](https://camo.githubusercontent.com/d6d8355e9c0cbde0bdf8a53d34b0e2b46bbaa5e7/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d372f6a6176612d646565702d616e642d7368616c6c6f772d636f70792e6a7067)

## Java三大特性

### 封装

封装把⼀个对象的属性私有化，同时提供⼀些可以被外界访问的属性的⽅法，如果属性不想被外界访 问，我们⼤可不必提供⽅法给外界访问。但是如果⼀个类没有提供给外界访问的⽅法，那么这个类也没 有什么意义了。

### 继承 

继承是使⽤已存在的类的定义作为基础建⽴新类的技术，新类的定义可以增加新的数据或新的功能，也 可以⽤⽗类的功能，但不能选择性地继承⽗类。通过使⽤继承我们能够⾮常⽅便地复⽤以前的代码。

 **关于继承如下 3 点请记住：** 

1. **⼦类拥有⽗类对象所有的属性和⽅法**（包括私有属性和私有⽅法），但是⽗类中的私有属性和⽅ 法⼦类是**⽆法访问，只是拥有**。
2. ⼦类可以拥有⾃⼰属性和⽅法，即⼦类可以对⽗类进⾏扩展。
3. ⼦类可以⽤⾃⼰的⽅式实现⽗类的⽅法。

### 多态

所谓多态就是指程序中定义的引⽤变量所指向的具体类型和通过该引⽤变量发出的⽅法调⽤在编程时并 不确定，⽽是在程序运⾏期间才确定，即⼀个引⽤变量到底会指向哪个类的实例对象，该引⽤变量发出 的⽅法调⽤到底是哪个类中实现的⽅法，必须在由程序运⾏期间才能决定。 

**在 Java 中有两种形式可以实现多态：继承（多个⼦类对同⼀⽅法的重写）和接⼝（实现接⼝并覆盖接 ⼝中同⼀⽅法）。**

## == 与 equals(重要) 

**==** : 它的作⽤是判断两个对象的地址是不是相等。即，判断两个对象是不是同⼀个对象(基本数据类型 WX⽐᫾的是值，引⽤数据类型WX⽐᫾的是内存地址)。 

**equals()** : 它的作⽤也是判断两个对象是否相等。但它⼀般有两种使⽤情况： 

1. 类没有覆盖 equals() ⽅法。则通过 equals() ⽐᫾该类的两个对象时，等价于通过 “WX”⽐᫾这两个对象。 

2. 类覆盖了 equals() ⽅法。⼀般，我们都覆盖 equals() ⽅法来⽐᫾两个对象的内容是 否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

    ```java
    public class test1 {
        public static void main(String[] args) {
            String a = new String("ab"); // a 为⼀个引⽤
            String b = new String("ab"); // b为另⼀个引⽤,对象的内容⼀样
            String aa = "ab"; // 放在常量池中
            String bb = "ab"; // 从常量池中查找
            
            if (a != b) // true，⾮同⼀对象
                System.out.println("a!=b");
            if (a.equals(b)) // true
                System.out.println("aEQb");
            if (aa == bb) // true
                System.out.println("aa==bb");
            if (a != aa) { // true
                System.out.println("a!=aa");
            }
        }
    }
    ```

**说明**： 

* String 中的 equals ⽅法是被重写过的，因为 object 的 equals ⽅法是比较的对象的内存地址，⽽ String 的 equals ⽅法比较的是对象的值。 

* 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引⽤。如果没有就在常量池中重新创建⼀个 String 对象。

##  `hashCode 与 equals (重要)

重写equals方法，为什么必须重写hashCode 方法？

> 重写 hashCode 方法和equals方法，一般是使用到了散列表这一类数据结构，因为这一类数据结构不允许存在重复的键，在添加值时，会先计算元素hashCode 值来确定元素在集合中的位置，进而判断集合中是否存在相同hashCode 的元素，如果集合里边不存在该值，可以直接插入进去。如果已经存在，则需要再次通过equals()来比较，判断是否为同一个对象(值相同即为同一个对象)，相同对象则不可以插入，如果不同的话，就会重新散列到其他位置(发生hash冲突)。这样我们就⼤⼤减少了 equals 的次 数，相应就⼤⼤提⾼了执⾏速度(不需要每次都进行equals比较)。
>
> **如果不重写hashcode方法，会导致equals相同的对象(值相同)，但是hashcode不相同，放入HashMap中，违反不可重复性。重写hahscode方法，确保equals相同的对象，hashcode一定相同，保证散列表的不可重复性。**
>
> `Object.hashcode()`源码结论
>
> + java6、7默认是返回随机数
> + java8默认是通过和当前线程有关的一个随机数+三个确定值，运用Marsaglia’s xorshift scheme随机数算法得到的一个随机数
>
> ### hashCode（）与 equals（）的相关规定
>
> 1. 如果两个对象相等，则 hashcode 一定也是相同的
> 2. 两个对象相等,对两个对象分别调用 equals 方法都返回 true
> 3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
> 4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
> 5. hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

## 异常处理之try-catch-finally 

* **try 块**： ⽤于捕获异常。其后可接零个或多个 catch 块，如果没有 catch 块，则必须跟⼀个 finally 块。 

* **catch 块**： ⽤于处理 try 捕获到的异常。 

* **finally 块**： ⽆论是否捕获或处理异常，finally 块⾥的语句都会被执⾏。当在 try 块或 catch 块中遇到 return 语句时，finally 语句块将在⽅法返回之前被执⾏。 

    在以下 4 种特殊情况下，finally 块不会被执⾏： 

    1. 在 finally 语句块第⼀⾏发⽣了异常。 因为在其他⾏，finally 块还是会得到执⾏ 
    2. 在前⾯的代码中⽤了 `System.exit(int)`已退出程序。 exit 是带参函数 ；若该语句在异常语句 之后，finally 会执⾏ 
    3. 程序所在的线程死亡。 
    4. 关闭 CPU。

    **注意**： 当 try 语句和 finally 语句中都有 return 语句时，在⽅法返回之前，finally 语句的内容 将被执⾏，并且 finally 语句的返回值将会覆盖原始的返回值。

## String类

### new Sting("abc") 和 String a = "abc" 分别创建了几个对象?

new String("abc")相当于new String(String s1="abc")，即先要执行String s1="abc"，然后再在堆区new一个String对象。 因此，现在可以解答本文的标题了，String s=new String("abc")创建了1或2个对象，String s="abc"创建了0或1个对象

**原理: **   如果字符串常量池已经存在某个字符串，那么new String(abc)创建一个堆中的对象，该对象指向常量池中的字符串，此时创建一个对象。如果不存在某个字符串，先在栈上创建一个字符串引用对象，然后在堆上创建一个对象，这两个都指向字符串常量池中的字符串，此时创建了两个对象.

**知识点**

* **String 对象引用不可以改变。对象内容可以改变**

* **String 对象一旦被创建就是固定不变的了，对 String 对象的任何改变都不影响到原对象，相关的任何变化性的操作都会生成新的对象。String 对象每次有变化性操作的时候，都会从新 new 一个 String 对象（这里指的是有变化的情况）。**







# 拦截器

### 接入后台端(SSO)登入拦截后, 如何获取当前登入用户的信息?

 通过拦截器获取header中的`userinfo`的value 再进行解码 然后`json`反序列化为实体类对象



## 异步调用

使用`Future`获得异步执行结果时，要么调用阻塞方法`get()`，要么轮询看`isDone()`是否为`true`，这两种方法都不是很好，因为主线程也会被迫等待。从Java 8开始引入了`CompletableFuture`，它针对`Future`做了改进，可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法

### 使用`CompletableFuture.suppplyAsync();`

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 创建异步执行任务:
        CompletableFuture<Double> cf = CompletableFuture.supplyAsync(Main::fetchPrice);
        // 如果执行成功:
        cf.thenAccept((result) -> {
            System.out.println("price: " + result);
        });
        // 如果执行异常:
        cf.exceptionally((e) -> {
            e.printStackTrace();
            return null;
        });
        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
        System.out.println("异步调用完毕!");
        System.out.println("2"+LocalTime.now());
        Thread.sleep(200);
        System.out.println("结束了!");
        System.out.println("3"+LocalTime.now());
    }

    static Double fetchPrice() {
       System.out.println("异步调用开始!");
       System.out.println("0"+LocalTime.now());
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        if (Math.random() < 0.3) {
            throw new RuntimeException("fetch price failed!");
        }
System.out.println("1"+LocalTime.now());
        return 5 + Math.random() * 20;
       
    }
}

```

# `FastJson  `

```java
@Data
public class UserDTO {

    @JsonField(name = "user_name")
    public String userName;	
    
    public String orderId;
}


// 解析json字符串 带有@JsonField注解的 有name则 根据name进行匹配  否则智能匹配 
//注意:加上@JsonField注解 高版本可能导致反序列化时 字段匹配不到

String jsonRestultStr = "{\"user_name\":\"zhangsan\",\"orderid\":22}";
// 反序列化
UserDTO userDTO = JSON.parse(jsonRestultStr, UserDTO.class);

// 序列化
JSON.toJSONString(userDTO);
```



## `Hutool`工具包

### 文件工具类 `FIileUtil  `

​	获取后缀名: `FileUtil.extName()`

### Excel 工具类

> **一次性读取数据**
>
> ```java
> try {
>       ExcelReader reader = ExcelUtil.getReader(file.getInputStream(), sheetIndex);
> 
>  	if (reader.getRowCount() > maxRowCount) {
>              return JsonResult.failure(ErrorCode.BUSINESS_ERROR, "备注信息大于1000条,请修改后重试");
>          }
> 
>       orderRemarkExcelDTOList = reader.readAll(OrderRemarkExcelDTO.class);
> 
>   } catch (Exception e) {
>       e.printStackTrace();
>       return JsonResult.failure(ErrorCode.PARAM_ERROR, "excel解析错误");
>   }
> ```
>
> **分批读取采用 `EasyExcel`**

# 其他常用技能

### `hashmap.computeIfAbsent(K key, Function remappingFunction)`

​	computeIfAbsent() 方法对 hashMap 中指定 key 的值进行重新计算，如果该key对应的值存在,直接返回map中这个key对应的值,  如果不存在，则 以key为参数 进行重新计算后 添加到 hasMap(key:"key重新计算后的值") 中。

实例:	`map.computeIfAbsent("key", k -> new Object());`

```java
 public static void main(String[] args) {
        // 创建一个 HashMap
        HashMap<String, Integer> prices = new HashMap<>();

        // 往HashMap中添加映射项
        prices.put("Shoes", 200);
        prices.put("Bag", 300);
        prices.put("Pant", 150);
        System.out.println("HashMap: " + prices);

        // 计算 Shirt 的值
        int shirtPrice = prices.computeIfAbsent("Shirt", key -> 280);
        System.out.println("Price of Shirt: " + shirtPrice);

        // 输出更新后的HashMap
        System.out.println("Updated HashMap: " + prices);
    }
输出:
HashMap: {Pant=150, Bag=300, Shoes=200}
Price of Shirt: 280
Updated HashMap: {Pant=150, Shirt=280, Bag=300, Shoes=200}
```



### `StringUtils.isNotBlank(str)` 

​	判断输入的字符串不为null 且 长度大于0 且不为 空白字符 (删除空白字符后长度大于0)

> ```java
> isNotEmpty(str) 等价于 str != null && str.length > 0
> // str.trim() 方法用于删除字符串的头尾空白符
> isNotBlank(str) 等价于 str != null && str.length > 0 && str.trim().length > 0  
> isEmpty 等价于 str == null || str.length == 0
> isBlank 等价于 str == null || str.length == 0 || str.trim().length == 0
> ```

### `Collections.singletonList(obj):`

​	这个方法主要用于只有一个元素的优化，减少内存分配，无需分配额外的内存，可以从`SingletonList`内部类看得出来,由于只有一个element,因此可以做到内存分配最小化，相比之下`ArrayList`的DEFAULT_CAPACITY(默认容量)=10个

### `Lists.newarraylist()和List list = new ArrayList()比较`

​	Lists和Maps是两个工具类,` Lists.newArrayList()`其实和`new ArrayList()`几乎一模一样, 唯一它帮你做的(其实是javac帮你做的), 就是自动推导(不是"倒")尖括号里的数据类型

### `BeanUtils.copyProperties(A,B)和Property.copyProperties(B,A) 将实体类B的值赋给A`

> `BeanUtils.copyProperties()`可以在一定范围内进行类型转换，同时还要注意一些不能转换时候，会将默认null值转化成0;
>
> `Property.copyProperties()`则是严格的类型转化，必须类型和属性名完全一致才转化。
>
> 对于null的处理：`PropertyUtils`支持为null的场景；`BeanUtils`对部分属性不支持null，具体如下：
>
> > 1. `java.util.Date`类型不支持,但是它的子类`java.sql.Date`是被支持的。`java.util.Date`直接copy会报异常；
> >
> > 2. Boolean，Integer，Long等不支持，会将null转化为0；
> >
> > 3. String支持，转化后依然为null。
>
> `BeanUtils`的高级功能`org.apache.commons.beanutils.Converter`接口可以自定义类型转化，也可以对部分类型数据的null值进行特殊处理，如`ConvertUtils.register(new DateConverter(null), java.util.Date.class);`但是`PropertyUtils`没有

### `@Validated` 校验注解配合以下注解使用

`@NotNull`：不能为null，但可以为empty

`@NotEmpty`：不能为null，而且长度必须大于0

`@NotBlank`：只能作用在String上，不能为null，而且调用trim()后，长度必须大于0

`@Min(value)`: 输入值大于等于value值

`@Max(value)`:输入小于等于value值

### 解决`BeanUtils`不能复制List,Map集合

1. for循环进行`BeanUtils.copyProperties()`

2. 使用`fastJson` 的 `Json.parseArray()`进行深复制   

   ```java
   List<People> source = new LinkedList();
   List<People> target = Json.parseArray(Json.toJsonString(List),People.class);
   
   
   public static <T> List copyList(List<T> list) {
       if (CollectionUtils.isEmpty(list)) {
           return new ArrayList();
       }
       return JSON.parseArray(JSON.toJSONString(list), list.get(0).getClass());
   }
   
   public static Map<String, Object> copyMap(Map map) {
       return JSON.parseObject(JSON.toJSONString(map));
   }
   ```

   

### MySQL

* 自定义排序

  *将获取出来的数据根据str1,str2,str3等的顺序排序*

  ```mysql
  order by filed(val, str1, str2, str3...)
  .last(String.format("order by field(id, %S)", Joiner.on(",").join(orderIdList)))
  ```

  

# Mybatis-Plus  

### 条件构造器中condition的使用

条件构造器所有条件有个重载的方法，第一个入参`boolean condition（`表示该条件**是否**加入最后生成的sql中，默认true）。

```
@Override
public Children eq(boolean condition, R column, Object val) {
    getWrapper().eq(condition, column, val);
    return typedThis;
}
```

```
Page<OrderLimitBuyDO> orderPage = lambdaQuery()
        .eq(StringUtils.isNotBlank(idcardNo), OrderLimitBuyDO::getIdcardNo, idcardNo)
        .page(new Page<>(queryDTO.getCurrent(), queryDTO.getSize()));
```

### 仅查询部分字段 select()   重复调用以最后一次为准

**注意：**使用普通查询 **list()** 返回的值依然是实体对象，只是除了id和name其他都是null。**而使用listMap()**来查询，这样会根据**查询语句中select的具体字段来赋值给map**

```java
QueryWrapper<Customer> qw = new QueryWrapper<>();
qw.select("id", "name");
List<Map<String, Object>> list = customerService.listMaps(qw);
```

### `LambdaQueryChainWrapper`（链式条件构造器查询）

## 服务调用成功  操作数据库无反应  

**注意:** 查看环境是否正确

# java开发规范

### 1. 浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用equals 来判断。

Java在java.math包中提供的API类BigDecimal，用来对超过16位有效位的数进行精确的运算。双精度浮点型变量double可以处理16位有效数，但在实际应用中，可能需要对更大或者更小的数进行运算和处理。一般情况下，对于那些不需要准确计算精度的数字，我们可以直接使用Float和Double处理，但是Double.valueOf(String) 和Float.valueOf(String)会丢失精度。所以开发中，如果我们需要精确计算的结果，则必须使用BigDecimal类来操作。

正例：

```java
 
(1) 指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的。
 
float a = 1.0f - 0.9f;
 
float b = 0.9f - 0.8f;
 
float diff = 1e-6f;
 
if (Math.abs(a - b) < diff) {
 
System.out.println("true");
 
}
 
(2) 使用 BigDecimal 来定义值，再进行浮点数的运算操作。
 
BigDecimal a = new BigDecimal("1.0");
 
BigDecimal b = new BigDecimal("0.9");
 
BigDecimal c = new BigDecimal("0.8");
 
BigDecimal x = a.subtract(b);
 
BigDecimal y = b.subtract(c);
 
if (x.equals(y)) {
 
System.out.println("true");
 
}
```

### 2. 为了防止精度损失，禁止使用构造方法 `BigDecimal(double)`的方式把 double 值转 化为 `BigDecimal `对象

说明：`BigDecimal(double)`存在精度损失风险，在精确计算或值比较的场景中可能会导致业务逻辑异常。

 如：`BigDecimal g = new BigDecimal(0.1f);` 实际的存储值为：0.10000000149

正例：优先推荐入参为 String 的构造方法，或使用 BigDecimal 的 valueOf 方法，此方法内部其实执行了 Double 的 toString，而 Double 的 toString 按 double 的实际能表达的精度对尾数进行了截断。

`BigDecimal recommend1 = new BigDecimal("0.1"); `

`BigDecimal recommend2 = BigDecimal.valueOf(0.1);`

### 3.  `javaBean pSkuId字段`   参数值丢失 

```java
@JsonProperty("pSkuId")
private String pSkuId;
```

 原因: 

​	未遵循驼峰命名法  序列化与反序列化时会自动转换成 `pskuId`  原本的Getting/Setting方法为 `getPSkuId / setPSkuId`  序列化后的 `pskuID` 匹配不到对应的get/set方法 导致参数无法赋值 



### 4. 正则的预编译主要注意两点：

1. Pattern 表达式要提前定义，不要再需要的地方临时定义；

2. Pattern 表达式要定义为 static final 静态变量，以避免执行多次预编译,  可以有效加快正则匹配速度。

   `private static final Pattern pattern = Pattern.compile(regexRule);`
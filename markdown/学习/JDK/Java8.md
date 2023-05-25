# 行为(方法)参数化传递代码

1.  自定义一个方法对数据进行操作, 入参为 **数据, lambda(行为)**
2. 通过 **泛型函数式接口** 使用lambda 具体实现函数式接口(行为)  
3. 调用自定义方法 ,将该**行为**作为参数进行传递  从而实现对数据的不同操作

# lambda表达式

## 泛型函数式接口

```java
package java.util.function;
```

![img](https://upload-images.jianshu.io/upload_images/5654620-54f3bb66b35cd761.png?imageMogr2/auto-orient/strip|imageView2/2/w/752/format/webp)



### Predicate <T>

入参 T 返回值 Boolean

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
}
```

### Consumer<T>  

入参 T 无返回值

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
}
```

### Function<T>

入参 T 返回值 R

```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
}
```

## 常用的函数式接口 

原始类型特化函数接口 防止自动拆装箱 消耗性能

| 函数式接口          | 函数描述符          | 原始类型特化                                                 |
| ------------------- | :------------------ | ------------------------------------------------------------ |
| `Predicate<T>`      | `T-> boolean`       | `IntPredicate,LongPredicate, DoublePredicate`                |
| `Consumer<T>`       | T -> void           | `IntConsumer,LongConsumer, DoubleConsumer`                   |
| `Function<T>`       | `T -> R`            | `IntFunction, IntToDoubleFunction, IntToLongFunction, LongFunction,` <br>`LongToDoubleFunction, LongToIntFunction, DoubleFunction,`<br>`ToIntFunction, ToDoubleFunction, ToLongFunction` |
| `Supplier<T>`       | `() -> T`           | `BooleanSupplier,IntSupplier, LongSupplier, DoubleSupplier`  |
| `UnaryOperator<T>`  | `T -> T`            | `IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator`   |
| `BinaryOperator<T>` | `(T, T) -> T`       | `IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator` |
| `BiPredicate<L,R>`  | `(L, R) -> boolean` |                                                              |
| `BiConsumer<T,U>`   | `(T, U) -> void`    | `ObjIntConsumer, ObjLongConsumer, ObjDoubleConsumer`         |
| `BiConsumer<T,U,R>` | `(T, U) -> R`       | `ToIntBiFunction, ToLongBiFunction, ToDoubleBiFunction`      |

### 案例 

| 使用案例              | Lambda 的例子                                                | 对应的函数式接口                                             |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 布尔表达式            | `(List<String> list) -> list.isEmpty()`                      | `Predicate<List<String>>`                                    |
| 创建对象              | `() -> new Apple(10)`                                        | `Supplier<Apple>`                                            |
| 消费一个对象          | `(Apple a) -> System.out.println(a.getWeight())`             | `Consumer<Apple>`                                            |
| 从一个对象中选择/提取 | `(String s) -> s.length()`                                   | `Function<String, Integer> 或 ToIntFunction<String>`         |
| 两个值进行计算        | `(int a, int b) -> a * b`                                    | `IntBinaryOperator`                                          |
| 比较两个对象          | `(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())` | `Comparator<Apple> 或`<br>` BiFunction<Apple, Apple, Integer>  `或 <br>`ToIntBiFunction<Apple, Apple>` |

## 使用局部变量

**限制:**  Lambda可以没有限制地捕获（也就是在其主体中引用）实例变量和静态变量。但局部变量必须显式声明为final， 或事实上是final ,不会被改变的

## 方法引用

​	方法引用主要有三类

>1. 指向静态方法的方法引用（例如`Integer`的`parseInt`方法，写作`Integer::parseInt`）
>
>2. 指向任意类型实例方法的方法引用（例如 String 的 length 方法，写作 String::length) 
>
>   在引用一个对象的方法，而这个对象本身是Lambda的一个参数。例如，Lambda 表达式`(String s) -> s.toUppeCase()`可以写作`String::toUpperCase`
>
>3. 指向现有对象的实例方法的方法引用 （假设你有一个局部变量`expensiveTransaction` 用于存放Transaction类型的对象，它支持实例方法`getValue`，那么你就可以写`expensiveTransaction::getValue`）即在Lambda中调用一个已经存在的外部对象中的方法



# Steam 流





| 操作        | 类型              | 返回类型      | 使用的类型/函数式接口    | 函数描述符       |
| ----------- | ----------------- | ------------- | ------------------------ | ---------------- |
| `filter`    | 中间              | `Stream<T>`   | `Predicate<T>`           | `T -> boolean`   |
| `distinct`  | 中间(有状态-无界) | `Stream<T>`   |                          |                  |
| `skip`      | 中间(有状态-有界) | `Stream<T>`   | `long`                   |                  |
| `limit`     | 中间(有状态-有界) | `Stream<T>`   | `long`                   |                  |
| `map`       | 中间              | `Stream<T>`   | `Function<T, R>`         | `T -> R`         |
| `flatMap`   | 中间              | `Stream<T>`   | `Function<T, Stream<R>>` | `T -> Stream<R>` |
| `sorted`    | 中间(有状态-无界) | `Stream<T>`   | `Comparator<T>`          | `(T, T) -> int`  |
| `anyMatch`  | 终端              | `boolean`     | `Predicate<T>`           | `T -> boolean`   |
| `nonMatch`  | 终端              | `boolean`     | `Predicate<T>`           | `T -> boolean`   |
| `allMatch`  | 终端              | `boolean`     | `Predicate<T>`           | `T -> boolean`   |
| `findAny`   | 终端              | `Optional<T>` |                          |                  |
| `findFirst` | 终端              | `Optional<T>` |                          |                  |
| `foreach`   | 终端              | `void`        | `Consumer<T>`            | `T -> void`      |
| `collect`   | 终端              | `R`           | `Collector<T, A, R>`     |                  |
| `reduce`    | 终端(有状态-有界) | `Optional<T>` | `BinaryOperator<T>`      | `(T, T) -> T`    |
| `count`     | 终端              | `long`        |                          |                  |
| `max/min`   | 终端              | `Optional<T>` | `Comparator<T>`          | `(T, T) -> int`  |

实战

1. 找出2011年发生的所有交易，并按交易额排序（从低到高）

   ```java
   transactions.stream()
       .filter(transaction -> transaction.getYear() == 2011)
       .sorted(comparing(Transcation::getValue))
       .collect(Collectors.toList());
   ```

   

2.  交易员都在哪些不同的城市工作过？

   ```java
   transactions.stream()
       .map(transaction -> transaction.getTrader().getCity())
       .distinct()
       .collect(Collectors.toList());
   或
   transactions.stream()
       .map(transaction -> transaction.getTrader().getCity())
       .collect(Collectors.toSet());
   ```

   

3. 查找所有来自于剑桥的交易员，并按姓名排序。

   ```java
   transactions.stream()
       .map(Transaction::getTrader)
       .filter(trader -> "Cambridge".equals(trader.getCity()))
       .distinct()
       .sorted(comparing(Trader::getName))
       .collect(Collectors.toList());
   ```

   

## Collectors类的静态工厂方法

已经引入Collectors类的所有静态工厂方法

`import static java.util.stream.Collectors.*;`

| 工厂方法              | 返回类型               | 用 于                                                        | 案例                                                         |
| --------------------- | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `toList()`            | `List<T>`              | 把流中所有项目收集到一个 List                                | `List dishes = menuStream.collect(toList());`                |
| `toSet()`             | `List<T>`              | 把流中所有项目收集到一个 Set，删除重复项                     | `Set dishes = menuStream.collect(toSet());`                  |
| `toCollection()`      | `Collection<T>`        | 把流中所有项目收集到给定的供应源创建的集合                   | `Collection dishes = menuStream.collect(toCollection(), ArrayList::new);` |
| `counting()`          | `Long`                 | 计算流中元素的个数                                           | `long howManyDishes = menuStream.collect(counting());`       |
| `summingInt()`        | `Integer`              | 对流中项目的一个整数属性求和                                 | `int totalCalories = menuStream.collect(summingInt(Dish::getCalories));` |
| `averagingInt()`      | `Double`               | 计算流中项目 Integer 属性的平均值                            | `double avgCalories = menuStream.collect(averagingInt(Dish::getCalories));` |
| `summarizingInt()`    | `IntSummaryStatistics` | 收集关于流中项目 Integer 属性的统计值，例如最大、最小、 总和与平均值 | `IntSummaryStatistics menuStatistics = menuStream.collect(summarizingInt(Dish::getCalories));` |
| `joining()`           | String                 | 连接对流中每个项目调用 `toString` 方法所生成的字符串         | `String shortMenu = menuStream.map(Dish::getName).collect(joining(", "));` |
| `maxBy()/minBy()`     | `Optional<T>`          | 一个包裹了流中按照给定比较器选出的最大/小元素的 Optional， 或如果流为空则为 `Optional.empty()` | `Optional fattest = menuStream.collect(maxBy(comparingInt(Dish::getCalories)));` |
| `reducing()`          | 归约操作产生的类型     | 从一个作为累加器的初始值开始，利用 `BinaryOperator `与流 中的元素逐个结合，从而将流归约为单个值 | `int totalCalories = menuStream.collect(reducing(0, Dish::getCalories, Integer::sum));` |
| `collectingAndThen()` | 转换函数返回的类型     | 包裹另一个收集器，对其结果应用转换函数                       | `int howManyDishes = menuStream.collect(collectingAndThen(toList(), List::size));` |
| `groupingBy()`        | `Map<K, List<T>>`      | 根据项目的一个属性的值对流中的项目作问组，并将属性值作 为结果 Map 的键 | `Map<<Dish.Type,List<Dish>> dishesByType = menuStream.collect(groupingBy(Dish::getType));` |
| `partitioningBy()`    | `Map<Boolean,List<T>>` | 根据对流中每个项目应用谓词的结果来对项目进行分区             | `Map<Boolean,List<Dish>> vegetarianDishes = menuStream.collect(partitioningBy(Dish::isVegetarian));` |




# Lambda

Lambda表达式的优势：

- 可以把函数当作参数进行传递。
- 可以使代码变的更加简洁紧凑。

## lambda表达式

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```

lambda 表达式的语法由参数列表、箭头符号 `->` 和函数体组成。函数体既可以是一个表达式，也可以是一个语句块：

- 表达式：表达式会被执行然后返回执行结果。
- 语句块：语句块中的语句会被依次执行，就像方法中的语句一样
  - `return` 语句会把控制权交给匿名方法的调用者
  - `break` 和 `continue` 只能在循环中使用
  - 如果函数体有返回值，那么函数体内部的每一条路径都必须返回值

表达式函数体适合小型 lambda 表达式，它消除了 `return` 关键字，使得语法更加简洁。

## 函数式接口

```java
@FunctionalInterface
public interface Function<T, R> {
  R apply(T t);
}
```

上述`Function`接口只有一个抽象方法，我们把这种只拥有一个抽象方法的接口称为**函数式接口**；可以通过`@FunctionalInterface`注解来显式的指定一个接口是函数式接口，加上这个注解后，编译器就会验证该接口是否满足函数式接口的要求。

Java8中新增了一个新的包：`java.util.function`，它里面包含了常用的函数式接口，例如：

- `Predicate<T>`——接收 `T` 并返回 `boolean`
- `Consumer<T>`——接收 `T`，不返回值
- `Function<T, R>`——接收 `T`，返回 `R`
- `Supplier<T>`——提供 `T` 对象（例如工厂），不接收值

## 词法作用域

在内部类中使用变量名非常容易出错，可能会把外部类的成员覆盖，此外`this`的引用会指向内部类自己而非外部类。

```java
String s = "run";
new Thread(new Runnable() {
    @Override
    public void run() {
        String s = "not run"; // 可以覆盖外部类的成员
        System.out.println(this);
        System.out.println(s);
    }
}).start();

// 运行结果
// edu.lambda.InnerClass$1@5d6623be
// not run
```

lambda表达式不会引入一个新的作用域，在lambda表达式函数体里面的变量包括`this`关键字和它外部环境的变量具有相同的语义。

## 方法引用

方法引用和 lambda 表达式拥有相同的特性，不过我们并不需要为方法引用提供方法体，我们可以直接通过方法名称引用已有方法。

```java
public class LambdaMethodReferences {

    private String name;
  
  	public LambdaMethodReferences() {

    }

    public LambdaMethodReferences(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public static String staticMethod() {
        return "staticMethod";
    }

    public static void main(String[] args) {
        // 实例方法引用
        Function<LambdaMethodReferences, String> getName = LambdaMethodReferences::getName;
        System.out.println(getName.apply(new LambdaMethodReferences("name")));
        
        // 静态方法引用
        Supplier<String> staticMethod = LambdaMethodReferences::staticMethod;
        System.out.println(staticMethod.get());
      
        // 构造方法引用，根据目标类型使用单个String参数的构造方法
      	Function<String, LambdaMethodReferences> constructMethod = LambdaMethodReferences::new;
        LambdaMethodReferences lambdaMethodReferences = constructMethod.apply("name");
      
        // 构造方法引用，根据目标类型使用无参数的构造方法
        Supplier<LambdaMethodReferences> noArgsConstructMethod = LambdaMethodReferences::new;
        lambdaMethodReferences = noArgsConstructMethod.get();
    }

}
```

# Stream

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

- **无存储**：*stream*不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器等。
- **惰式执行**：*stream*上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
- **可消费性**：*stream*只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。

Stream上的所有操作分为两类：

- 中间操作：记录中间操作过程并返回一个新的*stream*。
  - 无状态：元素的处理不受前面元素的影响。
  - 有状态：必须等到所有元素处理之后才知道最终结果，比如排序是有状态操作，在读取所有元素之前并不能确定排序结果。
- 结束操作：触发实际计算，把所有积攒的中间操作以*pipeline*的方式执行，这样可以减少迭代次数；计算完成之后*stream*就会失效。
  - 非短路操作：必须计算所有结果。
  - 短路操作：不用计算所有结果，可提前返回；相当于在for循环中做了breke。

| Stream操作分类                    |                                                         |                                                              |
| --------------------------------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| 中间操作 | 无状态                                       | unordered() filter() map() mapToInt() mapToLong() mapToDouble() flatMap() flatMapToInt() flatMapToLong() flatMapToDouble() peek() |
|| 有状态             | distinct() sorted() limit() skip()             |
| 结束操作     | 非短路操作                                              | forEach() forEachOrdered() toArray() reduce() collect() max() min() count() |
|| 短路操作        | anyMatch() allMatch() noneMatch() findFirst() findAny() |

## 实现原理

### 以前的方式

不用Stream的实现方式：

```java
// Sink.begin(size)
List<String> goodsNameList = new ArrayList<>(goodsList.size());
for (Goods goods : goodsList) {
  // Sink.accept()
  if (!"品质生活".equals(goods.getCategory())) {
    continue;
  }
  goodsNameList.add(goods.getName());
}
// Sink.end
goodsNameList.sort(String::compareTo);
```

- begin：构建一个容器
- accept：执行自定义的操作逻辑
- end：操作逻辑全部执行完后做后续处理

上面代码逻辑用Stream的实现方式：

```java
List<String> goodsNameList = goodsList.stream()
        .filter(goods -> "品质生活".equals(goods.getCategory()))
        .map(Goods::getName)
  			.sorted(String::compareTo)
        .collect(Collectors.toList());
```

### Sink结构

| 方法名                          | 作用                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| void begin(long size)           | 开始遍历元素之前调用。                                       |
| void end()                      | 所有元素遍历完成之后调用。                                   |
| boolean cancellationRequested() | 是否可以结束操作，可以让短路操作尽早结束。                   |
| void accept(T t)                | 遍历元素时调用，接受一个待处理元素，并对元素进行处理，然后将处理结果传递给下游Sink的accept方法。 |

Sink的这几个方法都是按照`处理->转发`的模式实现的，一种可能的`accept()`方法实现方式如下：

```java
void accept(U u){
    1. 使用当前Sink包装的回调函数处理u
    2. 将处理结果传递给下游的Sink
}
```

### 执行过程

不含有状态操作的执行流程：

![image-20200630124038462](../../图片/image-20200630124038462.png)

- 通过`Collection.stream()`返回一个`Head Stream`，不包含任何操作，只是为了生成下游的Stream。
- 通过调用`filter()`、`map()`等中间操作的方法都会返回一个包含当前操作和上游Stream引用的新Stream。
- 在触发`collect()`结束操作的时候，会通过`wrapSink()`方法生成一个Sink链，每个Sink都包含下游Sink的引用。
- Sink的`begin()`、`accept()`、`end()`方法都具有传递性，当前Sink在执行完自己的操作后，又会去调用下游Sink的对应方法。

包含有状态操作的执行过程：

![image-20200630134124284](/Users/hejun/Library/Application Support/typora-user-images/image-20200630134124284.png)

- 因为`sorted()`方法是有状态操作，它需要所有上游操作全部执行完毕。
- `SortingSink`会在`end()`方法中对中间结果进行排序，然后调用下游Sink的`begin()`、`accept()`、`end()`方法。

包含短路操作的执行过程：

![image-20200630150317880](/Users/hejun/Library/Application Support/typora-user-images/image-20200630150317880.png)

### filter

```java
@Override
public final Stream<P_OUT> filter(Predicate<? super P_OUT> predicate) {
    Objects.requireNonNull(predicate);
    return new StatelessOp<P_OUT, P_OUT>(this, StreamShape.REFERENCE,
                                 StreamOpFlag.NOT_SIZED) {
        @Override
        Sink<P_OUT> opWrapSink(int flags, Sink<P_OUT> sink) {
            return new Sink.ChainedReference<P_OUT, P_OUT>(sink) {
                @Override
                public void begin(long size) {
                    downstream.begin(-1);
                }

                @Override
                public void accept(P_OUT u) {
                    if (predicate.test(u))
                        downstream.accept(u);
                }
            };
        }
    };
}
```

### map

```java
@Override
@SuppressWarnings("unchecked")
public final <R> Stream<R> map(Function<? super P_OUT, ? extends R> mapper) {
    Objects.requireNonNull(mapper);
    return new StatelessOp<P_OUT, R>(this, StreamShape.REFERENCE,
                                 StreamOpFlag.NOT_SORTED | StreamOpFlag.NOT_DISTINCT) {
        @Override
        Sink<P_OUT> opWrapSink(int flags, Sink<R> sink) {
            return new Sink.ChainedReference<P_OUT, R>(sink) {
                @Override
                public void accept(P_OUT u) {
                    downstream.accept(mapper.apply(u));
                }
            };
        }
    };
}
```

### sorted

```java
private static final class RefSortingSink<T> extends AbstractRefSortingSink<T> {
    private ArrayList<T> list;

    RefSortingSink(Sink<? super T> sink, Comparator<? super T> comparator) {
        super(sink, comparator);
    }

    @Override
    public void begin(long size) {
        if (size >= Nodes.MAX_ARRAY_SIZE)
            throw new IllegalArgumentException(Nodes.BAD_SIZE);
        list = (size >= 0) ? new ArrayList<T>((int) size) : new ArrayList<T>();
    }

    @Override
    public void end() {
        list.sort(comparator);
        downstream.begin(list.size());
        if (!cancellationWasRequested) {
            list.forEach(downstream::accept);
        }
        else {
            for (T t : list) {
                if (downstream.cancellationRequested()) break;
                downstream.accept(t);
            }
        }
        downstream.end();
        list = null;
    }

    @Override
    public void accept(T t) {
        list.add(t);
    }
}
```

### collect

```java
public static <T, I> TerminalOp<T, I>
    makeRef(Collector<? super T, I, ?> collector) {
        Supplier<I> supplier = Objects.requireNonNull(collector).supplier();
        BiConsumer<I, ? super T> accumulator = collector.accumulator();
        BinaryOperator<I> combiner = collector.combiner();
        class ReducingSink extends Box<I>
                implements AccumulatingSink<T, I, ReducingSink> {
            @Override
            public void begin(long size) {
                state = supplier.get();
            }

            @Override
            public void accept(T t) {
                accumulator.accept(state, t);
            }

            @Override
            public void combine(ReducingSink other) {
                state = combiner.apply(state, other.state);
            }
        }
        return new ReduceOp<T, I, ReducingSink>(StreamShape.REFERENCE) {
            @Override
            public ReducingSink makeSink() {
                return new ReducingSink();
            }

            @Override
            public int getOpFlags() {
                return collector.characteristics().contains(Collector.Characteristics.UNORDERED)
                       ? StreamOpFlag.NOT_ORDERED
                       : 0;
            }
        };
    }
```

### warpSink

```java
final <P_IN> Sink<P_IN> wrapSink(Sink<E_OUT> sink) {
    Objects.requireNonNull(sink);

    for ( @SuppressWarnings("rawtypes") AbstractPipeline p=AbstractPipeline.this; p.depth > 0; p=p.previousStage) {
        sink = p.opWrapSink(p.previousStage.combinedFlags, sink);
    }
    return (Sink<P_IN>) sink;
}
```

### copyInto

```java
final <P_IN> void copyInto(Sink<P_IN> wrappedSink, Spliterator<P_IN> spliterator) {
    Objects.requireNonNull(wrappedSink);

    if (!StreamOpFlag.SHORT_CIRCUIT.isKnown(getStreamAndOpFlags())) {
        wrappedSink.begin(spliterator.getExactSizeIfKnown());
        spliterator.forEachRemaining(wrappedSink);
        wrappedSink.end();
    }
    else {
        copyIntoWithCancel(wrappedSink, spliterator);
    }
}
```


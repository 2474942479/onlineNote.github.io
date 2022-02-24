# 处理异常

**使用 @ControllerAdvice 和 @ExceptionHandler处理全局异常这是目前很常用的一种方式，非常推荐**。

## 1. 自定义异常

> 1. **自定义异常根类**
>
>     > ```java
>     > /**
>     >  * 自定义异常根类
>     >  * @author 张
>     >  */
>     > public class BaseException extends RuntimeException{
>     > 
>     >     public BaseException() {
>     >     }
>     > 
>     >     public BaseException(String message) {
>     >         super(message);
>     >     }
>     > 
>     >     public BaseException(Throwable cause) {
>     >         super(cause);
>     >     }
>     > 
>     >     public BaseException(String message, Throwable cause) {
>     >         super(message, cause);
>     >     }
>     > }
>     > ```
>
> 2. **自定义异常类**
>
>     >```java
>     >/**
>     > * 自定义业务异常类
>     > *
>     > * @author 张
>     > * @AllArgsConstructor 全部参数构造器
>     > * @NoArgsConstructor   无参构造器
>     > */
>     >@Data
>     >@AllArgsConstructor
>     >@NoArgsConstructor
>     >public class MyException extends BaseException {
>     >
>     >    private Integer status;
>     >
>     >    private String msg;
>     >
>     >    @Override
>     >    public String toString() {
>     >        return "MyNullPointerException{" +
>     >                "message=" + msg +
>     >                ", status=" + status +
>     >                '}';
>     >    }
>     >}
>     >
>     >```



## 2. 统一异常处理

使用 @ControllerAdvice 和 @ExceptionHandler处理全局异常

​	`我们只需要在类上加上@ControllerAdvice注解这个类就成为了全局异常处理类，当然你也可以通过 assignableTypes指定特定的 Controller 类，让异常处理类只处理特定类抛出的异常。`

```java
/**
 * 统一异常处理类
 *
 * @author 张
 */
@ControllerAdvice
@Slf4j
public class GlobalException {

    /**
     * 指定出现什么异常执行这个方法
     * 全局异常
     */

    @ExceptionHandler(Exception.class)
    @ResponseBody
    public MyResultUtils error(Exception e) {

        e.printStackTrace();
        return MyResultUtils.error().message("执行了全局异常处理");

    }

    /**
     * 特定异常
     */
    @ExceptionHandler(NullPointerException.class)
    @ResponseBody
    public MyResultUtils error(NullPointerException e) {

        e.printStackTrace();
        return MyResultUtils.error().message("发生了空指针错误").code(20001);

    }


    /**
     * 自定义异常类
     */
    @ExceptionHandler(MyException.class)
    @ResponseBody
    public MyResultUtils error(MyException e) {

        log.error(ExceptionUtils.outMore(e));
        e.printStackTrace();
        return MyResultUtils.error().message(e.getMsg()).code(e.getStatus());

    }

}

```

## 异常信息输出工具类

```java
/**
 * 输出更多异常信息
 * @author 张
 */
public class ExceptionUtils {

    public static String outMore(Exception e) {
        StringWriter sw = null;
        PrintWriter pw = null;
        try {
            sw = new StringWriter();
            pw = new PrintWriter(sw);
            // 将出错的栈信息输出到printWriter中
            e.printStackTrace(pw);
            pw.flush();
            sw.flush();
        } finally {
            if (sw != null) {
                try {
                    sw.close();
                } catch (IOException e1) {
                    e1.printStackTrace();
                }
            }
            if (pw != null) {
                pw.close();
            }
        }
        return sw.toString();
    }
}

```


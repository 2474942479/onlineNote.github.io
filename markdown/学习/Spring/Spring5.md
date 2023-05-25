# Spring

## 7大组件

![img](https://images2017.cnblogs.com/blog/1219227/201709/1219227-20170930225010356-45057485.gif)



**核心容器(Spring core)**

核心容器提供Spring框架的基本功能。Spring以bean的方式组织和管理Java应用中的各个组件及其关系。Spring使用BeanFactory来产生和管理Bean，它是工厂模式的实现。BeanFactory使用控制反转(IOC)模式将应用的配置和依赖性规范与实际的应用程序代码分开。BeanFactory使用依赖注入的方式提供给组件依赖。

**Spring上下文(Spring context)**

Spring上下文是一个配置文件，向Spring框架提供上下文信息。Spring上下文包括企业服务，如JNDI、EJB、电子邮件、国际化、校验和调度功能。

**Spring面向切面编程(Spring AOP)**

通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring框架中。所以，可以很容易地使 Spring框架管理的任何对象支持 AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。

**Spring DAO模块**

DAO模式主要目的是将持久层相关问题与一般的的业务规则和工作流隔离开来。Spring 中的DAO提供一致的方式访问数据库，不管采用何种持久化技术，Spring都提供一直的编程模型。Spring还对不同的持久层技术提供一致的DAO方式的异常层次结构。

**Spring ORM模块**

Spring 与所有的主要的ORM映射框架都集成的很好，包括Hibernate、JDO实现、TopLink和IBatis SQL Map等。Spring为所有的这些框架提供了模板之类的辅助类，达成了一致的编程风格。

**Spring Web模块**

Web上下文模块建立在应用程序上下文模块之上，为基于Web的应用程序提供了上下文。Web层使用Web层框架，可选的，可以是Spring自己的MVC框架，或者提供的Web框架，如Struts、Webwork、tapestry和jsf。

**Spring MVC框架(Spring WebMVC)**

MVC框架是一个全功能的构建Web应用程序的MVC实现。通过策略接口，MVC框架变成为高度可配置的。Spring的MVC框架提供清晰的角色划分：控制器、验证器、命令对象、表单对象和模型对象、分发器、处理器映射和视图解析器。Spring支持多种视图技术

## 核心容器

**Beans、Core、Context、SpEL**

1. core和beans模块提供了整个框架最基础的部分，包括了IoC(控制反转)和Dependency Injection(依赖注入)。

2. Context建立在Core和Beans模块提供的基础之上：他提供了框架式访问对象的方式

3. core、beans、context构成了Spring的骨架

4. SpEL:提供了一种强大的用于在运行时操作对象的表达式语言

## 一 IOC

    实现原理: 工厂模式  通过xml配置文件获取类的全路径  再根据反射的Class.forName(全路径)获取类的class文件,再通过class.newInstance()实例化对象  从而实现解耦。
1、IOC思想基于IOC容器完成，IOC容器底层就是对象工厂

    (1)BeanFactory:IOC容器基本实现，是Spring内部的使用接口，不提供开发人员进行使用
    *加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象
    
    (2)ApplicationContext:BeanFactory,接口的子接口，提供更多更强大的功能，一般由开发人员进行使用
    *加载配置文件时候就会把在配置文件对象进行创建·

1、生命周期:从对象创建到对象销毁的过程

    (1）通过构造器创建bean实例（无参数构造)
    (2）为bean的属性设置值和对其他bean引用（调用set方法）
    (3）调用bean的初始化的方法（需要进行配置初始化的方法）
    (4) bean可以使用了对象获取到了)
    (5）当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法）

## 二 AOP
1. 什么是AOP

    (1）面向切面编程（方面)，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
    (2）通俗描述:不通过修改源代码方式，在主干功能里面添加新功能

2. AOP〔底层原理）

    AOP底层使用动态代理 ,有两种情况动态代理
        第一种有接口情况，使用JDK动态代理。 创建接口实现类代理对象，增强类方法
        第二种没有接口情况，使用CGLIB动态代理。创建子类的代理对象，增强类的方法
3. 操作术语

    (1)、连接点
    类里面哪些方法可以被增强，这些方法称为连接点
    (2)、切入点
    实际被真正增强的方法，称为切入点
    (3)、通知（增强)
        1> 实际增强的逻辑部分称为通知（增强）
        2> 通知有5种类型 前置通知 后置通知 环绕通知  异常通知 最终通知(类似finally)
    (4)、切面 (是动作) : 把通知应用到切入点过程

4. AOP操作

    (1)、Spring框架一般都是基于Aspect实现AOP操作
        Aspect不是Spring 组成部分，独立AOP框架，一般把Aspect和Spirng.框架一起使用，进行AOP操作
    (2)、基于AspectJ实现AOP操作
        (1）基于xml配置文件实现
        (2）基于注解方式实现（使用）。
    (3)、切入点表达式
        (1）切入点表达式作用:知道对哪个类里面的哪个方法进行增强
        (2）语法结构:  execution([权限修饰符][返回类型][类限定名][方法名称]([参数列表]))
        
            修饰符可省略
            举例1:对edu.zsq.springtest.dao.UserDao类里面的 insert进行增强
            execution(* edu.zsq.springtest.dao.UserDao.insert(..))
            举例2:对edu.zsq.springtest.dao.UserDao类里面的所有的方法进行增强
            execution(* edu.zsq.springtest.dao.UserDao.*(..))
            举例3：对edu.zsq.springtest.dao包里面所有类，类里面所有方法进行增强：
            execution(* edu.zsq.springtest.dao.*.*(..))
    (4) 相同的切入点抽取
        申明目标类切入点范围
    　　　 1.方法必须private，没有返回值，没有参数
                @Pointcut("execution(* edu.zsq.springtest.dao.impl.UserDaoImpl.update(..))")
                public void pointCut(){}
    　　　 2.之后使用将其当成方法调用。例如：@After("pointCut()")
    (5)有多个增强类多同一个方法进行增强，设置增强类优先级
       在增强类上面添加注解@Order(数字类型值) 数字越小  优先级越高

##  三 事务      
### 什么是事务

```java
(1）事务是数据库操作最基本单元，逻辑上一组操作，要么都成功，如果有一个失败所有操作都失败
(2）典型场景:银行转账
    lucy转账100元给mary
    露西转账100元给玛丽
```
### 事务四个特性(ACID) 

```java
(1）原子性: 过程中不可分割 逻辑上一组操作，要么都成功，要么都失败
(2）一致性: 操作前后总量不变
(3)隔离性：各个事务操作之间互不影响
(4)持久性：事务提交后 表中数据发生变化
```

### 数据库设计三大范式

为了建立冗余较小、结构合理的数据库，设计数据库时必须遵循一定的规则。在关系型数据库中这种规则就称为范式。范式是符合某一种设计要求的总结。要想设计一个结构合理的关系型数据库，必须满足一定的范式。  

在实际开发中最为常见的设计范式有三个：

> **第一范式(确保每列保持原子性)**
>
> > 第一范式是最基本的范式。如果数据库表中的所有字段值都是不可分解的原子值，就说明该数据库表满足了第一范式。
> >
> > 第一范式的合理遵循需要根据系统的实际需求来定。比如某些数据库系统中需要用到“地址”这个属性，本来直接将“地址”属性设计成一个数据库表的字段就行。但是如果系统经常会访问“地址”属性中的“城市”部分，那么就非要将“地址”这个属性重新拆分为省份、城市、详细地址等多个部分进行存储，这样在对地址中某一部分操作的时候将非常方便。这样设计才算满足了数据库的第一范式，如下表所示。
> >
> >   
> >
> > 
> >
> > ![img](https://img-blog.csdn.net/20180115143238397?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ1Njk0OTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
> >
> > 上表所示的用户信息遵循了第一范式的要求，这样在对用户使用城市进行分类的时候就非常方便，也提高了数据库的性能。         
>
> **第二范式(确保表中的每列都和主键相关)**
>
> > 第二范式在第一范式的基础之上更进一层。第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）。也就是说在一个数据库表中，一个表中只能保存一种数据，不可以把多种数据保存在同一张数据库表中。
> >
> >   
> >
> > 比如要设计一个订单信息表，因为订单中可能会有多种商品，所以要将订单编号和商品编号作为数据库表的联合主键，如下表所示。
> >
> > ![img](https://img-blog.csdn.net/20180115143312100?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ1Njk0OTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
> >
> >  **订单信息表**
> >
> > 
> >
> > 这样就产生一个问题：这个表中是以订单编号和商品编号作为联合主键。这样在该表中商品名称、单位、商品价格等信息不与该表的主键相关，而仅仅是与商品编号相关。所以在这里违反了第二范式的设计原则。
> >
> > 而如果把这个订单信息表进行拆分，把商品信息分离到另一个表中，把订单项目表也分离到另一个表中，就非常完美了。如下所示。
> >
> > ![img](https://img-blog.csdn.net/20180115143338604?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ1Njk0OTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
> >
> > 
> >
> > 这样设计，在很大程度上减小了数据库的冗余。如果要获取订单的商品信息，使用商品编号到商品信息表中查询即可。 
>
> **第三范式(确保每列都和主键列直接相关,而不是间接相关)**
>
> > 第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。
> >
> >   
> >
> > 比如在设计一个订单数据表的时候，可以将客户编号作为一个外键和订单表建立相应的关系。而不可以在订单表中添加关于客户其它信息（比如姓名、所属公司等）的字段。如下面这两个表所示的设计就是一个满足第三范式的数据库表。
> >
> > ![img](https://img-blog.csdn.net/20180115143405849?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ1Njk0OTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
> >
> > 
> >
> > 这样在查询订单信息的时候，就可以使用客户编号来引用客户信息表中的记录，也不必在订单信息表中多次输入客户信息的内容，减小了数据冗余。

### 事务操作（Spring事务管理介绍）

在Spring进行事务管理操作,有两种方式

* 编程式事务管理

* 声明式事务管理(推荐)
    * 基于注解方式(推荐)
    * 基于xml配置文件方式

### 声明式事务管理

6.  在Spring进行声明式事务管理，底层使用AOP原理实现了事务的添加

2.  Spring开启声明式事务管理

    (1）提供一个接口(PlatformTransactionManager)，代表事务管理器，这个接口针对不同的框架提供不同的实现类。
    (2)  xml开启事务管理添加并在service实现类上添加@Transactional注解

    #### @Transactional注解参数

    > 1. propagation:**事务传播行为**(默认REQUIRED)  多事务方法(对表中数据进行变化的操作 增删改)之间直接调用，这个过程中事务是如何管理的
    >
    >     > ```java
    >     >  掌握：PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_NESTED
    >     > 1. PROPAGATION_REQUIRED,required (默认值)如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。 
    >     >     A 如果使用事务，B 使用同一个事务。（支持当前事务）
    >     >     A 如果没有事务，B 将创建一个新事务。
    >     > 2. PROPAGATION_SUPPORTS，supports,支持事务
    >     > 	A 如果使用事务，B 使用同一个事务。（支持当前事务）
    >     >    	A 如果没有事务，B 将以非事务执行。
    >     > 3.PROPAGATION_MANDATORY，mandatory 强制 支持当前事务，如果当前没有事务，就抛出异常
    >     >     A 如果使用事务，B 使用同一个事务。（支持当前事务）
    >     >     A 如果没有事务，B 抛异常
    >     > 4.PROPAGATION_REQUIRES_NEW ， requires_new ，新建事务，如果当前存在事务，把当前事务挂起。
    >     > 	A 如果使用事务，B 将A的事务挂起，再创建新的。
    >     > 	A 如果没有事务，B 将创建一个新事务  
    >     > 5.PROPAGATION_NOT_SUPPORTED 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
    >     > 	A 如果使用事务，B 将A的事务挂起，以非事务执行
    >     > 	A 如果没有事务，B 以非事务执行
    >     > 6.PROPAGATION_NEVER，never 以非事务方式执行，如果当前存在事务，则抛出异常。
    >     > 	A 如果使用事务，B 抛异常
    >     > 	A 如果没有事务，B 以非事务执行 
    >     > 7.PROPAGATION_NESTED nested 嵌套事务
    >     > 	A 如果使用事务，B 将采用嵌套事务。
    >     >  　　支持当前事务，新增Savepoint点，与当前事务同步提交或回滚。
    >     >     嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚
    >     > ```
    >
    > 2. ioslation:**事务隔离级别**
    >
    >     | **隔离级别**             | **脏读(读到未提交数据)** | **不可重复读(读到修改数据)** | **幻读(读到插入或删除数据)** |
    >     | :----------------------- | ------------------------ | ---------------------------- | ---------------------------- |
    >     | 未提交读Read uncommitted | 可能发生                 | 可能发生                     | 可能发生                     |
    >     | 提交读Read committed     | -                        | 可能发生                     | 可能发生                     |
    >     | 可重复读Repeatable read  | -                        | -                            | 可能发生                     |
    >     | 可序列化Serializable     | -                        | -                            | -                            |
    >
    > 3. **timeout:**超时时间
    >             (1）事务需要在一定时间内进行提交，如果不提交进行回滚↓
    >             (2）默认值是-1，设置时间以秒单位进行计算
    >
    > 3. readOnly:**是否只读**
    >             (1）读:查询操作，写:添加修改删除操作
    >             (2) readOnly_默认值false，表示可以查询，可以添加修改删除操作
    >             (3）设置readOnly值是true，设置成true之后，只能查询。
    >         
    > 5. rollbackFor:**回滚**  设置出现哪些异常进行事务回滚
    >
    > 5. noRollbackFor:**不回滚**  设置出现哪些异常不进行事务回滚
    
    #### 注意
    
    ​	事务方法往往会占用数据库连接,如果事务方法耗时过长,会导致应用数据库连接池不够用,同时增加DB的连接数量。既影响应用服务器同时给数据库服务器造成压力。所以处理事务方法时候要进行细粒度进行操作。尽量避免
    
    ​	事务方法中包含了大量不必要的完成调用。潜规则: (把自己的db生命安全,交给服务服务方式一种很危险的行为,别人服务抖了一下,可能自己没有影响,但是把你搞挂了就很难受了)
    
    
    
    建议对要求保证事务的方法进行细粒度操作:
    
    
    
    - 使用编程式事务进行操作,Session要注意释放,否则有严重问题
    
    ```
    SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH,false);
    //提交
    sqlSession.commit();
    //回滚
    sqlSession.rollback();
    //切记必须关闭
    sqlSession.close();
    ```
    
    - 或者使用Spring提供的编程式事务进行coding
    
    ```
    package com.idanchuang.purchase.center.core.common.util;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;
    import org.springframework.transaction.support.TransactionCallback;
    import org.springframework.transaction.support.TransactionTemplate;
    
    /**
     * 声明式事务固有好处,比如当代码块中只涉及到数据库操作时候。这种是最佳使用场景。 <br>
     * 但是往往一些系统中,声明式的使用,会降低性能。如方法中涉及到多个系统的查询或者无关事务的方法。<br>
     * 此时如果 还使用声明式事务是有问题的。就好比把自己系统的生命交给了第三方系统,第三方系统抖一抖,我们跟着抖一抖。
     *
     * 因为在使用声明式事务时候,要考虑下这个问题。Spring 框架也为我们提供了编程式事务,如下<br>
     * </> 可以使用Spring原生的事务,也可以使用当前工具类,工具类并没有技术含量,主要是将我们不需要使用的东西给屏蔽了<br>
     * 让我们只专注于业务。
     *
     * @see TransactionTemplate#execute(TransactionCallback)
     * @author liuxin 2020/12/1 2:26 下午
     */
    @Component
    public class PmsTransactionTemplate {
    
        /**
         * 事务方法
         */
        private TransactionTemplate transactionTemplate;
    
        @Autowired
        public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
            this.transactionTemplate = transactionTemplate;
        }
    
        /**
         * 执行事务动作(有返回值)
         */
        public <T> T transactionAction(TransactionCodeBlockSupplier<T> codeBlockSupplier) {
            return transactionTemplate.execute(transactionStatus -> codeBlockSupplier.transactionCallbackAction());
        }
    
        /**
         * 执行事务动作
         */
        public void transactionAction(TransactionCodeBlock codeBlock) {
            transactionTemplate.execute(transactionStatus -> {
                codeBlock.transactionAction();
                return null;
            });
        }
    
        /**
         * 事务代码块,代码块里面的操作会放在一个单独的事务中
         */
        @FunctionalInterface
        public interface TransactionCodeBlock {
    
            /**
             * 事务动作
             */
            void transactionAction();
        }
    
        /**
         * 事务代码块(包含返回值),代码块里面的操作会放在一个单独的事务中
         */
        @FunctionalInterface
        public interface TransactionCodeBlockSupplier<T> {
    
            /**
             * 事务动作(包含返回值)
             *
             * @return T 返回值泛型
             */
            T transactionCallbackAction();
        }
    }
    ```
    
    - 将必须要保证事务的方法抽出到单独的事务方法并加上事务注解,减少不必要的远程调用嵌套在事务方法中
    
    
## 四、Spring5 新特性
1、核心特性

        JDK8的增强:
        。访问Resuouce时提供getFile或和isFile防御式抽象
        。有效的方法参数访问基于java 8反射增强
        。在Spring核心接口中增加了声明default方法的支持一贯使用JDK┐Charset和StandardCharsets的增强
        。兼容JDK9
        . Spring 5.o框架自带了通用的日志封装
        。持续实例化via构造函数(修改了异常处理)
        . Spring 5.o框架自带了通用的日志封装
        . springjcl替代了通用的日志，仍然支持可重写
        。自动检测log4j 2.x,SLF4J,JUL(java.util.Logging）而不是其他的支持
        。访问Resuouce时提供getFile或和isFile防御式抽象
        。基于NIO的readableChannel也提供了这个新特性

2、核心容器

        。支持候选组件索引(也可以支持环境变量扫描)
        。支持@Nullable注解
            @Nullable注解可以使用在方法上面，属性上面，参数上面，表示方法返回可以为空，属性值可以为空，参数值可以为空
        。函数式风格GenericApplicationContext/AnnotationConfigApplicationContext
        。基本支持bean API注册
        。在接口层面使用CGLIB动态代理的时候，提供事物，缓存，异步注解检测
        .XML配置作用域流式
        .Spring WebMVC
        。全部的Servlet 3.1签名支持在Spring-provied Filter实现
        。在Spring MVC Controller方法里支持Servlet4.o PushBuilder参数
        。多个不可变对象的数据绑定(Kotlin/ Lombok/@ConstructorPorties)
        。支持jackson2.9
        。支持JSON绑定API
        。支持protobuf3
        。支持Reactorg.1Flux和Mono

3、Spring5框架新功能（Webflux) 

    1、SpringWebflux介绍(应用于网关)
        (1)是Spring5添加新的模块，用于web 开发的，功能和SpringMVC类似的，Webflux使用当前一种比较流程响应式编程出现的框架。
        (2)使用传统web框架，比如SpringMVC，这些基于Servlet,容器，Webflux是一种异步非阻塞的框架，异步非阻塞的框架在Servlet3.1以后才支持，核心是基于Reactor的相关API实现的
        (3）什么是异步非阻塞
            异发和同步 ： 异步和同步针对调用者，调用者发送请求，如果等着对方回应之后才去做其他事情就是同步，如果发送请求之后不等着对方回应就去做其他事情就是异步。
            非阻塞和阻塞：阻塞和非阻塞针对被调用者，被调用者受到请求之后，做完请求任务之后才给出反馈就是阻塞，受到请求之后马上给出反馈然后再去做事情就是非阻塞。
            上面都是针对对象不一样，
        (4)Webflux特点:
            第一 非阻塞式:在有限资源下，提高系统吞吐量和伸缩性，以Reactor为基础实现响应式编程.
            第二 函数式编程:Spring5框架基于java8，Webflux使用Java8函数式编程方式实现路由请求
        (5)与SpringMVC比较:
            第一 两个框架都可以使用注解方式，都运行在Tomet等容器中
            第二 SpringMVC采用命令式编程，Webflux采用异步响应式编程
    
    2、响应式编程
        (1)什么是响应式编程
            响应式编程是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。
            电子表格程序就是响应式编程的一个例子。单元格可以包含字面值或类似"=B1+C1"的公式，而包含公式的单元格的值会依据其他单元格的值的变化而变化。
        (2）Java8及其之前版本
            观察者模式(案例:军队哨兵,发现敌人,发出通知，做出响应)
            实现观察者模式的两个类 Observer和 Observable
        (3)、响应式编程（Reactor实现）
            (1  响应式编程操作中，Reactor是满足Reactive规范框架
            (2  Reactor有两个核心类，Mono和Flux，这两个类实现接口Publisher，提供丰富操作符。Flux对象实现发布者，返回N个元素;Mono实现发布者，返回0或者1个元素
            (3  Flux和Mono都是数据流的发布者，使用Flux和Mono都可以发出三种数据信号:元素值，错误信号，完成信号，错误信号和完成信号都代表终止信号，终止信号用于告诉订阅者数据流结束了，错误信号终止数据流同时把错误信息传递给订阅者
            (4  三种信号特点
                *错误信号和完成信号都是终止信号，不能共存的
                *如果没有发送任何元素值，而是直接发送错误或者完成信号，表示是空数据流
                *如果没有错误信号，没有完成信号，表示是无限数据流
            (5  调用just或者其他方法只是声明数据流，数据流并没有发出，只有进行订阅之后才会触发数据流，不订阅什么都不会发生的
                //just方法直接声明    subscribe()订阅
                Flux.just(1,2,3,4). subscribe(System.out::print);
                Mono.just(1).subscribe(System.out::print);
            (6  操作符(类似java8 新特性stream流的操作符): 对数据流进行一道道操作，称为操作符，比如工厂流水线·
                第一 map操作符 将元素映射为新元素
                第二 flatmap操作符 将每个元素映射为流,最终合并成一个大流  
                    
    3、SpringFlux执行流程:   SpringWebflux执行过程和 SpringMVC相似的
    
        SpringWebflux核心控制器DispatchHandler，实现接口WebHandler 
            *接口 WebHandler有一个方法: 
                Mono<Void> handle(ServerWebExchange var1);
                实现类: DispatcherHandler中的handle方法
                
                    public Mono<Void> handle(ServerWebExchange exchange) {  //  ServerWebExchange 放http请求响应信息
                            return this.handlerMappings == null ? this.createNotFoundError() : Flux.fromIterable(this.handlerMappings).concatMap((mapping) -> {
                                return mapping.getHandler(exchange);    //根据请求获取对应的Mapping(映射匹配)
                            }).next().switchIfEmpty(this.createNotFoundError()).flatMap((handler) -> {
                                return this.invokeHandler(exchange, handler);   //执行具体业务
                            }).flatMap((result) -> {
                                return this.handleResult(exchange, result);     //返回处理结果
                            });
                        }
        SpringWebflux核心组件
            *DispatcherHandler，负责请求的处理
            *HandlerMapping:请求查询到处理的方法
            *HandlerAdapter:真正负责请求处理
            *HandlerResultHandler:响应结果处理
            
        Spring Webflux实现函数式编程，实现两个接口:RourerFunction(路由处理)和HandlerFunction(处理函数）↓
        
    4、Spring Webflux(基于注解编程模型)
        说明:注解方式表面无太大区别，底层不一样
        SpringMVC方式实现，同步阻塞的方式，基于SpringMVC+Servlet+Tomcat.
        SpringWebflux:方式实现，异步非阻塞方式，基于SpringWebflux+Reactor+Netty.
    5、SpringWebflux（基于函数式编程模型）
        (1)在使用函数式编程模型操作时候，需要自己初始化服务器
        (2)基于函数式编程模型时候，有两个核心接口:RouterFunction(实现路由功能，请求转发
        给对应的handler）和HandlerFunction(处理请求生成响应的函数）。核心任务定义两个函数
        式接口的实现并且启动需要的服务器。
        (3) SpringWebflux 请求和响应不再是ServletRequest和 ServletResponse,，而是
        ServerReguest.和SerxerResponse
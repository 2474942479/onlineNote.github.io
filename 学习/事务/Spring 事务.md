### 事务操作（Spring事务管理介绍）

在Spring进行事务管理操作,有两种方式

* 编程式事务管理

* 声明式事务管理(推荐)
  * 基于注解方式(推荐)
  * 基于xml配置文件方式

### 声明式事务管理

6. 在Spring进行声明式事务管理，底层使用AOP原理实现了事务的添加

7. Spring开启声明式事务管理

   (1）提供一个接口(PlatformTransactionManager)，代表事务管理器，这个接口针对不同的框架提供不同的实现类。
   (2)  xml开启事务管理添加并在service实现类上添加@Transactional注解

   #### @Transactional注解参数

   > 1. propagation:**事务传播行为**(默认REQUIRED)  多事务方法(对表中数据进行变化的操作 增删改)之间直接调用，这个过程中事务是如何管理的
   >
   >    > ```java
   >    > 掌握：PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_NESTED
   >    > 1. PROPAGATION_REQUIRED,required (默认值)如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。 
   >    >     A 如果使用事务，B 使用同一个事务。（支持当前事务）
   >    >     A 如果没有事务，B 将创建一个新事务。
   >    > 2. PROPAGATION_SUPPORTS，supports,支持事务
   >    > 	A 如果使用事务，B 使用同一个事务。（支持当前事务）
   >    >    	A 如果没有事务，B 将以非事务执行。
   >    > 3.PROPAGATION_MANDATORY，mandatory 强制 支持当前事务，如果当前没有事务，就抛出异常
   >    >     A 如果使用事务，B 使用同一个事务。（支持当前事务）
   >    >     A 如果没有事务，B 抛异常
   >    > 4.PROPAGATION_REQUIRES_NEW ， requires_new ，新建事务，如果当前存在事务，把当前事务挂起。
   >    > 	A 如果使用事务，B 将A的事务挂起，再创建新的。
   >    > 	A 如果没有事务，B 将创建一个新事务  
   >    > 5.PROPAGATION_NOT_SUPPORTED 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
   >    > 	A 如果使用事务，B 将A的事务挂起，以非事务执行
   >    > 	A 如果没有事务，B 以非事务执行
   >    > 6.PROPAGATION_NEVER，never 以非事务方式执行，如果当前存在事务，则抛出异常。
   >    > 	A 如果使用事务，B 抛异常
   >    > 	A 如果没有事务，B 以非事务执行 
   >    > 7.PROPAGATION_NESTED nested 嵌套事务
   >    > 	A 如果使用事务，B 将采用嵌套事务。
   >    >  　　支持当前事务，新增Savepoint点，与当前事务同步提交或回滚。
   >    >     嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚
   >    > ```
   >
   > 2. ioslation:**事务隔离级别**
   >
   >    | **隔离级别**             | **脏读(读到未提交数据)** | **不可重复读(读到修改数据)** | **幻读(读到插入或删除数据)** |
   >    | :----------------------- | ------------------------ | ---------------------------- | ---------------------------- |
   >    | 未提交读Read uncommitted | 可能发生                 | 可能发生                     | 可能发生                     |
   >    | 提交读Read committed     | -                        | 可能发生                     | 可能发生                     |
   >    | 可重复读Repeatable read  | -                        | -                            | 可能发生                     |
   >    | 可序列化Serializable     | -                        | -                            | -                            |
   >
   > 3. **timeout:**超时时间
   >        (1）事务需要在一定时间内进行提交，如果不提交进行回滚↓
   >        (2）默认值是-1，设置时间以秒单位进行计算
   >
   > 4. readOnly:**是否只读**
   >        (1）读:查询操作，写:添加修改删除操作
   >        (2) readOnly_默认值false，表示可以查询，可以添加修改删除操作
   >        (3）设置readOnly值是true，设置成true之后，只能查询。
   >
   > 5. rollbackFor:**回滚**  设置出现哪些异常进行事务回滚
   >
   > 6. noRollbackFor:**不回滚**  设置出现哪些异常不进行事务回滚

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
# Spring自动注入

## Spring 将接口的多实现通过@AutoWired自动注入到List后Map中

​	在SpringBoot开发中，当一个接口A有多个实现类时，spring会很智能的将bean注入到List<A>或Map<String,A>变量中。@Order注解可以调整List中的顺序。Map集合中 key为实现名，value为接口实现

## 实现策略模式：根据配置使用对应的实现类

对应Map的注入，key必须为String类型，即bean的名称，而value为IPerson类型的对象实例。

通过对上述Map类型的注入，可以改写为根据bean名称，来获取并使用对应的实现类。


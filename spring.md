# Spring IOC容器和Bean

## 容器总揽
```org.springframework.beans``` 和```org.springframework.context```是Sprig 框架的 IoC 容器的基础包。```BeanFactory```提供能够管理任何类型的对象的高级配置机制。ApplicationContext是BeanFactory的一个子接口。它增加了与 Spring AOP 功能的整合更容易;消息资源处理（用于国际化），事件发布;和应用层的上下文，如WebApplicationContext 中的 Web 应用程序使
用。

The interface ```org.springframework.context.ApplicationContext``` represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the aforementioned beans. The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata. The configuration metadata is represented in XML, Java annotations, or Java code.

Spring ```ApplicationContext``` 接口提供了几种即装即用的实现方式。在独立应用中，通常以创建ClassPathXmlApplicationContext 或FileSystemXmlApplicationContext 的实例。虽然 XML 一直是传统的格式来定义配置元数据，但也可以指示容器使用 Java 注解或代码作为元数据格式，并通过提供少量的XML配置以声明方式启用这些额外的元数据格式的支持。

应用程序的类是通过配置元数据来结合的，以便 ApplicationContext 需要创建和初始化后，你有一个完全配置和可执行的系统或应用程序。
## 实例化容器
实例化 Spring IoC 容器是直截了当的。提供给 ApplicationContext构造器的路径就是实际的资源字符串，使容器装入从各种外部资源的配置元数据，如本地文件系统， Java CLASSPATH，等等。
``` java
ApplicationContext context =
new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});
```
## 使用容器
ApplicationContext 是能够保持 bean 定义以及相互依赖关系的高级工厂接口。使用方法```getBean(String name, Class requiredType)```就可以取得 bean 的实例。
ApplicationContext 中可以读取 bean 定义并访问它们，如下所示：
``` java
// create and configure beans
ApplicationContext context =
new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});
// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);
// use configured instance
List<String> userList = service.getUsernameList();
```
## Bean总览
Spring IoC 容易管理一个或者多个 bean。 bean 由应用到到容器的配置元数据创建,例如,在XML 中定义 <bean/> 的形式。
容器内部,这些 bean 定义表示为 BeanDefinition 对象,其中包含(其他信息)以下元数据:
- 限定包类名称:典型的实际实现是定义 bean 的类。
- bean 行为配置元素,定义了容器中的Bean应该如何行为(范围、生命周期回调,等等)。
- bean需要引用其他 bean 来完成工作,这些引用也称为合作者或依赖关系。
- 其他配置设置来设置新创建的对象,例如,连接使用 bean 的数量管理连接池,或者池的大小限制。


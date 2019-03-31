一、Spring IOC全面认知
1.1IOC和DI概述

spring容器spring-context官方文档参考:https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-java

在《精通Spring4.x 企业应用开发实战》中对IOC的定义是这样的：
IoC(Inversion of Control)控制反转，包含了两个方面：一、控制。二、反转
我们可以简单认为：
控制指的是：当前对象对内部成员的控制权。
反转指的是：这种控制权不由当前对象管理了.

总结:即对象对自身的属性和方法的控制权交给了第三方来进行管理了.

IoC(思想，设计模式)主要的实现方式有两种：依赖查找，依赖注入。
依赖注入是一种更可取的方式(实现的方式)
对我们而言，其实也没必要分得那么清，混合一谈也不影响我们的理解…
再通过昨天写过的工厂模式理解了没有？，其实所谓的IOC容器就是一个大工厂.
1.2IOC容器的原理
从上面就已经说了：IOC容器其实就是一个大工厂，它用来管理我们所有的对象以及依赖关系。
原理就是通过Java的反射技术来实现的！通过反射我们可以获取类的所有信息(成员变量、类名等等等)！
再通过配置文件(xml)或者注解来描述类与类之间的关系(之前是基于xml,现在是基于注解的@Configuration来进行实现的)
我们就可以通过这些配置信息和反射技术来构建出对应的对象和依赖关系了！

总结：其创建对象的方式是配置（配置、Javaconfig）的方式，然后通过反射的方式去获取到相应的类的信息，来生成bean。
我们简单来看看实际Spring IOC容器是怎么实现对象的创建和依赖的：

​​
步骤:（三个步骤(重)：生成注册表->生成bean->放入bean的缓存池）
1.根据Bean配置信息在容器内部创建Bean定义注册表
2.根据注册表加载、实例化bean、建立Bean与Bean之间的依赖关系
3.将这些准备就绪的Bean放到Map缓存池中，等待应用程序调用
Spring容器(Bean工厂)可简单分成两种：
BeanFactory
这是最基础、面向Spring的
ApplicationContext(此类实现了BeanFactory接口)
这是在BeanFactory基础之上，面向使用Spring框架的开发者。提供了一系列的功能！几乎所有的应用场合都是使用ApplicationContext！



1.3IOC容器装配Bean
1.3.1装配Bean方式
Spring4.x开始IOC容器装配Bean有4种方式：
XML配置
注解
JavaConfig
基于Groovy DSL配置(这种很少见)
总的来说：我们以XML配置+注解来装配Bean得多，其中注解这种方式占大部分！
1.3.2依赖注入方式
依赖注入的方式有3种方式：
依赖注入就是：只需要添加描述生成bean的注入的方式，IOC容器会根据注入的方式，去自动的组装Bean。
属性注入-->通过setter()方法注入
      public class A {
    private B b;
    public void importantMethod() {
        b.usefulMethod();
    }
    public void setB(B b) {
        this.b=b;
    }
}

构造函数注入
public class A {
    private B b;
    public A(B b) {
        this.b = b;
    }
    public void importantMethod() {
        b.usefulMethod();
    }
}
工厂方法注入
spring注解的方式注入：（上面的三种方式是针对spring的之前的xml的配置方式）
@Autowired或 @Resource注解方式进行Spring的依赖注入，在springAOP编程中就采用这种方式注入。
1.3.3对象之间关系
<bean>对象之间有三种关系：
依赖-->挺少用的(使用depends-on就是依赖关系了-->前置依赖【依赖的Bean需要初始化之后，当前Bean才会初始化】)
继承-->可能会用到(指定abstract和parent来实现继承关系)
引用-->最常见(使用ref就是引用关系了)
1.3.4Bean的作用域
Bean的作用域：
单例Singleton
多例prototype
与Web应用环境相关的Bean作用域
reqeust  一个请求一个实例
session  一个会话一个实例

注意（重）：在springboot中，使用@Bean来生成IOC的bean的时候，可以使用    @Scope("prototype")  或者是@SessionScope 来标注Bean的作用域，同时可以设置其值为单例还是多例。同时可以使用 @Description("Provides a basic example of a bean")来生成描述信息。使用xml的方式仍然可以配置。
使用@Bean注解时，可以配置initMethod()和destoryMethod方法，分别在实例化和销毁的时候执行。
或者使用通过@PostConstruct 和 @PreDestroy 方法 实现初始化和销毁bean之前进行的操作 
将Bean配置单例的时候还有一个问题：
如果我们的Bean配置的是单例，而Bean对象里边的成员对象我们希望是多例的话。那怎么办呢？？
参考文档：点击...

默认的情况下我们的Bean单例，返回的成员对象也默认是单例的(因为对象就只有那么一个)！

1.3.6处理自动装配的歧义性

使用@Primary注解设置为首选的注入Bean
使用@Qualifier注解设置特定名称的Bean来限定注入！
也可以使用自定义的注解来标识    

​

二 . Spring IOC源码分析（总结性分析）
源码分析文章汇总：http://www.iocoder.cn/Spring/good-collection/

带来的好处（背）
1、	代码耦和度低：Bean直接的依赖关系，通过控制反转，依赖注入的方式进行实现.
2、	降低了代码冗余：Bean的实例化，属性初始化，都直接交给IOC容器进行统一完成，在依赖JavaBean中只需要注入使用即可，无需重复代码的开发。
3、	JavaBean的单例性可配置可管理：通过简单的配置在IOC容器初始化时候，就进行JavaBean的对象的提前创建，可以实现在注入依赖时，根据需要完成其单例性效果，减少资源浪费。
4、	可测试性高：通过使用注解或者xml等方式，测试时无需对业务代码进行任何手动干预，完全交给IOC容器去完成，提高了测试效率与简单性。

IOC容器初始化过程
IOC容器的初始化过程整体来说分为以下三步：
Resource定位
1、	在开发过程中我们一般使用外部资源来表述Bean对象，所以初始化IOC容器的第一步就是要定位Resource外部资源，找到IOC容器初始化的需求文档（例如类和XML）。
2、	Resource定位实质就是BeanDefinition的资源定位。ApplicationContext的各种实现类从体系结构图来看，通过继承DefaultResourceLoader，具备了定位的Resource功能，也可以理解为IOC容器的定位功能是通过Resource众多接口以+各种实现类实现的。
载入
1、	通过Resource定位了IOC容器初始化的需求文档，接下来要做的可以理解为转换翻译，也就是将不同的Java Bean的定义信息，转换为统一的，Spring容器所认可的格式BeanDefinition，方便IoC容器按照需求对于普通JavaBean的管理。
2、	载入主要是通过一个BeanDefinition对Bean的配置信息进行统一包装，实现IOC容器中的统一管理。实质上BeanDefinition就是POJO对象在IoC容器中的抽象，通过BeanDefinition定义的一系列数据来使得IoC能够方便，直接的对Spring的Bean进行管理，也就是说BeanDefinition是Spring的领域对象。配置文件中的每一个或者@Service，@Component都对应着一个BeanDefinition。
      BeanDefinition 实例，其工作时需要工具BeanDefinitionReader去获取相关的类的信息. 该实例负责保存bean对象的所有必要信息，包括 bean 对象的 class 类型、是否是抽象类、构造方法和参数、其它属性等等。当客户端向容器请求相应对象时，容器就会通过这些信息为客户端返回一个完整可用的 bean 实例。
      
     BeanFactory 接口中主要包含 getBean、containBean、getType、getAliases 等管理 bean 的方法.
注册
1、	配置文件解析包装好之后，就需要将其注册到容器当中，便于以后的使用某个JavaBean的时候，读取该BeanDefinition进行初始化等等相关操作。
2、	IOC容器主要通过一个HashMap容器来维护这些BeanDefinition。此处对于某些特殊需求的JavaBean可能会做一些预处理，例如某个Bean设置了lazyinit属性，那么提前将这个Bean初始化好，并实现一定的依赖注入。
IOC初始化核心源码解析
下面是基于xml配置的Bean加载方式的时序图，在springboot下是通过注解的方式去扫描然后再加载Bean的。
idea中查看UML类图的时候，可以选择类图上面的某个接口或者是某个类，然后右键向父类衍生和向子类衍生。。
物料组件 
Resource、BeanDefinition、PropertyEditor以及最终的Bean它们是加工流程中被加工，被消费的组件，就像流水线上被加工的物料。
加工设备组件
ResourceLoader、BeanDefinitionReader、BeanFactoryPostProcessor、InstantiationStrategy以及BeanWrapper等组件像是流水线程不同环节的加工设备，对物料组件进行加工，各司其职，分工明确，互不依赖。

自己总结：
IOC源码中涉及到的两个关键类ApplicationContext  和 BeanFactory；
如下图所说：ApplicationContext接口去实现了BeanDefinition接口的、
​​
ApplicationContext主要负责资源的加载，例如xml或者是类中的@Configruation中的资源。
右键查看其实现有很多：
​​

例如以其中的一个实现为例：
AnnotationConfigApplicationContext

其中的构造方法中，就会去生成与容器相关的东西：
public AnnotationConfigApplicationContext(String... basePackages) {
   this();
   scan(basePackages);
   refresh();
}
其中的refresh（）方法，里面就汇总了Spring IOC的bean的加载流程：(里面去使用了BeanDefinition来生成容器的相关信息，其中的子实现由资源发现的功能 ，主要的功能流程都在下面的方法中)
其springboot启动过程，初始化后，在运行阶段，即调用SpringApplication类的run（）方法的时候，也会调用下面的refresh（）方法，来创建容器和装载通过SringFactoriesLoadder去扫描出来的@Configuration下面的bean。具体的可以在springboot启动的过程中debug调试。
@Override
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
所以：分析IOC容器的源码，可以从ApplicationContext为入口，可以以下面的手写代码的形式作为入口：
ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
Person person = context.getBean("person", Person.class);
 
 
 
 
 BeanFactory: bean的存储容器，在ApplicationContext刷新的时候，会去创建BeanFactory工厂。BeanDefinition就是包装后的Bean的形式，最终存储在BeanFactory中。	 上面是解析注解@Bean成为bean，解析xml形成Bean，其原理一样。
 
 本质上来讲：ApplicationContext是BeanFactory的增强实现，一般在硬编码声明式方式来使用ApplicationContext 。
 
​



三、Spring IOC相关面试题
2.1什么是spring?
什么是spring?
Spring 是个java企业级应用的开源开发框架。Spring主要用来开发Java应用，但是有些扩展是针对构建J2EE平台的web应用。Spring框架目标是简化Java企业级应用开发，并通过POJO为基础的编程模型促进良好的编程习惯。
2.2使用Spring框架的好处是什么？
使用Spring框架的好处是什么？
轻量：Spring 是轻量的，基本的版本大约2MB。
控制反转：Spring通过控制反转实现了松散耦合，对象们给出它们的依赖，而不是创建或查找依赖的对象们。
面向切面的编程(AOP)：Spring支持面向切面的编程，并且把应用业务逻辑和系统服务分开。
容器：Spring 包含并管理应用中对象的生命周期和配置。
MVC框架：Spring的WEB框架是个精心设计的框架，是Web框架的一个很好的替代品。
事务管理：Spring 提供一个持续的事务管理接口，可以扩展到上至本地事务下至全局事务（JTA）。
异常处理：Spring 提供方便的API把具体技术相关的异常（比如由JDBC，Hibernate or JDO抛出的）转化为一致的unchecked 异常。
2.3Spring由哪些模块组成?
Spring由哪些模块组成?
简单可以分成6大模块：
Core
AOP
ORM
DAO
Web
Spring EE
​​
2.4BeanFactory 实现举例
BeanFactory 实现举例
Bean工厂是工厂模式的一个实现，提供了控制反转功能，用来把应用的配置和依赖从正真的应用代码中分离。
在spring3.2之前最常用的是XmlBeanFactory的，但现在被废弃了，取而代之的是：XmlBeanDefinitionReader和DefaultListableBeanFactory
2.5什么是Spring的依赖注入？
什么是Spring的依赖注入？
依赖注入，是IOC的一个方面，是个通常的概念，它有多种解释。这概念是说你不用创建对象，而只需要描述它如何被创建。你不在代码里直接组装你的组件和服务，但是要在配置文件里描述哪些组件需要哪些服务，之后一个容器（IOC容器）负责把他们组装起来。
2.6有哪些不同类型的IOC（依赖注入）方式？
有哪些不同类型的IOC（依赖注入）方式？
构造器依赖注入：构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参数代表一个对其他类的依赖。
Setter方法注入：Setter方法注入是容器通过调用无参构造器或无参static工厂 方法实例化bean之后，调用该bean的setter方法，即实现了基于setter的依赖注入。
工厂注入：这个是遗留下来的，很少用的了！
2.7哪种依赖注入方式你建议使用，构造器注入，还是 Setter方法注入？
哪种依赖注入方式你建议使用，构造器注入，还是 Setter方法注入？
你两种依赖方式都可以使用，构造器注入和Setter方法注入。最好的解决方案是用构造器参数实现强制依赖，setter方法实现可选依赖。
2.8什么是Spring beans?
什么是Spring beans?
Spring beans 是那些形成Spring应用的主干的java对象。它们被Spring IOC容器初始化，装配，和管理。这些beans通过容器中配置的元数据创建。比如，以XML文件中<bean/>的形式定义。
这里有四种重要的方法给Spring容器提供配置元数据。
XML配置文件。
基于注解的配置。
基于java的配置。
Groovy DSL配置
2.9解释Spring框架中bean的生命周期
解释Spring框架中bean的生命周期
Spring容器 从XML 文件中读取bean的定义，并实例化bean。
Spring根据bean的定义填充所有的属性。
如果bean实现了BeanNameAware 接口，Spring 传递bean 的ID 到 setBeanName方法。
如果Bean 实现了 BeanFactoryAware 接口， Spring传递beanfactory 给setBeanFactory 方法。
如果有任何与bean相关联的BeanPostProcessors，Spring会在postProcesserBeforeInitialization()方法内调用它们。
如果bean实现IntializingBean了，调用它的afterPropertySet方法，如果bean声明了初始化方法，调用此初始化方法。
如果有BeanPostProcessors 和bean 关联，这些bean的postProcessAfterInitialization() 方法将被调用。
如果bean实现了 DisposableBean，它将调用destroy()方法。
2.10解释不同方式的自动装配
解释不同方式的自动装配
no：默认的方式是不进行自动装配，通过显式设置ref 属性来进行装配。
byName：通过参数名 自动装配，Spring容器在配置文件中发现bean的autowire属性被设置成byname，之后容器试图匹配、装配和该bean的属性具有相同名字的bean。
byType:：通过参数类型自动装配，Spring容器在配置文件中发现bean的autowire属性被设置成byType，之后容器试图匹配、装配和该bean的属性具有相同类型的bean。如果有多个bean符合条件，则抛出错误。
constructor：这个方式类似于byType， 但是要提供给构造器参数，如果没有确定的带参数的构造器参数类型，将会抛出异常。
autodetect：首先尝试使用constructor来自动装配，如果无法工作，则使用byType方式。
只用注解的方式时，注解默认是使用byType的！
2.11IOC的优点是什么？
IOC的优点是什么？
IOC 或 依赖注入把应用的代码量降到最低。它使应用容易测试，单元测试不再需要单例和JNDI查找机制。最小的代价和最小的侵入性使松散耦合得以实现。IOC容器支持加载服务时的饿汉式初始化和懒加载。
2.12哪些是重要的bean生命周期方法？ 你能重载它们吗？
哪些是重要的bean生命周期方法？ 你能重载它们吗？
有两个重要的bean 生命周期方法，第一个是setup， 它是在容器加载bean的时候被调用。第二个方法是 teardown 它是在容器卸载类的时候被调用。
The bean 标签有两个重要的属性（init-method和destroy-method）。用它们你可以自己定制初始化和注销方法。它们也有相应的注解（@PostConstruct和@PreDestroy）。
2.13怎么回答面试官：你对Spring的理解？
怎么回答面试官：你对Spring的理解？
来源：
https://www.zhihu.com/question/48427693?sort=created

2.14Spring框架中的单例Beans是线程安全的么？
Spring框架中的单例Beans是线程安全的么？
Spring框架并没有对单例bean进行任何多线程的封装处理。关于单例bean的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的Spring bean并没有可变的状态(比如Serview类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。
最浅显的解决办法就是将多态bean的作用域由“singleton”变更为“prototype”
2.15FileSystemResource和ClassPathResource有何区别？
FileSystemResource和ClassPathResource有何区别？
在FileSystemResource 中需要给出spring-config.xml文件在你项目中的相对路径或者绝对路径。在ClassPathResource中spring会在ClassPath中自动搜寻配置文件，所以要把ClassPathResource文件放在ClassPath下。
如果将spring-config.xml保存在了src文件夹下的话，只需给出配置文件的名称即可，因为src文件夹是默认。
简而言之，ClassPathResource在环境变量中读取配置文件，FileSystemResource在配置文件中读取配置文件。

附录：
@Async标注的方法，同时也使用了@Transactional（事物也是使用了代理的）来声明的注解式事物，这样可能导致事物不生效，因为方法的执行已经异步了。
这个是RxJava和响应式编程的区别？？
http://coyee.com/article/12086-spring-5-reactive-web

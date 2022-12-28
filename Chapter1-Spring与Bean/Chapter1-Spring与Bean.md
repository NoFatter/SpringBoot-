# 1.为什么要使用Spring？

## 简化Java开发

1996年12月，Sun公司发布了JavaBean 1.00-A 规范，针对Java定义了软件组件模型，使简单的Java对象不仅可以被重用，而且可以轻松地构建更复杂的应用。
但是，复杂的应用需要诸如事务支持、安全、分布式计算此类的服务，但JavaBean规范并未直接提供。因此，在1998年3月，Sun发布了EJB 1.0 规范，把Java组件的设计理念延伸到了服务器端，并提供了许多必须的企业级服务。
尽管如此，EJB始终没有实现最初的设想：简化企业级应用开发，EJB在部署描述符和配套代码实现（home、remote和local接口）等方面变得异常复杂。
而Spring作为基于POJO的轻量级开发框架的领导者，引入了AOP（面向切面）和DI（依赖注入）等最新理念，致力于**简化Java开发**这一根本使命上。 

## 为降低Java开发的复杂性，Spring采用了什么策略？

- 基于POJO的轻量级和最小侵入性编程
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模版减少样板式代码

# 2.激发POJO的潜能

## 依赖注入（DI）

### 一个简单的构造器注入

```java
	public class RescuingKnight implements Knight{
		private Quest quest;
		public RescuingKnightQuest(){
			quest = new RescueQuest();
		}
		public void embarkOnQuest(){
			quest.embark();
		}
	}
```

上述代码中可以看见，RescuingKnight类在它的构造函数中自行创建了一个RescueQuest类，这使得Quest类和RescuingKnight类耦合在了一起，只能完成救援任务，极大的限制了该骑士类执行任务的能力。

```java
	public class BraveKnight implements Knight{
		private Quest quest;
		public BraveQuest(Quest quest){
			this.quest = quest;
		}
		public void embarkOnQuest(){
			quest.embark();
		}
	}
```

不同于之前的RescuingKnight类，BraveKnight类在构造时没有自行创建任务，而是将探险任务作为构造器参数传入。这是依赖注入的方式之一，**构造器注入**。
并且，它被传入的任务类型为Quest，是所有探险任务都必须实现的接口，意味着BraveKnight类可以响应任何一种Quest实现，完成任务。

### 装配

创建应用组件之间协作的的行为通常称为装配。
现在，我们的BraveKnight类已经可以接受任何一种Quest类的实现。那么，我们要如何把特定的Quest实现传给它呢？

```xml
	<bean id="knight" class="com.spring.knights.BraveKnight">
		<constructor-args ref="quest" />
	</bean>

	<bean id="quest" class="com.spring.knights.SlayDragonQuest" />
```

这是Spring装配Bean的一种简单方式，我们只需要装载XML文件，并把应用启动起来。

### 工作流程

```java
	public class KnightMain{
		public static void main(String[] args){
			ApplicationContext context = new ClassPathXmlApplicationContext("knights.xml");
			Knight knight = (Knight)context.getBean("knight");
			knight.embarkOnQuest();
		}
	}
```

这里的main()方法创建了一个Spring应用上下文，该应用上下文加载了XML文件。随后，调用上下文，获取ID为knight的Bean。接下来，只需简单调用embark方法，即可让骑士完成所赋予的任务了。



## 应用切面（AOP）

### 什么是AOP

系统由许多不同组件构成，这些组件在实现自身核心功能之外，往往还承担着诸如日志、事务管理和安全等额外的职责。而AOP编程允许你把遍布应用各处的功能分离出来，形成**可重用的组件**

![[截屏2022-12-28 10.47.47.png]]

### AOP的应用

现在，我们想使用一个Minstrel（吟游诗人）作为一个服务类，来记载骑士的所有事迹：
```java
	public class Minstrel{
		public void beforeQuest(){
			System.out.println("The knight is brave!");
		}
		public void afterQuest(){
			System.out.println("The knight did embark on a quest!");
		}
	}
```

接下来，骑士在执行探险任务前，要执行beforeQuest方法；完成任务后，要执行afterQuest方法：

```java
	public class BraveKnight implements Knight{
		private Quest quest;
		private Minstrel minstrel;
		public BraveQuest(Quest quest,Minstrel minstrel){
			this.quest = quest;
			this.minstrel = minstrel;
		}
		public void embarkOnQuest(){
			minstrel.beforeQuest();
			quest.embark();
			minstrel.afterQuest();
		}
	}
```

但问题是，管理吟游诗人应该是骑士的工作吗？
这样的方式无疑会让骑士类变得复杂，但我们利用AOP，可以让骑士摆脱吟游诗人的困扰：

```xml
	<bean id="knight" class="com.spring.knights.BraveKnight">
		<constructor-args ref="quest" />
	</bean>

	<bean id="quest" class="com.spring.knights.SlayDragonQuest" />

	<bean id="minstrel" class="com.spring.knights.Minstrel" />

	<aop:config>
		<!- 定义切面 >
		<aop:aspect ref="minstrel">
			<aop:pointcut id="embark" expressopn="execution(* *.embarkOnQuest(..))" />
			<!- 前置通知 >
			<aop:before pointcut-ref="embark" method="beforeQuest" />
			<!- 后置通知 >
			<aop:after pointcut-ref="embark" method="afterQuest" />
		</aop:aspect>
	</aop:config>
		
```

这里我们定义了名为embark的切入点，使用前置通知和后置通知实现了所需的功能。

# 3.Spring容器

## 容器是什么

在基于Spring的应用中，应用对象生存于Spring容器中。Spring容器创建对象，装配它们，配置它们，管理它们的整个生命周期，从生存到死亡。

容器是Spring框架的核心，Spring自带了几种容器实现，一般归为两种不同的类型：
- Bean工厂
	最简单的容器，提供基本的DI支持。
- 应用上下文
	基于BeanFactory之上构建，提供面向应用的服务，例如从属性文件解析文本信息的能力，以及发布应用事件给感兴趣的事件监听者的能力。

### 常用的应用上下文
- ClassPathXmlApplicationContext
	从类路径下的XML配置文件中加载上下文定义，把应用上下文定义文件当作类资源
- FileSystemXmlApplicationContext
	读取文件系统下的XML配置文件并加载上下文定义
- XmlWebApplicationContext
	读取Web应用下的XML配置文件并装载上下文定义

## Bean的生命周期

1. Spring对Bean进行实例化
2. Spring将值和Bean的引用注入进Bean对应属性中
3. 如果Bean实现了BeanNameAware接口，Spring将Bean的ID传递给setBeanName()接口方法
4. 如果Bean实现了BeanFactorAware接口，Spring将调用setBeanFactory()接口方法，将BeanFactory容器实例传入
5. 如果Bean实现了ApplicationContextAware接口，Spring将调用setApplicationContext()接口方法，将应用上下文的引用传入
6. 如果Bean实现了BeanPostProcessor接口，Spring将调用ProcessBeforeInitialization()接口方法
7. 如果Bean实现了InitizalizingBean接口，Spring将调用它们的afterPropertiesSet()接口方法。类似地，如果Bean使用init-method声明了初始化方法，该方法也会被调用
8. 如果Bean实现了BeanPostProcessor接口，Spring将调用ProcessAfterInitialization()接口方法
9. 此时此刻，Bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁。
10. 如果Bean实现了DisposableBean接口，Spring将调用它的destroy()接口方法。同样，如果Bean使用destroy-method声明了销毁方法，该方法也会被调用。

![[截屏2022-12-28 12.07.03.png]]

## Spring模块

Spring框架是由几个不同模块所构成的。在Spring的dist目录下，我们可以看到20个不同的JAR文件。
这20个JAR文件依据所属功能可以划分为6个不同的功能模块，如图所示：

![[截屏2022-12-28 12.10.00.png]]

- 核心Spring容器：Core Spring container
- AOP模块：AOP
- 数据访问和集成：Data access & integration
- Web和远程调用：Web and remoting
- JVM代理：Instrumentation
- 测试：Testing

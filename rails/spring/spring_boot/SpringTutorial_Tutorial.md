> 看了两个tutorial, 感觉说的都是怎么用, 没有进代码里说sb怎么做的. 有点傻. 
> 倒是两个tutorial 第一篇讲的原理都还行, 哈哈哈哈. SB怎么打包的,  启动会做校验. SB自动配置的原理. 最想知道的流程没有讲... 
>  TODO 感觉自己这周可以把启动过程源码看一下. springboot就过了.


Spring Boot: Configuring a Main Class
	SpringBoot 需要指定项目的Main-Class, 不像普通jar包, 在META-INF里面配置mainClass. 
	SB想要我们把MainClass设置成在JarLauncher/WarLauncher, 把我们的启动类放在里面, 
	这两个类会对我们的包做一些校验啊什么的.
	```java
		Manifest-Version: 1.0
		Start-Class: com.xxxx.xxx.xxx.xxxx.Application
		Main-Class: org.springframework.boot.loader.JarLauncher
	```
	让我们一起看怎么用Maven/Gradle控制这个Start-Class参数, 把我们的放进去.

	1. Maven: pom文件设置参数, 然后parent的maven-jar-plugin就会取出来.
		我猜, 如果不指定, 它就会在classpath下面扫描SpringBootConfiguration这个类.
	```xml
		<properties>
			  <!-- The main class to start by executing "java -jar" -->
			  <start-class>com.baeldung.DemoApplication</start-class>
		</properties>
	```
	2. gradle: 也在springBoot配置block里面加上.


	
Migrating from Spring to Spring Boot
	迁移会保留我们之前的beans, 大部分都是要配置修改. Spring-B的优点:
	1. 更简单的依赖管理, 默认的自动配置, 内置了server, 暴露了metrics可以健康检查, 先进的外部配置.
	
	
	
	
Quick Guide on Loading Initial Data with Spring Boot
	
	


Spring Boot Exit Codes
	spring-boot 项目退出的时候会有exitCode给应用, 启动报错的exitCode是1, 如果其他的正常退出就是0;
	spring注册了JVM的shoutdownHook, 可以优雅的退出. 也提供了接口ExitCodeGenerator可以指定退出码.
	1. ExitCodeGenerator:
		SpringApplication.exit, 主动退出的时候, 会生成退出码.
	2. ExitCodeExceptionMapper
		我们就根据运行时的错误来返回一个exitCode, 我们可以自定义一个ExitCodeExceptionMapper, 把exception转成exitCode;
	3. ExitCodeEvent:
		在退出的时候会有ExitCodeEvent, 我们注册一个eventListener来捕捉这个.
		这个就可以在退出的时候做点事情了.
	总结: 
		1. 主动调用SpringApplication.exit 方法使用ExitCodeGenerator ,可以通过 Bean注册，也可通过传值。
		2. 应用异常退出使用 ExitCodeExceptionMapper, 只能通过 Bean 注册使用。
		3. 取值： 前后正负不同，取最新, 相同，取绝对值大的。


这个.... 就是简单的介绍......... 不太值得看.




========================================================================================================================================



Internationalization in Spring Boot
	SpringBoot的国际化: 可以用interceptor, 从请求里面拿到param, 或者session/cookie之类的, 然后使用LocaleResolver解析参数拿到Locale, 然后用它设置到request里的servlet里面.
	配置文件可以根据Messages_xx.properties后面的xx来区分语言.


How Spring Boot auto-configuration works
	SpringBoot根据classpath里的依赖, 自动的配置一个Spring应用. 自动的配置机制会保证创建和编织必要的bean. 这是最重要的功能之一.
	
	1. Auto-Configuration Report:
		开启debug输出就会有report输出出来. spring-boot-auto-configuration会根据 condition条件加载bean. 
		
	2. Understanding Conditional
		Spring4引入的条件加载, 为的是Spring profile feature.
		
	3. Understanding Auto-configuration
		自动配置, 通过@Configuration 这个注解实现的. 在@Conditional注解决定了这个类/...是否需要自动配置
	
	4. Spring Boot Auto-Configuration:
		@EnableAutoConfiguration 这个注解放在configration上面, 就开启了SpringApplicationContext的自动配置: 扫描classpath下所有的components, 并把需要的bean注册上.
		4.1 定位自动配置的classes:
			为了加载 auto-configuration 的类, spring需要知道在哪里找到配置类. 事实上, 它会去`META-INF/spring.factories`里面找. 这个文件包含了所有的configuration类.
			```java
				// 开启了自动配置之后, 就会用SpringFactoriesLoader去下面的数十个配置类里乱配一通
				org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
				org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
				org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
			```
			
	5. Custom Starter with Spring Boot
		1. 第一步在META-INFO里面放个spring.factories文件, 里面放着k-v是EnableAutoConfiguration注解和我们的配置类mainclass的全限定名.
			spring会去扫描所有文件里面的配置类, 然后会依赖它扫描的classpath下面的类来加载配置文件. 如果找到了JPA, 就load JPAconfiguration.
			
下面的topic里面写的都感觉太水了, 怎么用. 用到再看吧.
How to Log Incoming Requests In Spring
Failure analyzer in Spring Boot
Introduction to Spring Profiles Using Spring Boot
Custom Starter with Spring Boot
Change the default port in Spring Boot
Multiple Data Sources with Spring Boot
How to add a filter in Spring Boot
Spring Boot Starter Parent
​Spring Boot logging with application.properties
​Send Email Using Spring
​Custom Validation MessageSource in Spring Boot
​Configuring Hikari Connection Pool with Spring Boot
​Spring Boot Security Auto-Configuration
​Spring Boot With Caffeine Cache
​Spring Boot CORS
​The @ServletComponentScan Annotation in Spring Boot
​Spring Boot Scheduler
​Spring Boot With MongoDB




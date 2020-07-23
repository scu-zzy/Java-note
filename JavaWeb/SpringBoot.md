# 1. 什么是 Spring Boot? #

首先，重要的是要理解 Spring Boot 并不是一个框架，它是一种创建独立应用程序的更简单方法，只需要很少或没有配置（相比于 Spring 来说）。Spring Boot最好的特性之一是它利用现有的 Spring 项目和第三方项目来开发适合生产的应用程序。

# 2. 说出使用Spring Boot的主要优点  #

1. 开发基于 Spring 的应用程序很容易。
1. Spring Boot 项目所需的开发或工程时间明显减少，通常会提高整体生产力。
1. Spring Boot不需要编写大量样板代码、XML配置和注释。
1. Spring引导应用程序可以很容易地与Spring生态系统集成，如Spring JDBC、Spring ORM、Spring Data、Spring Security等。
1. Spring Boot遵循“固执己见的默认配置”，以减少开发工作（默认配置可以修改）。
1. Spring Boot 应用程序提供嵌入式HTTP服务器，如Tomcat和Jetty，可以轻松地开发和测试web应用程序。（这点很赞！普通运行Java程序的方式就能运行基于Spring Boot web 项目，省事很多）
1. Spring Boot提供命令行接口(CLI)工具，用于开发和测试Spring Boot应用程序，如Java或Groovy。
1. Spring Boot提供了多种插件，可以使用内置工具(如Maven和Gradle)开发和测试Spring Boot应用程序。

# 3. 为什么需要Spring Boot? #

Spring Framework旨在简化J2EE企业应用程序开发。Spring Boot Framework旨在简化Spring开发。

# 4. 什么是 Spring Boot Starters? #

Spring Boot Starters 是一系列依赖关系的集合，因为它的存在，项目的依赖之间的关系对我们来说变的更加简单了。举个例子：在没有Spring Boot Starters之前，我们开发REST服务或Web应用程序时; 我们需要使用像Spring MVC，Tomcat和Jackson这样的库，这些依赖我们需要手动一个一个添加。但是，有了 Spring Boot Starters 我们只需要一个只需添加一个spring-boot-starter-web一个依赖就可以了，这个依赖包含的字依赖中包含了我们开发REST 服务需要的所有依赖。

# 5. 如何在Spring Boot应用程序中使用Jetty而不是Tomcat? #

Spring Boot Web starter使用Tomcat作为默认的嵌入式servlet容器, 如果你想使用 Jetty 的话只需要修改pom.xml(Maven)或者build.gradle(Gradle)就可以了。

# 6. 介绍一下@SpringBootApplication注解 #

	package org.springframework.boot.autoconfigure;
	@Target(ElementType.TYPE)
	@Retention(RetentionPolicy.RUNTIME)
	@Documented
	@Inherited
	@SpringBootConfiguration
	@EnableAutoConfiguration
	@ComponentScan(excludeFilters = {
			@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
			@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
	public @interface SpringBootApplication {
	   ......
	}

----------

	package org.springframework.boot;
	@Target(ElementType.TYPE)
	@Retention(RetentionPolicy.RUNTIME)
	@Documented
	@Configuration
	public @interface SpringBootConfiguration {
	
	}

可以看出大概可以把 @SpringBootApplication 看作是 @SpringBootConfiguration(内部为@Configuration)、@EnableAutoConfiguration、@ComponentScan 注解的集合。根据 SpringBoot官网，这三个注解的作用分别是：

- @EnableAutoConfiguration：启用 SpringBoot 的自动配置机制
- @ComponentScan： 扫描被@Component (@Service,@Controller)注解的bean，注解默认会扫描该类所在的包下所有的类。
- @SpringBootConfiguration(内部为@Configuration)：被标注的类等于在spring的XML配置文件中(applicationContext.xml)，装配所有bean事务，提供了一个spring的上下文环境。

# 7.SpringBoot如何启动的 #

每个SpringBoot程序都有一个主入口，也就是main方法，main里面调用SpringApplication.run()启动整个spring-boot程序，该方法所在类需要使用@SpringBootApplication注解，以及@ImportResource注解(if need)，@SpringBootApplication包括三个注解，功能如下：

- @EnableAutoConfiguration：启用 SpringBoot 的自动配置机制
- @ComponentScan： 扫描被@Component (@Service,@Controller)注解的bean，注解默认会扫描该类所在的包下所有的类。
- @SpringBootConfiguration(内部为@Configuration)：被标注的类等于在spring的XML配置文件中(applicationContext.xml)，装配所有bean事务，提供了一个spring的上下文环境。

启动流程主要分为三个部分：

1. 第一部分进行SpringApplication的初始化模块，配置一些基本的环境变量、资源、构造器、监听器；
1. 第二部分实现了应用具体的启动方案，包括启动流程的监听模块、加载配置环境模块、及核心的创建上下文环境模块；
1. 第三部分是自动化配置模块，该模块作为springboot自动配置核心，下一个问题详细讲解。

# 8. Spring Boot 的自动配置是如何实现的? #

主要是 @SpringBootApplication 中的 @EnableAutoConfiguration 注解

	import java.lang.annotation.Documented;
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Inherited;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;
	import org.springframework.context.annotation.Import;
	
	@Target({ElementType.TYPE})
	@Retention(RetentionPolicy.RUNTIME)
	@Documented
	@Inherited
	@AutoConfigurationPackage
	@Import({AutoConfigurationImportSelector.class})
	public @interface EnableAutoConfiguration {
	    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
	
	    Class<?>[] exclude() default {};
	
	    String[] excludeName() default {};
	}

@EnableAutoConfiguration 注解通过Spring 提供的 @Import 注解导入了AutoConfigurationImportSelector类（@Import 注解可以导入配置类或者Bean到当前类中）。

AutoConfigurationImportSelector类中getCandidateConfigurations方法会将所有自动配置类的信息以 List 的形式返回。这些配置信息会被 Spring 容器当作 bean 来管理。

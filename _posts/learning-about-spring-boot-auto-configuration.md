---
title: Learning About Spring Boot Auto Configuration
tags:
  - Interview
category:
  - Spring Boot
author: bsyonline
lede: 没有摘要
date: 2020-02-25 12:07:18
thumbnail:
---

Spring Boot 自动配置主要依靠注解来实现。
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
}
```
@EnableAutoConfiguration 通过 @Import 引入了 AutoConfigurationImportSelector ，我们需要重点关注一下 getAutoConfigurationEntry() 方法。
<img src="https://s2.ax1x.com/2020/02/25/3tCwt0.png" alt="3tCwt0.png" border="0" />
这个方法通过 SpringFactoriesLoader 加载 META-INF/spring.factories 中的配置，在 spring-boot-autoconfigure 中 auto configuration 配置了 100+ 个类的全名，通过排除，过滤最终剩下需要自动配置的 29 个类。有了类的全名，在 doCreateBean() 方法中通过 beanFactory 创建类的实例。
自动配置除了上边的加载过程，还有一个重要的环节就是条件注解。我们看到在 spring.factories 中配置的类都是 xxxAutoConfiguration 这样的配置类，这些类有一个共同的特点就是都会带一些条件注解，以  JdbcTemplateAutoConfiguration 为例。
```
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class }) //当类路径下有指定类的条件下
@ConditionalOnSingleCandidate(DataSource.class)		      //当指定Bean在容器中只有一个，或者虽然有多个但是指定首选Bean	
@AutoConfigureAfter(DataSourceAutoConfiguration.class)      //自动配置必须在指定类之后进行
@EnableConfigurationProperties(JdbcProperties.class)
@Import({ JdbcTemplateConfiguration.class, NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {

}
```
可以看到，有很多 @ConditionalOnXXX 的注解，这就是条件注解。条件注解的作用就是给 bean 的创建加上一些限制条件，以保证在自动实例化和注入时正确执行。 
综上所述，Spring Boot 的自动配置就是通过 @EnableAutoConfiguration + 条件注解实现的。

>常见的一些条件注解：
@ConditionalOnBean：当容器里有指定Bean的条件下
@ConditionalOnClass：当类路径下有指定类的条件下
@ConditionalOnExpression：基于SpEL表达式作为判断条件
@ConditionalOnJava：基于JV版本作为判断条件
@ConditionalOnMissingBean：当容器里没有指定Bean的情况下
@ConditionalOnMissingClass：当类路径下没有指定类的条件下
@ConditionalOnProperty：指定的属性是否有指定的值
@ConditionalOnResource：类路径是否有指定的值
@ConditionalOnSingleCandidate：当指定Bean在容器中只有一个，或者虽然有多个但是指定首选Bean
## 第二章 装配Bean

三种可选的装配方案：

1、自动装配

2、Java代码配置

3、xml配置



### 2.2 自动装配

从两个角度实现自动装配：

- 组件扫描(component scanning)：自动发现应用上下文中所创建的bean
- 自动装配(autowiring)：Spring自动满足Bean之间的依赖



```java
package com.yuan

@Component
@ComponentScan
public class Config {

}

@ComponentScan 默认扫描与配置类同一包下的所有带有@Component注解的类
对应的xml代码
<context:component-scan base-package="com.yuan">
```



#### 2.2.2 为组件扫描的bean命名

命名规则：

- 若未指定名字，Spring会默认给bean一个ID，ID就是类名的首字母小写

- 也可以用以下方式指定ID

```java
@Component("bei")
public class Dog {
    
}
```





#### 2.2.3 设置组件扫描基础包

```java
- 以下几种方式
@ComponentScan("com.yuan")
@ComponentScan(basePackages="com.yuan")
@ComponentScan(basePackages={"com.yuan.entity", "com.yuan.service"})
@ComponentScan(basePackagesClasses={Dog.class})
```



#### 2.2.4 自动装配@Aurowired

用于自动满足bean的依赖，可用在任何方法上，或者属性域上

```java
@Autowired
public void setDog(Dog d){};

会为Dog查找上下文中满足的bean
```



- 如果**有且只有一个bean**满足依赖，就会被装配进来
- 如果**没有bean**满足，则会抛出一个异常，但是可以设置required=false来避免这个异常

```java
@Autowired(required=false)
```

- 如果有**多个bean**满足依赖，会抛出异常





### 2.3 Java代码装配Bean

有的时候无法使用注解（自动装配）进行装配，这个时候就需要显示地进行配置，**进行显示配置时，JavaConfig通常是更好的方法**，JavaConfig是配置代码，而非业务代码。



#### 2.3.1配置类

```java
@Configuration
public class VehicleConfig() {
}
```



#### 2.3.2声明Bean

```java
@Configuration
public class VehicleConfig() {
    @Bean
    public Wheel wheel() {
        return new Wheel;
    }
    
    @Bean(name="engine")
    ...
}
```



@Bean方法体内为创建新对象的逻辑，可以通过name属性指定相关的名字，默认情况下name为方法名的名字



#### 2.3.3 使用javaconfig进行注入

```java
@Configuration
public class VehicleConfig() {
    @Bean
    public Wheel wheel() {
        return new Wheel();
    }
    
    @Bean
    public Plate plate() {
        return new Plate(wheel())
    }
}
```



在这里Spring会拦截Plate() 构造器内的wheel()方法，而直接返回一个wheel对象

因为如果调用方法的话，就会产生很多个wheel实例



上述方法最好改为下列形式，这样不要求两者声明到一个配置类中

```java
@Bean
public Plate plate(Wheel wheel) {
    return new Plate(wheel)
}
```





### 2.4 通过xml装配Bean



#### 2.4.1 创建xml配置规范

创建一个xml文件，以`<beans>`标签为根

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  
</beans>
```

定义一个bean最基本的xml元素包含在spring-beans模式中



#### 2.4.2 声明一个bean

``` xml
<bean id="wheel" class="com.yuan.Wheel"/>

一般只对需要引用的bean指定相关的id
默认id为"com.yuan.Wheel#0" #0代表计数
```



- Spring会调用Wheel的默认构造器来创建这个bean
- class以字符串形式传入，无法在编译器进行类型检查



#### 2.4.3 借助构造器初始化一个bean

两种方式：

- 使用constructor-arg
- 使用c命名空间



```xml
<bean id="wheel" class="com.yuan.Wheel"/>
<bean id="vehicle" class="com.yuan.Vehicle">
	<constructor-arg ref="wheel"/>
</bean>
```

 ```xml
<bean id="wheel" class="com.yuan.Wheel"/>
<bean id="vehicle" class="com.yuan.Vehicle" c:wheel-ref="wheel"(或c:_0-ref="wheel")/>

c:wheel-ref: c:参数名-ref:
c:_0-ref: c:_参数位置-ref:

如果只有一个构造器参数，也可以使用
<bean id="vehicle" class="com.yuan.Vehicle" c:_-ref="wheel"/>
 ```



装配字面量

- 使用value属性

```xml
<bean id="vehicle" class="com.yuan.Vehicle">
	<constructor-arg value="porsche 911"/>
</bean>
<bean id="vehicle" class="com.yuan.Vehicle" c:_0="porsche 911"/>
```



装配集合

```xml
<bean id="vehicle" class="com.yuan.Vehicle">
	<constructor-arg>
        <list>
            <ref bean="LeftWheel"/>
            <ref bean="RightWheel"/>
        </list>
</bean>
```

当构造参数是List时，可以用list标签；当构造参数为Set时，也可以使用set标签

list和set可以也用来装配数组，而c-命名空间无法装配集合



#### 2.4.4 setter方法注入



使用property标签完成setter注入，一般对于强依赖的属性，用构造方法注入，对于可选择依赖，可用Setter注入

- property标签
- p-命名空间：用法与c-相似

```xml
<bean id="vehicle" class="com.yuan.Vehicle">
	<property name="Passenger" ref="yuan"/>
</bean>
```



装配字面量

- 使用value属性



装配集合

- 使用list或者set



util命名空间，可以使用util:list元素，这会创建一个列表bean，在其他地方可以进行引用

```xml
<util:list id="tracklist">
	<value>1</value>
    <value>2</value>
    <value>3</value>
</util:list>

<bean id="vehicle" class="com.yuan.Vehicle" p:nums-ref="tracklist">
```



### 2.5 混合使用

1、在JavaConfig中引入其他JavaConfig配置   `@Import`

```java
@Configuration
@Import(EngineConfig.class)
public class VehicleConfig {
    
}
```



2、在JavaConfig中引入xml配置  `@ImportResource`

```java
@Configuration
@ImportResource("classpath:engine-config.xml")
public class VehicleConfig {
    
}
```



3、在xml配置中引入xml配置  `<import>`

```xml
<import resource="engine-config.xml"/>
```



4、在xml配置中引入JavaConfig配置  `<bean>`

```xml
<bean class="EngineConfig.class"/>
```





## 第三章 高级装配



### 3.1 环境与Profile

#### 3.1.1 配置profile

在JavaConfig中，通过 `Profile` 注解来标注配置类或是bean注解的方法

```java
public class Config {
    @Bean
    @Profile("dev")
    public Datasource datasource(){
        ...
    }
}

@Configuration
@Profile("prod")
public class Config {
    @Bean
    @Profile("dev")
    public Datasource datasource(){
        ...
    }
}
```



在xml配置中，通过profile属性指定`<beans>`标签的profile，也可以在`<beans>`中嵌套`<beans>`

```xml
<beans profile='dev'>
</beans>

<beans>
    <beans profile="dev"></beans>
    <beans profile="prod"></beans>
</beans>
```



#### 3.1.2 激活不同的profile

设置两个属性，来激活不同的profile，有多种方法可以设置这两个属性，如果都未设置，则只会加载那些没有标注Profile的Bean

```
spring.profiles.default
spring.profiles.active
```



### 3.2 条件化的Bean

`@Conditional`注解，使得bean在特定的条件下才进行创建

```java
@Bean
@Conditional(WheelExistCondition.class)
```

该注解通过Condition接口进行判断，即上述`WheelExistCondition`需要实现Condition接口，该接口只包含一个match方法，返回true则装配该Bean

```java
Boolean match(ConditionContext context, AnotatedTypeMetadata metadata);
```

通过ConditionContext接口，可以bean的定义，属性，环境变量，加载的资源等等。

通过AnotatedTypeMetadata，可以得到@Bean注解的方法是否还有其他的注解等



### 3.3 自动装配的歧义性

当有多个Bean满足自动装配时，会产生歧义，解决方法：



1、将其中一个可选的bean设置为primary，使用`@Primary`注解，可同时和`@Component`，`@Bean`使用，在xml配置中，可以设置primary="true"

2、添加限定符，缩小匹配的范围，使用`@Qualifier`注解，可以与`@Autowired`一起使用

```java
@Autowired
@Qualifier("leftWheel")
public Wheel wheel;
```

上述表名装配的bean包含字符串类型leftWheel的限定符，当创建Bean时，如果未指定限定符，则其限定符为bean的id

当创建带有限定符的bean时，也可以使用`@Qualifier`注解，使其搭配`@Component`，`@Bean`使用，通常，限定符用于指定bean的特性，而非id



### 3.4 Bean作用域

默认情况下，Bean以单例的模式进行创建



Spring中Bean的作用域：

- 单例（Singleton）：整个应用中，只创建一个对象
- 原型（Prototype）：每次注入或者是根据上下文请求对象时，创建新的
- 会话（Session）：Web应用中，为每一个会话创建一个新的对象
- 请求（Request）：Web应用中，为每一次请求创建一个新的对象



用`@Scope`注解创建不同的作用域，与`@Component`，`@Bean`搭配使用

```java
@Component   //或者@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)

也可以@Scope("prototype")，但上述更安全
```

```xml
<bean id... scope="prototype"...></bean>
```



@Scope包含属性 proxyMode

```java
@Scope(value="", proxyMode="ScopeProxyMode.INTERFACES")
需要代理的为一个接口

@Scope(value="", proxyMode="ScopeProxyMode.TARGET_CLASS")
需要代理的为一个类
```



当在一个单例的bean中注入一个会话类型的bean时，由于单例bean是上下文构建的时候创建的（此时会话类型的bean还未创建），Spring会创建一个代理类注入到相应的单例bean中



xml中，使用下列指定代理模式

```xml
<aop:scoped-proxy proxy-target-class="true(false)"/>
默认使用CGLIB代理
```





### 3.5 运行时值注入

为了避免将值硬编码，使用属性占位符或是spring表达式语言完成值的注入

#### 3.5.1 注入外部值

使用`@PropertySource`引入外部配置文件

```java
@Configuration
@PropertySource("classpath:value.properties")
public class config {
    @Autowired
    Environment env;
    
    @Bean
    public ... {
        env.getProperty("jdbc.url");
    }
}

String getProperty(String key, String default);
T getProperty(String key, Class<T> type, T default);

getRequiredProperty: 给定的值必须被定义
```



**spring占位符：**

在xml中，使用`${}`占位符，引入外部值

```xml
<bean c:_url="${jdbc.url}"/>

需要配置
<beans>
    <context:property-placeholder/>
</beans>
```

在java代码中，使用`@Value`注解

```java
public Datasource datasource(@Value("$(jdbc.url)") String url, 
                             @Value("$(jdbc.user)") String user...)
    
使用Value时，需要配置一个Bean时上述配置生效

@Bean
public PropertySourcesPlaceHolderConfigurer ... {
    return new PropertySourcesPlaceHolderConfigurer();
}
```



#### 3.5.2 Spring表达式语言进行装配(SpEL)

`#{}` 表达式



**表示字面量**：可用于表示浮点数 字符串 布尔等类型

```
#{3.1415} #{"stirng"} #{false}
```

**引用Bean、属性、方法**：可以利用SpEL将一个bean装入到另一个bean中，这时使用的就是bean的id值

```
#{vehicle}
#{vehicle.wheel}
#{vehicle.stop()}
#{vehicle.name()?.toUpperCase()}  这里name返回一个String，可以调用后续String的方法，?表示进行类型检查，是一个可选的参数
```

**使用类型**：T()

```
#{T(java.lang.Math)}

这里T返回一个Class对象，可以装入特定的Class类型中，但是T通常用来访问静态属性以及静态方法
#{T(java.lang.Math).PI}
```



SpEL表达式同时提供多种运算符





## 第九章 Spring Security

### 9.1 简介

#### 9.1.1 模块

Spring Security共分为11个模块

ACL 切面 CAS客户端 配置 核心 加密 LDAP OPENID Remoting 标签库(Tag Library) Web

至少需要包含配置（Configuration）和核心（Core）模块



#### 9.1.2 过滤Web请求

Spring Security通过一系列的Servlet Filter提供安全性功能

`DelegatingFilterProxy`是一个特殊的Filter，它将工作委托给一个java.servlet.Filter的实现类，这个实现类是作为一个Bean注册到Spring的上下文中

`springSecurityFilterChain`是另一个filter，可以链接一个或多个Fileter



#### 9.1.3 编写简单的安全性配置

`@EnableWebSecurity`注解能够启用Web安全性，但本身无作用

配置必须写在实现了`WebSecurityConfigurer`中，或者扩展`WebSecurityConfigurerAdapter`，

可以通过重载`WebSecurityConfigurerAdapter`中的三个方法来控制安全行为

| 方法                                      | 作用                          |
| ----------------------------------------- | ----------------------------- |
| configure(WebSecurity)                    | 配置Spring Security的Filter链 |
| configure(HttpSecurity)                   | 配置通过拦截器拦截http请求    |
| configure(AuthentificationManagerBuilder) | 配置用户相关服务              |



### 9.2 选择查询用户详细信息的服务

#### 9.2.1 基于内存

`AuthentificationManagerBuilder`的`isMemoryAuthentication`方法，配置基于内存的用户服务

#### 9.2.2 基于数据库表

`AuthentificationManagerBuilder`的`jdbcAuthentication`方法

可以使用PasswordEncoder接口对密码进行加密



### 9.3 拦截请求

通过`configure(HttpSecurity)`方法进行配置

```java
protected void configure(HttpSecurity http) {
    http
        .authorizeRequests()
        .antMatchers().authenticated()
        .regexMatchers().permitAll();
}

antMatchers 指定路径（支持Ant形式的通配符）
regexMatchers 正则式路径
authenticated 需要认证
permitAll 放行
```



#### 9.3.1 Spring表达式实现多维认证

上述对某一路径的拦截是一维的，如果需要多维度认证，使用`access(String)`加SpEL表达式

```java
protected void configure(HttpSecurity http) {
    http.authorizeRequests()
        .antMatchers().access("hasRole('ROLE_ROOT') and hasIpAddress('192.168.1.1')")
}
```



#### 9.3.2 强制通道的安全性

除了`authorizeRequests()`外，`HttpSecurity`还有`requireChannel()`，可强制要求使用https协议传输

```java
protected void configure(HttpSecurity http) {
    http.authorizeRequests()
        .antMatchers().access("hasRole('ROLE_ROOT') and hasIpAddress('192.168.1.1')")
        .and()
        .requireChannel().antMatchers().requiresSecure(); //需要https
}
requiresInsecure //始终使用http传输
```



#### 9.3.3 防止跨站请求伪造

Spring Security通过同步一个token实现CSRF防护的功能，拦截状态变化的请求并检查CSRF token



### 9.4 认证用户

调用httpSecurity.formLogin()可以提供一个默认的登录页

#### 9.4.1 添加自定义的登录页




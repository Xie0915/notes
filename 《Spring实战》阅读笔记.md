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


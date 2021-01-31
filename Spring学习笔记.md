[TOC]



### 配置文件的加载

​        ClassPathXmlApplicationContext在加载Spring的配置文件时，是从类路径中加载。即输出目录target下的classes文件目录下，因为通过maven构建的项目在maven编译后，resources文件夹下的文件是拷贝到了target的classess目录下。

### 依赖注入（DI）

#### 一、属性注入的原理

属性注入时，会根据配置的属性名去找对应的set方法，而不是去验证类中实际是否有这个属性。比如：

spring的xml文件配置：

```xml
<bean id="myStudent" class="com.tanci.study01.Student">
    <property name="name" value="李四"/>
    <property name="age" value="18"/>
    <property name="email" value="234@qq.com"/>
</bean>
```
student类：

```Java
public class Student {
    int age;
    String name;
    public void setAge(int age) {
        this.age = age;
    }
    public void setName(String name) {
        this.name = name;
    }
    public void setEmail(String email) {
        System.out.println("虽然没有email属性但我依然执行了此方法");
    }
}
```

```java
测试方法：
@Test
public void test01(){
    String config = "applicationContext.xml";
    ApplicationContext ac = new ClassPathXmlApplicationContext(config);
}

测试结果：
虽然没有email属性但我依然执行了此方法
```

#### 二、自动注入

自动注入只针对引用类型，不针对简单类型

* byName (按名称注入)
* byClass （按类型注入）

    ① 注入与bean引用类值相同的。

    ② 注入bean中引用类型的子类。

    ③ 注入bean中引用类型的实现类。
  
    注意：xml中配置时，只能有一个符合注入类型的bean.
    

### 多文件配置

每个文件都会比较小

* 避免多人竞争带来的冲突
* 分配方式
  * 按功能模块分
    
  * 按类的功能分
    
  * 主文件一般不定义对象，用来包含其他文件
    * 语法：&lt import resource= "其他文件路径"/ &gt
    * 关键字："classPath"表示的是类路径，即编译输出的target目录下的classes目录的路径。
    * 完整的示例：\<import resouce="classPath:****.xml"/\>
    * 还可以使用通配符一次性加载所有功能相同的文件,如以下三个文件：

       * \<import resouce="classPath:num01.xml"/\>
       * \<import resouce="classPath:num02.xml"/\>
       * \<import resouce="classPath:num02.xml"/\>

    都可以用\<import resouce="classPath:num-*.xml"/\>来代替,但要注意主文件不要也在通配符的包含范围之类。

    ***注意:被加载的所有的文件应该与主配置文件在同一个目录下才能保证被加载****

### 关于注解

#### 一、注解的使用步骤

​	1.加入maven的依赖，spring-context，加入spring-context的同时，简介加入了spring-aop的依赖，使用注解必须使用spring-aop。

​	2.在类中加入spring的注解（多个不同功能的注解）。

​	3.在spring的配置文件中需要加入一个组件扫描，说明注解在项目中的位置。

> 组件扫描器扫描多个包时的用法：
>
> * 配置多个组件扫描器标签
> * 一个标签中用；或者，隔开
> * 包名写成要扫描的多个包的父包（不建议用，因为会造成Spring启动过慢，扫描多个无用的包）

​	4.主要注解有：

> @component
>
> @Repository
>
> @Service
>
> @Controller
>
> @Value
>
> @Autowired
>
> @Resource

#### 二、注解的相关含义

* @Component (value = "对象名")

  > * 创建对象的，等同于\<bean/\>
  > * value就是对象名称，相当于\<bean/\>中的id
  > * value的值是唯一的，创建的对象在spring容器中只有一个
  > * 注解中只有value时，value这词可以省略
  > * 不指定对象名称的时候，spring会默认一个对象名称，类名的小写

* @Repository：放在持久层的注解，dao的实现类，也是创建对象的

* @Service：用在业务层，还可控制事务，创建对象

* @Controller：用在控制器层，创建对象

    后三个除了能创建对象外还有额外的功能，是能给项目分层的。

> **什么时候用component？**
>
> 当要创建的类既不属于controller，也不属于service和dao的时候

* @Value：给类中的属性赋值，也可放在set方法上（用的比较少）

    > @Value(value = "18")
    >
    > private int age;

    引用类型的赋值

* @Autowired：Spring使用时用到的是自动注入原理，默认是byType方法。使用时主要加在属性上。

    > 注解的引用类型变量其类必须已经加了创建对象和属性赋值的相关注解（或者配置文件使用了相关的bean标签）
    >
    > 比如下面student类中有school类型的引用变量

    ```java
    student类：
    @Autowired
    private School school;
    School 类：
    @Component("value = school")
    public School class{
    ​		@Value (value = "**")
    ​		private address;
    }
    ```

#### 三、注解的原理

配置文件中配置组件扫描器，扫描指定的包，包可以根据idea导入类的路径copy，包名千万不能写错。

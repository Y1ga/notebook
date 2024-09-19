# 1. 初识Spring

## 优点

1. spring是开源的免费的容器(框架
2. 轻量级、非入侵式的框架
3. 控制反转(IOC)、面向切面编程(AOP)
4. 支持事务的处理，对框架整合的支持

**总结：spring是轻量级的控制反转(IOC)和面向切面编程(AOP)的框架**

![image-20240615163543241](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240615163543241.png)

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>6.1.8</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>6.1.8</version>
</dependency>
```

## 组成

7大模块

![img](https://img-blog.csdn.net/20180914091500764?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1YW5nMTE3ODM4Nzg0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 拓展

![image-20240614113439638](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240614113439638.png)

- spring boot
  - 一个快速开发的脚手架
  - 基于spring boot可以快速开发单个微服务
- spring cloud
  - 基于springboot实现

spring发展了太久，违背了原来的理念，配置十分繁琐，称为配置地狱

# 2. IOC

- 之前，程序主动创建对象，控制权在程序员手上

- 使用set注入后，程序不再有主动性，而是在被动接收对象，实现了**控制反转**，这就是**IOC原型**
- 程序耦合性大大降低，使代码专注于业务代码

![image-20240615123421585](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240615123421585.png)

```java
public class UserServiceImpl implements UserService{
    private UserDao userDao = new UserDaoMysqlImpl();

    // 动态实现set注入
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    public void getUser(){
        userDao.getUser();
    }
}
```



```java
// MyTest冒充servlet
public class MyTest {
    public static void main(String[] args) {
        // 之前，要调用mysqlservice需要在UserServiceImpl程序中更改userDao的运行类型
        UserServiceImpl userService = new UserServiceImpl();
        userService.getUser();
        // 改之后，使用set方法后不再在具体的service程序中手动更改userDao的运行类型
        UserServiceImpl userService = new UserServiceImpl();
        ((UserServiceImpl) userService).setUserDao(new UserDaoMysqlImpl());
        userService.getUser();
    }
}
```

## IOC创建对象

三种方式

```xml
    <!--第一张方式，下标赋值-->
    <bean id="user1" class="com.league.pojo.User">
        <constructor-arg index="0" value="jacjk"/>
    </bean>
    <!--第二种方式，不推荐，类型赋值-->
    <bean id="user2" class="com.league.pojo.User">
        <constructor-arg type="java.lang.String" value="jack"/>
    </bean>
    <!--第三种方式，参数名赋值-->
    <bean id="user3" class="com.league.pojo.User">
        <constructor-arg name="str" value="jack"/>
    </bean>
```

配置文件加载的时候，容器中的对象已经被创建

## Spring配置

### name

可以起多个别名

```xml
    <bean id="user1" class="com.league.pojo.User" name="user01, user02">
        <constructor-arg index="0" value="jacjk"/>
```

### import

合并配置文件

applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 合并配置文件-->
    <import resource="beans.xml"/>
</beans>
```

## 构造器注入

### c命名空间

用于构造器注入，前提是要写有参构造器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.league.pojo.Student" p:name="jack" c:name="jack2"/>

</beans>
```



## set()方式注入

- 依赖：bean对象的创建依赖于容器
- 注入：bean对象的所有属性，由容器来注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="address" class="com.league.pojo.Address"/>
    <bean id="student" class="com.league.pojo.Student">
<!--        第一种注入，值注入-->
        <property name="name" value="jack"/>
<!--        第二种注入，bean注入-->
        <property name="address" ref="address"/>
<!--        注入数组-->
        <property name="books">
            <array>
                <value>红楼梦</value>
                <value>三国演义</value>
            </array>
        </property>
<!--        注入List-->
        <property name="hobbies">
            <list>
                <value>游泳</value>
                <value>游戏</value>
            </list>
        </property>
<!--        注入Map-->
        <property name="card">
            <map>
                <entry key="身份证" value="4412"/>
                <entry key="银行卡" value="4396"/>
            </map>
        </property>
<!--        注入Set-->
        <property name="games">
            <set>
                <value>LOL</value>
                <value>Dota2</value>
            </set>
        </property>
<!--            注入null-->
        <property name="wife">

            <null/>
        </property>
<!--        注入properties-->
        <property name="info">
            <props>
                <prop key="学号">2233</prop>
                <prop key="性别">Man</prop>
            </props>
        </property>
    </bean>
</beans>
```

### p命名空间

用于property注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       <!--导入p命名空间-->
        xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">
	<!--此处的p就是property嘛-->
    <bean id="user" class="com.league.pojo.Student" p:name="jack" />
</beans>
```

## Bean作用域

![image-20240615145605815](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240615145605815.png)

```xml
<bean id="user" class="com.league.pojo.Student" p:name="jack" scope="singleton" />
```

- 单例：每次从容器中get产生的是**同一个对象![image-20240615145913997](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240615145913997.png)**

- 原型：每次从容器中get都会产生一个**新对象![image-20240615145858469](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240615145858469.png)**

- 其他：用于web

  

## Bean的自动装配

- 自动装配是spring满足bean依赖的一种方式
- spring在上下文中自动寻找，并自动给bean装配属性

autowire="byName" 或者"byType"，根据名字或者类型自动装配实例对象，避免直接赋值装配

```xml
    <bean id="cat" class="com.league.pojo.Cat"/>
    <bean id="dog" class="com.league.pojo.Dog"/>
    
    <bean id="person1" class="com.league.pojo.Person" autowire="byName">
        <property name="name" value="jack"/>
<!--        <property name="cat" ref="cat"/>-->
<!--        <property name="dog" ref="dog"/>-->
    </bean>
```

## Bean的自动注解

先导入约束，在配置注解的支持

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		https://www.springframework.org/schema/context/spring-context.xsd">

	<context:annotation-config/>

</beans>
```

### @Autowired

```java
public class Person {
    @Autowired
    private Cat cat;
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">
	<!--注意需要注册注解驱动-->
    <context:annotation-config/>
    <bean id="cat" class="com.league.pojo.Cat"/>
    <bean id="dog" class="com.league.pojo.Dog"/>
    <bean id="person" class="com.league.pojo.Person"/>

</beans>
```

### @Qualifier

存在多个对象时，指定某个对象

```java
public class Person {
    @Autowired
    // 指定cat2对象自动装配
    @Qualifier(value = "cat2")
    private Cat cat;
```

```xml
    <context:annotation-config/>
    <bean id="cat" class="com.league.pojo.Cat"/>
    <bean id="cat2" class="com.league.pojo.Cat"/>
    <bean id="dog" class="com.league.pojo.Dog"/>
    <bean id="dog2" class="com.league.pojo.Dog"/>
    <bean id="dog3" class="com.league.pojo.Dog"/>
    <bean id="person" class="com.league.pojo.Person"/>
```



### @Resource

jdk的注解，也可以自动装配，jdk8以后已删除

区别：

- resource使用的是**byName**，如果找不到名字则使用**byType**，两个都找不到则报错

- autowired要求对象必须存在，使用的是**byType**，autowired更常用

### @Component

代表某个类注册到spring中装配bean

```xml
	<!--指定要扫描的包，这个包下的注解就会生效-->
	<context:component-scan base-package="com.league.pojo"/>
    <context:annotation-config/>
```



```java
// 等价于 <bean id="user" class="com.league.pojo.User/>
@Component
// 等价于 <bean id="user" class="com.league.pojo.User scope="singleton"/>
@Scope("singleton")
public class User {

    public String name;
    // 默认传值    
    @Value("jack")
    public void setName(String name) {
        this.name = name;
    }
}
```

表示组件，直接由Spring托管

衍生注解，四个注解本质是一样的，都是代表某个类注册到spring中装配bean

#### @Repository

表示dao

#### @Service

表示service

#### @Controller

表示controller

### 区别

- xml万能，适合各种场合，维护方便
- 注解不是自己类用不了，维护麻烦

一般**xml用来管理bean，注解负责注入**

## Java代替xml

#### pojo类

```java
@Component
public class User {
    private String name;
    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
    public String getName() {
        return name;
    }
    @Value("ff") // 属性注入之
    public void setName(String name) {
        this.name = name;
    }
}

```

#### config类

```java
@Configuration // 这个也会被Spring容器托管，注册到容器中，本身也是component
// @Configuration代表这是一个配置类
// 相当于<context:component-scan base-package="com.league.pojo"/>
@ComponentScan("com.league.pojo")
// 相当于<import resource="beans.xml"/>
@Import(MyConfig.class)
public class MyConfig {
    // 方法名getUser相当于bean中的id属性，返回值相当于bean中的class属性
    @Bean
    public User getUser(){
        return new User();
    }
}
```

#### 测试类

```java
public class MyTest {
    public static void main(String[] args) {
        // 如果使用配置类方式去做，只能通过annotationconfig来获取容器
        ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        User getUser = (User) context.getBean("getUser");
        System.out.println(getUser.getName());
    }
}
```

## 代理模式

工具人

### 静态代理

- 抽象角色：用接口或抽象类解决
- 真实角色：被代理的角色
- 代理角色：代理真实角色，代理真实角色后，做一些附属操作
- 客户：访问代理对象的人



#### 优点

1. 使真实角色操作更加纯粹，不用关注公共事务
2. 公共也交给代理角色，实现业务分工
3. 公共业务发生拓展时，方便集中管理

#### 缺点

1. 一个真实角色就产生一个代理角色，代码量翻倍，开发效率降低

#### 实现步骤

1. 接口

   ```java
   public interface Rent {
       public void rent();
   }
   ```

2. 真实角色

   ```java
   public class Host implements Rent{
   
       @Override
       public void rent() {
           System.out.println("Rent Running");
       }
   } 
   ```

3. 代理角色

   ```java
   public class Proxy implements Rent{
       private Host host;
   
       public Proxy(Host host) {
           this.host = host;
       }
       @Override
       public void rent(){
           host.rent();
       }
   
       public void seeHouse(){
           System.out.println("look house");
       }
   }
   
   ```

4. 客户端访问代理角色

   ```java
   public class Cient {
       public static void main(String[] args) {
           Host host = new Host();
           Proxy proxy = new Proxy(host);
           proxy.rent();
       }
   }
   ```

### 动态代理

- 动态代理和静态代理角色一样
- 动态代理的代理类是动态生成的，不是直接写好的
- 动态代理分为2类：基于接口的动态代理，基于类的动态代理
  - 基于接口：JDK动态代理
  - 基于类：cglib
  - java字节码实现：JAVAssist

需要了解2个类：Proxy：代理；InvocationHandler：调用处理程序

#### 优点

1. 静态的优点都有

2. 一个动态类代理的是一个接口，一般就是对应的一类业务

   

1. 代理角色

   ```java
   // 自动生成代理类
   public class ProxyInvocationHandler implements InvocationHandler {
   
       // 被代理的接口
       private Object target;
   
       public void setTarget(Object target) {
           this.target = target;
       }
   
       // 提供创建动态代理类和实例的静态方法，生成得到代理类
       public Object getProxy(){
           return Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
       }
   
       // 处理代理实例，并返回结果
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
           // 反射获得方法
           log(method.getName());
           return method.invoke(target, args);
       }
   
       public void log(String msg){
           System.out.println("执行了"+msg+"方法");
       }
   
   }
   ```

2. 客户端访问代理角色

   ```java
   public class Clinet{
       // 真实角色
   	UserServiceImpl userService = new UserServiceImpl();
   	// 代理角色，不存在
   	ProxyInvocationHandler pih = new ProxyInvocationHandler();
   	// 设置要代理的对象
   	pih.setTarget(userService);
   	// 动态生成代理类
   	UserService proxy = (UserService) phi.getProxy();
   	proxy.query();
   }
   ```

   

# 3.AOP

Aspect Oriented Programming，面向切面编程



![image-20240708151021162](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240708151021162.png)

## spring实现AOP

1. service类

   ```java
   package com.league.service;
   
   public interface UserService {
       public void add();
       public void delete();
       public void update();
       public void select();
   
   }
   ```

2. MethodBeforeAdvice类

   ```java
   package com.league.log;
   import org.springframework.aop.MethodBeforeAdvice;
   import java.lang.reflect.Method;
   
   public class Log implements MethodBeforeAdvice {
   
       @Override
       // method：要执行的对象的方法
       // args：参数
       // target：目标对象
       public void before(Method method, Object[] args, Object target) throws Throwable {
           System.out.println(target.getClass().getName()+"的"+method.getName()+"被执行了");
       }
   }
   ```

3. MethodAfterAdvice类

   ```java
   package com.league.log;
   import org.springframework.aop.AfterReturningAdvice;
   import java.lang.reflect.Method;
   
   public class AfterLog implements AfterReturningAdvice {
       // returnValue：返回值
       @Override
       public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
           System.out.println("执行了"+method.getName()+"返回结果为："+returnValue);
       }
   }
   ```

4. 配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
       <!--注册bean-->
       <bean id="userService" class="com.league.service.UserServiceImpl"/>
       <bean id="log" class="com.league.log.Log"/>
       <bean id="afterLog" class="com.league.log.AfterLog"/>
   <!--    方式1：使用原生spring api接口-->
       <!--配置aop：需要导入aop配置-->
       <aop:config>
           <!--切入点：execution：要执行的位置-->
           <aop:pointcut id="pointcut" expression="execution(* com.league.service.UserServiceImpl.*(..))"/>
           <!--执行环绕增加，pointcut-ref：切入点-->
           <!--log类在方法add执行前执行-->
           <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
           <!--afterlog方法在add方法执行后再执行-->
           <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
       </aop:config>
   
       <!--方式2：自定义类实现AOP-->
       <bean id="diy" class="com.league.diy.DiyPointCut"/>
       <aop:config>
           <aop:aspect ref="diy">
               <!--切入点-->
               <aop:pointcut id="point" expression="execution(* com.league.service.UserServiceImpl.*(..))"/>
               <!--通知-->
               <aop:before method="before" pointcut-ref="point"/>
               <aop:after method="after" pointcut-ref="point"/>
           </aop:aspect>
       </aop:config>
   </beans>
   ```

5. 客户端执行

   ```java
   import com.league.service.UserService;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.support.ClassPathXmlApplicationContext;
   
   public class MyTest {
       public static void main(String[] args) {
           ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           // 动态代理的是接口，不是具体实现类！此处应为UserService而不是UserServiceImpl
           UserService userService = context.getBean("userService", UserService.class);
           userService.add();
       }
   }
   /*输出：
   com.league.service.UserServiceImpl的add被执行了
   增加
   执行了add返回结果为：null
   */
   ```

6. 自定义类实现AOP

   ```java
   public class DiyPointCut {
       public void before(){
           System.out.println("方法执行前");
       }
       public void after(){
           System.out.println("方法执行后");
       }
   }
   ```

## 注解实现AOP

1. 切入点

   ```java
   package com.league.diy;
   
   import org.aspectj.lang.ProceedingJoinPoint;
   import org.aspectj.lang.annotation.After;
   import org.aspectj.lang.annotation.Around;
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Before;
   
   /**
    * @author yiga
    * @date 2024/07/08 15:04
    **/
   // 标注这是一个AOP
   @Aspect
   public class AnnotationPointCut {
       @Before("execution(* com.league.service.UserServiceImpl.*(..))")
       public void before(){
           System.out.println("before方法执行");
       }
       @After("execution(* com.league.service.UserServiceImpl.*(..))")
       public void after(){
           System.out.println("after方法执行");
       }
       @Around("execution(* com.league.service.UserServiceImpl.*(..))")
       public void around(ProceedingJoinPoint joinPoint) throws Throwable {
           System.out.println("around方法执行前");
           System.out.println("signature = " + joinPoint.getSignature());
           // 执行方法
           Object proceed = joinPoint.proceed();
           System.out.println("around环绕后");
       }
   }
   ```

2. ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
   		https://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
       <!--注册bean-->
       <bean id="userService" class="com.league.service.UserServiceImpl"/>
       <bean id="log" class="com.league.log.Log"/>
       <bean id="afterLog" class="com.league.log.AfterLog"/>
   
       <!-- 方法3，注解实现-->
       <bean id="annotationPointCut" class="com.league.diy.AnnotationPointCut"/>
       <!--proxy-target-class="false表示用jdk编译，true表示cglib编译-->
       <aop:aspectj-autoproxy proxy-target-class="false"/>
   </beans>
   ```



# 整合MyBaits

1. 导入jar包
   - junit
   - mybaits
   - mysql数据库
   - spring相关的
   - aop织入
2. 编写配置文件
3. 测试

## 回忆

1. 编写实体类
2. 编写核心配置文件
3. 编写接口
4. 编写Mapper.xml
5. 测试

# SpringBoot

在https://start.spring.io/初始化springboot

pom.xml中mysql相关需要注释掉

```
<!--       <dependency>-->
<!--          <groupId>org.springframework.boot</groupId>-->
<!--          <artifactId>spring-boot-starter-data-jpa</artifactId>-->
<!--       </dependency>-->
```

## rest api规范

### 路径

又称为终点（endpoint），表示API的具体网址

在Restful架构中，每个网址代表一种资源，所以网址中不能有动词只能有名词，而且所用名词往往与数据库的表格词相对应

### HTTP 动词

- GET（SELECT）：从服务器中取出资源
- POST（CREATE）：在服务器新建资源
- PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）
- PATCH(UPDATE)：在服务器更新资源（客户提供改变的属性）
- DELETE(DELETE)：在服务器删除资源



### 创建数据库表

建库语句

```
CREATE DATABASE test
	CHARACTER SET utf8mb4
	COLLATE utf8mb4_general_ci;
```



#### 建表语句

先使用$use test$

```
CREATE TABLE student(
	id INT AUTO_INCREMENT PRIMARY KEY,
	name VARCHAR(50) NOT NULL,
	email VARCHAR(100) NOT NULL,
	age INT
);
```

#### 插入表语句

```
use test
insert into student(name, email, age) values('eric', 'abc@123.com', 20);
```


# 1、spring配置文件

pom.xml

   无法读取xml文件,将<package>修改为war! 
   
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>spring</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    
        <dependencies>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.13.2</version>
                <scope>compile</scope>
            </dependency>
            <dependency>
                <groupId>org.aspectj</groupId>
                <artifactId>aspectjweaver</artifactId>
                <version>1.9.4</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>3.5.2</version>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.21</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-jdbc</artifactId> 
                <version>5.3.7</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
                <version>2.0.6</version>
            </dependency>
        </dependencies>
</project>
```

ApplicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

<!--指定扫描的包，此目录下注解生效-->
<context:component-scan base-package="com.wu"/>
<!--使用配置类时加上去-->
<context:annotation-config/>

</beans>
       
```



# 2、IOC注入

## 2.1、构造器注入

​	——必须在xml中配置所有属性

## 2.2、set方法注入

​	——不必须配置所有属性，但必须存在无参构造函数。若创建了有参构造函数，默认的无参构造函数会消失，必须重新手动添加

```xml
<!-- 构造器创建  -->
<bean id="monkey" class="com.wu.Dao.TigerDaoImpl">
        <constructor-arg name="age" value="11"></constructor-arg>
        <constructor-arg name="name" value="小王"></constructor-arg>
    </bean>

<!-- set方法创建  -->
<bean id="tiger" class="com.wu.Dao.TigerDaoImpl">
            <property name="age" value="11"></property>
            <property name="name" value="katty"></property>
 </bean>
```

## 2.3、bean作用域

1、单例模式（默认）

2、原型模式

​		——每次get获取新对象

```xml
<bean id="tiger" class="com.wu.Dao.TigerDaoImpl" scope="singleton"/>
<bean id="tiger" class="com.wu.Dao.TigerDaoImpl" scope="prototype"/>
```

# 3、bean自动装配
## 3.1 注解

@Autowired 加在属性或set()方法上，默认按照byType自动装配
        
        ——若不能唯一自动装配，补充@Qualifier根据name指定xml中的某个bean
   
@Resource 先根据byName，后根据byType自动装配

@Nullable 此字段可为空，不报错

@Component 此类被spring接管
    @Controller、@Service、@Repository
    
@Scope("prototype") 在类上设置单例模式OR原型模式，默认单例

```Java
@Autowired
@Qualifier(value="cat1")
private Cat cat;

@Resouce(name= "dog")
private Dog dog;

@Component
@Scope("prototype")
public class Tiger
{
    @Value("katty")
    private String name;
}
```
## 3.2、Java配置类

@Configuration表示此类为配置类，能通过AnnotationConfigApplicationContext获取容器

```java
@Configuration
public class TigerConfig
{
    //方法名为xml中的id
    //返回值类型为xml中的class
    @Bean
    public Animals tiger()
    {
        return new Tiger();
    }
    
    ApplicationContext context = new AnnotationConfigApplicationContext(TigerConfig.class);
}
```
# 4、动态代理-反射

动态代理 InvocationHandle的实现类
 ```java
@Component
public class ProxyInvocationHandle implements InvocationHandler {
    private Object target;

    public void setTarget(Object target) {
        this.target = target;
    }

    //得到代理类
    public Object getProxy()
    {
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }

    //处理代理实例，返回结果
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //此处增加代理方法
        return (Object) method.invoke(target, args);
        
    }
}
```
测试调用类
```java

public void test()
{
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    //获取需要代理的对象(真实对象)
    Rent houseowner = (Rent) context.getBean("houseowner");
    //获取动态代理工具对象
    ProxyInvocationHandle proxyInvocationHandle = (ProxyInvocationHandle) context.getBean("proxyInvocationHandle");

    //设置要代理的对象
    proxyInvocationHandle.setTarget(houseowner);
    //动态生成代理类
    Rent rentproxy = (Rent) proxyInvocationHandle.getProxy();
    rentproxy.rentHouse();
}
```

# 5、AOP实现

对于内含AOP切入点的bean对象，spring容器初始化时会自动生成一个动态代理对象
因此：使用getBean()获取bean时，获取的是动态代理对象，而不是它本身。

```xml

<bean id="logger" class="com.wu.logs.log"/>
<bean id="houseOwner" class="com.wu.HouseOwner"/>

<!--1、使用Spring原生AOP-->
    <aop:config>
    <!--创建切入点-->
        <aop:pointcut id="pointcut1" expression="execution(* com.wu.HouseOwner.*(..))"/>
        <aop:advisor advice-ref="logger" pointcut-ref="pointcut1"/>
    </aop:config>

<!-- 2、使用自定义类实现AOP-->
<bean id="pointcut2" class="com.wu.logs.pointcut">
    <aop:config>
    <!--定义切面-->
        <aop:aspect ref="pointcut2">
            <!--定义切点-->
            <aop:pointcut id="pointcut" expression="execution(* com.wu.HouseOwner.*(..))"/>
            <aop:after method="after" pointcut-ref="pointcut"/>
        </aop:aspect>
    </aop:config>

<!--3、注解实现AOP-->
    <aop:config>
        <!--开启注解支持-->
        <aop:aspectj-autoproxy/>
    </aop:config>
```

```java
public void test2()
    {
        ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
        Rent houseowner = (Rent) context.getBean("houseowner",Rent.class);
        houseowner.rentHouse();
    }
```

## 5.1、 Spring实现AOP
```java
public class log implements MethodBeforeAdvice //动态代理
{
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println(o.getClass().getName()+"的"+method.getName()+"执行成功");
    }
}
```


## 5.2、 自定义类实现AOP
```java
public class pointcut 
{
    public void after()
    {
        System.out.println("执行后!!!!!!!!!!!!");
    }
}
```

## 5.3、 注解实现AOP

```java
@Aspect
public class pointcut {
    
    @After("execution(* com.wu.HouseOwner.*(..))")
    public void after()
    {
        System.out.println("执行后!!!!!!!!!!!!");
    }
}
```
# 6、声明式事务

ACID 原子性、一致性、隔离性、持久性

# 1、mybatis demo

##注意点：
1、sqlSession线程不安全，不允许放入静态代码块，且使用完成后将sqlSession.close()放入finally中;

2、Mapper.xml中的namespace必须与对应的接口名保持一致

3、Mapper.xml中SQL语句，id对应接口中的方法名，parameterType代表传入参数类型，resultType表示sql执行的返回值

4、增删改SQL必须提交事务：sqlSession.commit()才会生效

5、使用map传递参数，在sql中可以使用map的key;使用对象传递参数,sql中必须匹配对象的属性值。

6、Mapper.xml中sql语句的resultType表示返回集合中的元素类型，而非集合类型;

7、读取不到xml文件：在pom文件中增加<build><build/>,配置resource文件
```xml
<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                    <includes>
                        <include>**/*.properties</include>
                        <include>**/*.xml</include>
                    </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                    <includes>
                        <include>**/*.properties</include>
                        <include>**/*.xml</include>
                    </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```
8、找不到Mapper.xml文件：每一个Mapper.xml都必须在mybatis-config.xml中导入，路径名必须用"/"分割！！！！
```xml
<mappers>
        <mapper resource="com/wu/dao/UserMapper.xml"/>
</mappers>
```
9、数据库URL中配置时区
```xml
<property name="url" value="jdbc:mysql://localhost:3306/test?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>
```

10、实体类设置序列化
    实体类实现Serializable接口，否则报错NotSerializableExcetion

11、拿不到数据库链接
    查看数据库进程是否启动
    查看pom中引入mysql-connector-java版本
##案例
pom文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>




    <groupId>org.example</groupId>
    <artifactId>mybatis</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>compile</scope>
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
    </dependencies>
    <!--在build中配置resources,防止找不到资源-->
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                    <includes>
                        <include>**/*.properties</include>
                        <include>**/*.xml</include>
                    </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                    <includes>
                        <include>**/*.properties</include>
                        <include>**/*.xml</include>
                    </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
</project>
```

 mybatis-config.xml 核心配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">


<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    
    <mappers>
        <mapper resource="com/wu/dao/UserMapper.xml"/>
    </mappers>

</configuration>
```

Mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace对应一个接口-->
<mapper namespace="com.wu.dao.UserDao">
    <select id="getUserList" resultType="com.wu.dao.User">
        select * from user
    </select>

    <select id="getUserById" parameterType="int" resultType="com.wu.dao.User">
        select * from user where id = #{id}
    </select>

    <insert id="addUser" parameterType="com.wu.dao.User" >
        insert into user (name,age,adress) values (#{name},#{age},#{adress})
    </insert>

</mapper>
```

mybatis标准工具类
```java
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static{
        try {
            //第一步，获得sqlSessionFactory对象
            String resouse = "/mybatis-config.xml";
            InputStream inputStream = Resources.class.getResourceAsStream(resouse);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //第二步，通过sqlSessionFactory获取sqlSession
    public static SqlSession getSqlSession()
    {
        return  sqlSessionFactory.openSession();
    }
}
```

```java
//具体类
public class User {
    }
//接口
public interface UserDao {
    List<User> getUserList();
    List<User> getUserById(int id);
    int addUser(User user);
}
```
测试类
```java
public class UserDaoTest {
    @Test
    public void test()
    {
        SqlSession sqlSession =  MybatisUtils.getSqlSession();
        try {
            UserDao userDao = sqlSession.getMapper(UserDao.class);
            List<User> list = userDao.getUserList();
            for (User user : list) {
                System.out.println(user);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //sqlSession使用完成必须关闭
            sqlSession.close();
        }
    }
}
```
# 2、配置解析

1、transactionManager事务管理,type= JDBC | MANAGED;默认JDBC

2、dataSource数据源,type= POOLED | UNPOOLED | JNDI;默认PLLOED

3、properties,在mybatis核心配置文件中引入外部配置文件,字段相同优先使用外部配置文件;ex:db.properties

4、typeAliases别名：给全路径实体类指定别名;指定实体类的包,自动扫描包下的实体类

5、映射器:使用resources文件路径引入,必须用"/"分割
```xml
    <mappers>
            <mapper resource="com/wu/dao/UserMapper.xml"/>
    </mappers>
```
6、生命周期作用域
    sqlSessionFatoryBuilder 创建sqlSessionFactory后失去作用,应作为局部变量
    sqlSessionFactory 应用运行期间一直存在,使用单例模式,应作为全局变量
    sqlSession 线程不安全,用完后须关闭,作为请求或方法内变量
    
 # 3、多对一&一对多处理
 
 方法一:子查询嵌套
 
 ```xml
<!--方法一:子查询-->
<select id="getStudents" resultMap="StudentTeacher1" >
    select * from student
</select>

<resultMap id="StudentTeacher1" type="Student">
    <result property="id" column="id"/>
    <result property="name" column="name"/>
    <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
</resultMap>
<select id="getTeacher" resultType="Teacher">
    select * from teacher where id = #{id}
</select>
```
 方法二:关联查询嵌套
 
 ```xml
<!--方法二:关联查询-->
    <select id="getStudent" resultMap="StudentTeacher2">
        select s.id , s.name , t.name as tname from student s ,teacher t where s.tid = t.id;
    </select>

    <resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <association property="teacher" javaType="Teacher">
            <result property="name" column="tname"/>
        </association>
    </resultMap>
```


关联- association [多对一]
集合- collection [一对多]

#4、动态SQL

# 5、缓存

## 一级缓存 

sqlsession级别
默认缓存类型,仅在一次sqlsession中生效
    涉及增删改操作时(涉及commit操作),清理所有缓存
    
    

## 二级缓存 全局缓存

namespace级别，同一个Mapper下有效
所有缓存存入一级缓存中，只有sqlSession 关闭或者事务提交时转存入一级缓存
<cache/>


用户读取顺序：
    先看二级缓存（全局缓存），再查询一级缓存，最后SQL查询数据库，再将结果保存至一级缓存中

    
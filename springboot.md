# 1、自动配置

默认包结构:

​	主程序所在包及其子目录下所有组件会被默认添加

改变扫描路径:@SpringBootApplication(scanBasepackages="xxx")



# 2、容器功能

## 2.1、组件添加

1、配置类中使用@Bean标注在方法上，给容器注册组件

2、配置类本身也是组件

3、proxyBeanMethod  代理Bean的方法

​	Full—@Configuration(proxyBeanMethod = true) 	@Bean方法返回对象保证单例模式

​	Lite—@@Configuration(proxyBeanMethod = false) 	@Bean 方法每次返回不同对象

组件之间存在依赖关系使用Full模式，否则使用Lite模式

4、@Import({xxx.class,xxx.calss})

​		标注在配置类中，导入对应类型的组件，组件名称为全类名

5、@Conditional 标注在配置类或@Bean方法上，满足某些条件才加载组件

​		因为组件是按顺序加载，所以加注解的方法需要在判断条件组件后面

​		由此实现用户未配置情况下默认配置的加载

##  2.2、xml导入

@ImportResource 

导入旧版xml文件中的组件

## 2.3、配置绑定

@ConfigurationProperties + @Component 标注在实体类中，实体类中的属性自动赋配置文件application.properties中的值

前提必须把这个类纳入到spring中



# 3、自动配置

spring boot会预先加载所有自动配置类：xxxAutoconfiguration

每个自动配置类按条件生效，默认绑定配置文件的指定值。

​		—默认读取xxxProperties中的配置，xxxProperties绑定application.properties



xxx.properties优先级高于yml

双引号将\n作为换行输出,单引号将\n作为字符串输出



## 3.1、 加载自动配置类

@EnableAutoConfiguration

```java
@AutoConfigurationPackage
@Import({org.springframework.boot.autoconfigure.AutoConfigurationImportSelector.class})
```



@AutoConfigurationPackage

main程序所在的包下所有组件批量导入



## 3.2、按需加载自动配置

使用@Conditional 按照条件规则，按需加载默认配置

## 3.3 、修改默认设置

```java
		@Bean
		@ConditionalOnBean(MultipartResolver.class)
		@ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME)
		public MultipartResolver multipartResolver(MultipartResolver resolver) {
			// Detect if the user has created a MultipartResolver but named it incorrectly
			return resolver;
		}
```

对于标注了@Bean的方法传入参数，这个参数会根据类型从容器中寻找

## 3.4、最佳实践

1、引入场景依赖

2、查看哪些组件已自动配置	

​		配置文件中开启debug模式  debug = true，查看打印日志

3、是否需要修改默认配置

​	定制配置：

​	1、@Bean直接替换底层组件

​	2、直接在application.properties中重新写对应的配置值

# 4、web开发

## 4.1、静态资源

静态资源在以下路径：/static、/public 、/resources、 /META-INF/resources下，可以通过/+资源名直接访问

请求优先由controller处理，无法处理将请求转发给静态资源



## 4.2、参数处理

·HanderMapping中找到能处理请求的Hander   Controller.method()

·为当前Hander找到适配器 HanderAdapter 



对于 Map Model  中加数据,都可以加到request域中


















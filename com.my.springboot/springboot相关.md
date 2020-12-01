
#springboot问题
```
    核心的配置文件包括：application 用于项目的自动化配置
                    bootstrap 使用spring cloud config配置中心时，需要在bootstrap配置文件中添加连接到配置中心的配置属性来加载外部配置中心信息--》一些固定的不能被覆盖的属性 一些加密/解密场景
    配置.yml 与.properties区别主要是书写格式不同  .yml不支持@propertySource注解导入配置
    @SpringBootApplication核心注解：
                                 @SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。
                                 @EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。
                                 @ComponentScan：Spring组件扫描。
    开启springboot特性的方式：
                          继承spring-boot-starter-parent项目
                          导入spring-boot-dependencies项目依赖
    springboot自动配置原理：
                        注解 @EnableAutoConfiguration, @Configuration, @ConditionalOnClass 就是自动配置的核心，首先它得是一个配置文件，其次根据类路径下是否有这个类去自动配置。
    starter怎么理解：
                    Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成 Spring 及其他技术，而不需要到处找示例代码和依赖包。如你想使用 Spring JPA 访问数据库，只要加入 spring-boot-starter-data-jpa 启动器依赖就能使用了。
                    Starters包含了许多项目中需要用到的依赖，它们能快速持续的运行，都是一系列得到支持的管理传递性依赖。
    在springboot启动的时候运行一些特定代码：
                                      实现接口 ApplicationRunner 或者 CommandLineRunner，这两个接口实现方式一样，它们都只提供了一个 run 方法
    读取配置的方式：
                 可以通过 @PropertySource,@Value,@Environment, @ConfigurationProperties 来绑定变量
    支持的日志框架：
                支持 Java Util Logging, Log4j2, Lockback 作为日志框架，如果你使用 Starters 启动器，Spring Boot 将使用 Logback 作为默认日志框架
    实现热部署的方式：
                   spring loaded
                   spring-boot-devtools
    配置文件的加载顺序：
                    properties文件
                    yml文件
                    系统环境变量
                    命令行参数
    兼容老项目：
              可以兼容，使用@ImportResource注解导入老spring项目配置文件
    保护springboot应用的方法：
                            在生产中使用HTTPS
                            使用Snyk检查你的依赖关系
                            升级到最新版本
                            启用CSRF保护
                            使用内容安全策略防止XSS攻击
```

#springboot启动
   - SpringApplicaiton初始化
    审查ApplicationContext类型
    加载ApplicationContextInitializer
    加载ApplicationListener
  - Environment初始化
    解析命令行参数
    创建Environment
    配置Environment
    配置SpringApplication
  - ApplicationContext初始化
    创建ApplicationContext
    设置ApplicationContext
    刷新ApplicationContext
  - 运行程序入口

##SpringApplicaiton初始化
```
@SpringBootApplication
@Slf4j
public class SpringEnvApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringEnvApplication.class, args);
     }
}

SpringApplication的构造函数干了些啥：
    基础变量赋值（resourceLoader、primarySources、...）
    审查ApplicationContext类型如（Web、Reactive、Standard)
    加载ApplicationContextInitializer
    加载ApplicationListener
    审查启动类（main方法的类）

SpringBoot会在初始化阶段审查ApplicationContext的类型，审查方式是通过枚举WebApplicationType的deduceFromClasspath静态方法：
WebApplicationType枚举用于标记程序是否为Web程序，它有三个值：
    NONE：不是web程序
    SERVLET：基于Servlet的Web程序
    REACTIVE：基于Reactive的Web程序
该方法会通过classpath来判断是否Web程序，方法中的常量是完整的class类名：
    例如通过pom.xml文件引入spring-boot-starter-web那classpath就会有org.springframework.web.context.ConfigurableWebApplicationContext和javax.servlet.Servlet类，这样就决定了程序的ApplicationContext类型为WebApplicationType.SERVLET。

@EnableAutoConfiguration 最关键的是@Import({EnableAUtoConfigurationImportSelector.class}) 注解
@EnableAutoConfiguration的作用可以描述为：从classpath下搜寻所有的META-INF/spring.factories配置文件，并将EnableAutoConfiguration对应的配置项通过反射实例化为对应标注了@Configuration的IoC容器配置类，然后汇总为一个并加载到IoC容器中
SpringFactoriesLoader 该类的作用是加载SpringBoot应用下META-INF/spring.factories配置文件。该配置文件是一个Properties文件。

```

###加载ApplicationContextInitializer
    SpringBoot通过SpringFactoriesLoader加载spring.factories中的配置读取key为org.springframework.context.ApplicationContextInitializer的value，前面提到过spring.factoies中的配置的value都为key的实现类：
    spring-boot-2.2.0.RELEASE.jar中包含的配置，其他jar包也有可能配置org.springframework.context.ApplicationContextInitializer来实现额外的初始化工作
    
###加载ApplicationListener
    ApplicationListener用于监听ApplicationEvent事件，它的初始加载流程跟加载ApplicationContextInitializer类似，在spring.factories中也会配置一些优先级较高的ApplicationListener：
    ApplicationListener的加载流程跟ApplicationContextInitializer类似都是通过SpringFactoriesLoader加载的。
    
##Environment初始化
###解析命令行参数
    命令行参数是由main方法的args参数传递进来的，SpringBoot在准备阶段建立一个DefaultApplicationArguments类用来解析、保存命令行参数。如--spring.profiles.active=dev就会将SpringBoot的spring.profiles.active属性设置为dev。
    SpringBoot还会将收到的命令行参数放入到Environment中，提供统一的属性抽象。
###创建Environment
    创建环境的代码比较简单，根据之前提到过的WebApplicationType来实例化不同的环境：

###准备Environment
```
环境（Environment）大致由Profile和PropertyResolver组成：
    Profile是BeanDefinition的逻辑分组，定义Bean时可以指定Profile使SpringBoot在运行时会根据Bean的Profile决定是否注册Bean
    PropertyResolver是专门用来解析属性的，SpringBoot会在启动时加载配置文件、系统变量等属性
SpringBoot在准备环境时会调用SpringApplication的prepareEnvironment方法：
    prepareEnvironment方法大致完成以下工作：
        创建一个环境
        配置环境
        设置SpringApplication的属性   
```
###配置Environment
    配置环境主要用于：
        设置ConversionService： 用于属性转换
        将命令行参数添加到环境中
        添加额外的ActiveProfiles
###SpringApplicaton属性设置
    配置SpringApplicaton主要是将已有的属性连接到SpringApplicaton实例，如spring.main.banner-mode属性就对应于bannerMode实例属性，这一步的属性来源有三种（没有自定义的情况）：
        环境变量
        命令行参数
        JVM系统属性
    SpringBoot会将前缀为spring.main的属性绑定到SpringApplicaton实例：

##ApplicationContext初始化
###创建ApplicationContext
    创建ApplicationContext的过程与创建环境基本模相似，根据WebApplicationType判断程序类型创建不同的ApplicationContext：
    前面提到过WebApplicationType有三个成员（SERVLET，REACTIVE，NONE），分别对应不同的context类型为:
###准备ApplicationContext
```
创建完ApplicationContext完后需要初始化下它，设置环境、应用ApplicationContextInitializer、注册Source类等，SpringBoot的准备Context的流程可以归纳如下：
    为ApplicationContext设置环境（之前创建的环境）
    基础设置操作设置BeanNameGenerator、ResourceLoader、ConversionService等
    执行ApplicationContextInitializer的initialize方法（ApplicationContextInitializer是在初始化阶段获取的）
    注册命令行参数（springApplicationArguments）
    注册Banner（springBootBanner）
    注册sources（由@Configuration注解的类）    
注意注册sources这一步，sources是@Configuration注解的类SpringBoot根据提供的sources注册Bean，基本原理是通过解析注解元数据，然后创建BeanDefinition然后将它注册进ApplicationContext里面

```
###刷新ApplicationContext

    刷新ApplicationContext基本步骤：
        准备刷新（验证属性、设置监听器）
        初始化BeanFactory
        执行BeanFactoryPostProcessor
        注册BeanPostProcessor
        初始化MessageSource
        初始化事件广播
        注册ApplicationListener
        ...
###运行程序入口
    context刷新完成后Spring容器可以完全使用了，接下来SpringBoot会执行ApplicationRunner和CommandLineRunner，这两接口功能相似都只有一个run方法只是接收的参数不同而以。通过实现它们可以自定义启动模块，如启动dubbo、gRPC等。
    callRunners执行完后，SpringBoot的启动流程就完成了。

##spring.factories需要知道
```
spring.factories是一个properties文件
spring.factories里的键值对的value是以逗号分隔的完整类名列表
spring.factories里的键值对的key是完整接口名称
spring.factories键值对的value是key的实现类
spring.factories是由SpringFactoriesLoader工具类加载
spring.factories位于classpath:/META-INF/目录
SpringFactoriesLoader会加载jar包里面的spring.factories文件并进行合并
```
###配置变量的引用
 @profiles.active@

##springboot
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

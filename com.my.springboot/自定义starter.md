#自定义starter
    二级目录结构：
    .
    ├── pom.xml
    ├── rgyb-spring-boot-autoconfigure
    │   ├── pom.xml
    │   └── src
    ├── rgyb-spring-boot-sample
    │   ├── pom.xml
    │   └── src
    └── rgyb-spring-boot-starter
        ├── pom.xml
        └── src
##Auto-Configure Module
    Auto-Configure Module (自动配置模块) 是包含自动配置类的 Maven 或 Gradle 模块。通过这种方式，我们可以构建可以自动贡献于应用程序上下文的模块，以及添加某个特性或提供对某个外部库的访问

###创建一个空的父亲 Maven Module，主要提供依赖管理，这样 SubModule 不用单独维护依赖版本号，来看 pom.xml 内容:
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
        <!--  添加其他全局依赖管理到这里，submodule默认不引入这些依赖，需要显式的指定  -->
    </dependencyManagement>
###新建类 GreetingAutoConfiguration
    1.我们用 @Configuration 注解标记类 GreetingAutoConfiguration，作为 starter 的入口点。这个配置包含了我们需要提供starter特性的所有 @Bean 定义
        @Configuration
        public class GreetingAutoConfiguration {
            @Bean
            public GreetingService greetingService(GreetingProperties greetingProperties){
                return new GreetingService(greetingProperties.getMembers());
            }
        }
    2.在 resources 目录下新建文件 META-INF/spring.factories (如果目录 META-INF 不存在需要手工创建)，向文件写入内容:    
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
     top.dayarch.autoconfigure.GreetingAutoConfiguration    
     Spring 启动时会在其 classpath 中所有的 spring.factoreis 文件，并加载里面的声明配置，GreetingAutoConfiguration 类就绪后，我们的 Spring Boot Starter 就有了一个自动激活的入口点。到这里这个 "不完全的 starter" 已经可以使用了。但因为它是自动激活的，为了个让其灵活可用，我们需要让其按照我们的意愿来激活使用，所以我们需要条件注解来帮忙   
###条件配置
       @Configuration
       @ConditionalOnProperty(value = "rgyb.greeting.enable", havingValue = "true")
       @ConditionalOnClass(DummyEmail.class)
       public class GreetingAutoConfiguration {
           ...
       } 
    通过使用 @ConditionalOnProperty 注解，我们告诉 Spring，只有属性 rgyb.greeting.enable 值被设置为 true 时，才将 GreetingAutoConfiguration (以及它声明的所有 bean ) 包含到应用程序上下文中
    通过使用 @ConditionalOnClass 注解，我们告诉Spring 只有类 DummyEmail.class 存在于 classpath 时，才将 GreetingAutoConfiguration (以及它声明的所有 bean ) 包含到应用程序上下文中
    多个条件是 and/与的关系，既只有满足全部条件时，才会加载 GreetingAutoConfiguration
###配置属性管理
    上面使用了 @ConditionalOnProperty 注解，实际 starter 中可能有非常多的属性，所以我们需要将这些属性集中管理:
    
    @Data
    @ConfigurationProperties(prefix = "rgyb.greeting")
    public class GreetingProperties {
    
        /**
         * GreetingProperties 开关
         */
        boolean enable = false;
    
        /**
         * 需要打招呼的成员列表
         */
        List<String> members = new ArrayList<>();
    }
    
    我们知道这些属性是要在 application.yml 中使用的，当我们需要使用这些属性时，为了让 IDE 给出更友好的提示，我们需要在 pom.xml 中添加依赖:
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    这样当我们 mvn compile 时，会在生成一个名为 spring-configuration-metadata.json JSON 文件，文件内容如下:
###提升启动时间
    对于类路径上的每个自动配置类，Spring Boot 必须计算 @Conditional… 条件值，用于决定是否加载自动配置及其所需的所有类，根据 Spring 启动应用程序中 starter 的大小和数量，这可能是一个非常昂贵的操作，并且会影响启动时间，为了提升启动时间，我们需要在 pom.xml 中添加另外一个依赖:
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure-processor</artifactId>
        <optional>true</optional>
    </dependency>
    这个注解会生成一个名为 spring-autoconfigure-metadata.properties Property 文件

##Starter Module
    Spring Boot Starter 是一个 Maven 或 Gradle 模块，其唯一目的是提供 "启动" 某个特性所需的所有依赖项。可以包含一个或多个 Auto-Configure Module (自动配置模块)的依赖项，以及可能需要的任何其他依赖项。这样，在Spring 启动应用程序中，我们只需要添加这个 starter 依赖就可以使用其特性
    Starter Module 的构建很简单了，你可以认为它就是一个空 module，除了依赖 Auto-Configure Module，其唯一作用就是为了使用 starter 功能特性提供所有必须依赖，所以我们为 starter module 的 pom.xml 文件添加如下内容:
    <dependencies>
        <dependency>
            <groupId>top.dayarch.learnings</groupId>
            <artifactId>rgyb-spring-boot-autoconfigure</artifactId>
            <version>1.0.0.RELEASE</version>
        </dependency>
        <!-- 在此处添加其他必要依赖，保证starter可用 -->
    </dependencies>
    
##创建 Sample Module测试
    通过 Spring Initializr 正常初始化一个 Spring Boot 项目 (rgyb-spring-boot-sample)，引入我们刚刚创建的 starter 依赖，在 sample pom.xml 中添加依赖:
    <dependency>
        <groupId>top.dayarch.learnings</groupId>
        <artifactId>rgyb-spring-boot-starter</artifactId>
        <version>1.0.0.RELEASE</version>
    </dependency>
    
    接下来配置 application.yml 属性
    rgyb:
      greeting:
        enable: true
        members:
          - 李雷
          - 韩梅梅 
    
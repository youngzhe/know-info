1.面向切面编程的优点及解决的问题
##切入点，
    通知   
    通过切入点筛选出想切入的地方，再通过通知定义要织入的方法【是织入before还是织入after，然后是方法的内容】
##注解
    @AspectJ 
                  @Aspect   【类名上的注解如结合@Component】
                  @Pointcut 切入点【定义匹配方式】
                  @Advice  通知【如@Before  @After】
##切入点表达式
            designators【指示器----》通过什么方式匹配你想拦截的方法包筛选出来】
                                  匹配方法   execution()
                                              execution(
                                                      修饰符
                                                      返回值*
                                                      包名
                                                      方法名(方法参数)*
                                                      异常
                                                              )    
                                          
                                  匹配注解   @target（）
                                                     @args（）
                                                     @within（）
                                                     @annotation（）方法级别的注解
                                   匹配包/类型 within（）如：@Pointcut("within(com.sithch.service..*)")
                                   匹配对象      this()
                                                      bean()
                                                      target()
                                    匹配参数   args（）【可通过操作符配合使用】
             wildcards  通配符  
                                         * 匹配任意数量字符
                                         +匹配指定类及其子类
                                          .. 一般用于匹配任意数的子包或者参数
               operators 【运算符】
                                          && 与操作符
                                           ||   或操作符
                                           ！非操作符

	1. 
    Advice注解


                      @Before（） 前置通知  MethodBeforeAdvice 
                      @After（） 后置通知，方法执行完之后  AfterReturningAdvice 
                       @AfterReturning（） 返回通知，成功执行之后 可取回返回值【returning=“result”】
                       @AfterThrowing（）  异常通知，抛出异常之后
                       @Around（）环绕通知 【包括了以上四种通知类型】  ThrowsAdvice 

#aop的实现原理
```
   【织入的时机】
   6.1 编译期（aspectj）
   6.2类加载时（aspectj 5++）
   6.3运行时（spring aop）
         6.3.1静态代理 代理模式【代理的方法多时，重复逻辑会很多】
         6.3.2动态代理   代理类 implement invocationhandler
                 基于接口的代理  jdk的代理【只能基于接口动态代理】
                 基于继承的代理  cglib的代理 【 implement methodinterceptor】【无法对static final 类 private 方法代理】
 
如果目标对象实现了接口，则默认采用jdk动态代理
如果目标对象没有实现接口，采用cglib进行动态代理
如果目标实现了接口，且且强制cglib代理则使用cglib代理

设置@SpringBootAplication
       @EnableAspectJAutoProxy(proxyTargetClass=true)强制使用cglib代理

对方法的增强叫做 Weaving（织入）
对类的增强叫做 Introduction（引入），而 Introduction Advice（引入增强）就是对类的功能增强( 某个类实现了 A 接口，但没有实现 B 接口，该类也可以调用 B 接口的方法。【引入增强】)


Ioc：Inversion of Control —— 控制反转

DI：Dependency Injection —— 依赖注入

其实这两个概念本质上是没有区别的，那我们先来看看什么叫做Ioc?

假设这么一个场景：

在A类中调用B类的方法，那么我们就称 A依赖B，B为被依赖（对象），相信这点大家能够理解。

传统做法：

（1）直接在A（方法）中new出B类对象，然后调用B类方法 —— 硬编码耦合；

（2）通过简单工厂获取B类对象，然后调用B类的方法 —— 摆脱了与B的耦合，却又与工厂产生了耦合；

以上两种做法，都是在A中主动去new或调用简单工厂的方法产生B的对象，注意，关键字是“主动”
```

1、用一句自己的话总结学过的设计模式（要精准）。

| 设计模式                | 一句话归纳                                      | 举例                                                         |
| ----------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| 工厂模式（Factory）     | 只对结果负责，封装创建过程                      | BeanFactory、Calender                                        |
| 单例模式（Singleton）   | 保证独一无二                                    | ApplicationContext、Calender                                 |
| 原型模式（Prototype）   | 拔一根猴毛，吹出千万个                          | ArrayList、PrototypeBean                                     |
| 代理模式（Proxy）       | 找人办事，增强职责                              | ProxyFactoryBean、 JdkDynamicAopProxy、CglibAopProxy         |
| 委派模式（Delegate）    | 干活算你的（普通员工），功 劳算我的（项目经理） | DispatcherServlet、 BeanDefinitionParserDelegate             |
| 策略模式（Strategy）    | 用户选择，结果统一                              | InstantiationStrategy                                        |
| 模板模式（Template）    | 流程标准化，自己实现定制                        | JdbcTemplate、HttpServlet                                    |
| 适配器模式（Adapter）   | 兼容转换头                                      | AdvisorAdapter、HandlerAdapter                               |
| 装饰器模式（Decorator） | 包装，同宗同源                                  | BufferedReader、InputStream、  OutputStream、 HttpHeadResponseDecorator |
| 观察者模式（Observer）  | 任务完成时通知                                  | ContextLoaderListener                                        |

2、列举SpringAOP、IOC、DI应用的代码片段。

Aop片段

```java
@Component
@Aspect
@Slf4j
public class Aop{
    @Pointcut("execution(* com.vip.pattern.service..*(..))")
    public void aspect(){}
    @Before("aspect()")
    public void before(JoinPoint joinPoint){
        log.info("前置通知"+joinPoint);
    }
    @After("aspect()")
    public void after(JoinPoint joinPoint){
        log.info("后置通知"+joinPoint);
    }
    @Around("aspect()")
    public void around(JoinPoint joinPoint){
        joinPoint.proceed();
        log.info("循环通知"+joinPoint);
    }
    @AfterReturning("aspect()")
    public void afterReturn(JoinPoint joinPoint){
        log.info("后置返回通知"+joinPoint);
    }
    @AfterThrowing(pointcut="aspect()",throwing="ex")
    public void afterThrow(JoinPoint joinPoint,Exception ex){
        log.info("后置异常通知"+joinPoint+ex.getMessage());
    }
}
```

IOC片段

```java
@Component
public class Service{
    .....
}
```

DI片段

```java
public class Service{
    @Autowired
    public ServiceA service；
    .....
}
```


# 1、使用ProcessOn绘制Spring DI容器初始化过程的时序图。

# ![1554276790093](images/一步一步手绘Spring+DI运行时序图.jpg)2、整理笔记，完全理解Spring DI容器的核心原理和设计模式的应用背景。

核心原理：

DI(DependencyInjection)依赖注入：就是指对象是被动接受依赖类而不是自己主动去找，换句话说就
是指对象不是从容器中查找它依赖的类，而是在容器实例化对象的时候主动将它依赖的类注入给它

应用背景：

采用了简单工厂和工厂方法生成java实例对象，还用了策略模式是创建单例还是多例 或者是代理对象，对实例对象进行包装

# 3、加强理解练习，掌握看源码的要领；看源码从此不晕车。

BeanFactory
AbstractBeanFactory#getBean()
    创建BeanWrapper
        缓存在FactoryBeanRegistrySupport的Map<String , Object> factoryBeanObjectCache中。
    保存到AbstractAutowireCapableBeanFactory的Map<String , BeanWrapper> factoryBeanInstanceCache中。
BeanWrapperImpl#setValue()
循环赋值？
    Set<Class> autowired;
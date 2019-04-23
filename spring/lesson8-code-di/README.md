1、自己手写完成高仿真版本的Spring DI部分。

```java
public class MyApplicationContext extends MyDefaultListaeBeanFactory implements MyBeanFactory{
    private String[] configLocations;
    private MyBeanDefinitionReader reader;
    private Map<String,Object> singletonBeanCacheMap = new HashMap<String,Object>();
    private Map<String,MyBeanWrapper> beanWrapperMap = new ConcurrentHashMap<String,MyBeanWrapper>();
    public Object getBean(String beanName){
        MyBeanDefinition beanDefinition = this.beanDefinitionMap.get(beanName);
        try{
            MyBeanPostProcessor beanPostProcessor = new MyBeanPostProcessor();
            Object instance = instantiateBean(beanDefinition);
            if(null==instance) return null;
            beanPostProcessor.postProcessBeforeInitialization(instance,beanName);
            MyBeanWrapper beanWrapper = new MyBeanWrapper(instance);
            this.beanWrapperMap.put(beanName,beanWrapper);
            beanPostProcessor.postProcessAfterInitialization(instance,beanName);
            populateBean(beanName,instance);
            return this.beanWrapperMap.get(beanName).getWrappedInstance();
        }catch (Exception e){
            return null;
        }
    }
    private void populateBean(String beanName,Object instance){
        Class clazz = instance.getClass();
        if(!(clazz.isAnnotationPresent(MyController.class)||
            clazz.isAnnotationPresent(MyService.class))) return;
        Field[] fields = clazz.getDeclaredFields();
        for(Field field:fields){
            if(!field.isAnnotationPresent(MyAutowired.class)) continue;
            MyAutowired autowired = field.getAnnotation(MyAutowired.class);
            String autowiredBeanName = autowired.value().trim();
            if("".equals(autowiredBeanName)) autowiredBeanName = field.getType().getName();
            field.setAccessible(true);
            try{
                field.set(instance,this.beanWrapperMap.get(autowiredBeanName).getWrappedInstance());
            }catch(IllegalAccessException e){
                
            }
        }
    }
    private Object instantiateBean(MyBeanDefinition beanDefinition){
        Object instance = null;
        String className  beanDefinition.getBeanClassName();
        try{
            if(this.singletonBeanCacheMap.containsKey(className)){
                instance = this.singletonBeanCacheMap.get(className);
            }else{
                Class<?> clazz = Class.forName(className);
                instance = clazz.newInstance();
                this.singletonBeanCacheMap.put(beanDefinition.getFactoryBeanName(),instance);
            }
            return instance;
        }catch(Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
public class MyBeanPostProcessor{
    public Object postProcessBeforeInitialization(Object bean,String beanName) throws Exception{
        return bean;
    }
    public Object postProcessAfterInitialization(Object bean,String beanName) throws Exception{
        return bean;
    }
}
```


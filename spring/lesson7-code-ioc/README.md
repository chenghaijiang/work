1、自己手写完成高仿真版本的Spring IOC部分。

先设计一个顶层的IOC容器MyAbstractApplicationContext

```java
public abstract class MyAbstractApplicationContext {
    public void refresh() throw Exception{}
}
public interface MyBeanFactory {
    Object getBean(String beanName);
}
```

再弄个默认的ioc容器实现类MyDefaultListableBeanFactory

```java
public class MyDefaultListableBeanFactory extends MyAbstractApplicationContext {
    protected final Map<String,MyBeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>();
}
```

存储bean元信息的类MyBeanDefinition

```java
@Data
public class MyBeanDefinition {
    String beanClassName;
    boolean lasyInit = false;
    String factoryBeanName;
}
```

bean 的加载器类MyBeanDefinitionReader

```java
public class MyBeanDefinitionReader {
    private List<String> registryBeanClasses = new ArrayList<>();
    private Properties config = new Properties();
    private final String SCAN_PACKAGE = "scanPackage";
    public MyBeanDefinitionReader(String... locations){
        InputStream in = this.getClass().getClassLoader().getResourceAsStream(location[0].replace("classpath:",""));
        try{
            config.load(in);
        }catch (Exception e){
            e.printStackTrace();
        }finally{
            if(null!=in){
                try{
                    in.close();
                }catch (IOException e){
                    e.printStackTrace();
                }
            }
        }
        doScanner(config.getProperty(SCAN_PACKAGE));
    }
    private void doScanner(String scanPackage){
        URL url = this.getClass().getClassLoader().getResource("/"+scanPackage.replaceAll("\\.","/"));
        File classPath = new File(url.getFile());
        for(File file:classPath.listFiles()){
            if(file.isDirectory()){
                doScanner(scanPackage+"."+file.getName());
            }else{
                if(!file.getName().endsWith(".class")){continue;}
                String className = scanPackage + "." + file.getName().replace(".class","");
                registryBeanClasses.add(className);
            }
        }
    }
    public Properties getConfig(){
        return this.config;
    }
    public List<MyBeanDefinition> loadBeanDefinitions(){
        List<MyBeanDefinition> result = new ArrayList<>();
        try{
            for(String className:registryBeanClasses){
                Class<?> beanClass = Class.forName(className);
                if(beanClass.isInterface()) continue;
                result.add(doCreateBeanDefinition(toLowerFirstCase(beanClass.getSimpleName()),beanClass.getName()));
                Class<?> [] interfaces = beanClass.getInterfaces();
                for(Class<?> i:interfaces){
                    result.add(doCreateBeanDefinition(i.getName,beanClass.getName()));
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }
        return result;
    }
    private MyBeanDefinition doCreateBeanDefinition(String factoryBeanName,String beanClassName){
        MyBeanDefinition beanDefinition = new MyBeanDefinition();
        beanDefinition.setBeanClassName(beanClassName);
        beanDefinition.setFactoryBeanName(factoryBeanName);
        return beanDefinition;
    }
    private String toLowerFirstCase(String simpleName){
        char[] chars = simpleName.toCharArray();
        chars[0]+=32;
        return String.valueOf(chars);
    }
 }
```

容器实现类MyApplicationContext

```java
public class MyApplicationContext extends MyDefaultListableBeanFactory implements MyBeanFactory{
    private String[] configLocations;
    private MyBeanDefinitionReader reader;
    public MyApplicationContext(String... configLocations) {
        this.configLocations = configLocations;
        try{
            refresh();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    @Override
    public void refresh() throw Exception {
        reader = new MyBeanDefinitionReader(this.configLocations);
        List<MyBeanDefinition> beanDefinitions = reader.loadBeanDefinitions();
        doRegisterBeanDefinition(beanDefinitions);
        doAutowired();
    }
    public void doAutowired(){
        for(Map.Entry<String,MyBeanDefinition> beanDefinition:super.beanDefinitionMap.entrySet()) {
            String beanName = beandefinition.getKey();
            if(!beanDefinition.getValue().isLasyInit()) {
                getBean(beanName);
            }
        }
    }
    private void doRegisterBeanDefinition(List<MyBeanDefinition> beanDefinitions) throws Exception {
        for(MyBeanDefinition beanDefinition:beanDefinitions) {
            if(super.beanDefinitionMap.containsKey(beanDefinition.getFactoryBeanName())) {
                throw new Exception("The" + beanDefinition.getFactoryBeanName() + "is exists!");
            }
            super.beanDefinitionMap.put(beanDefinition.getFactoryBeanName(),beanDefinition);
        }
    }
    public Object getBean(String beanName) {
        return null;
    }
}
```

2、回顾各设计模式的特点和区别，以及在手写源码的应用。

采用模板方法，比如ioc抽象的顶层类AbstractApplicaitonContext中的refresh方法

采用注册式单例，比如DefaultListableBeanFactory中的伪ioc容器beanDefinitionMap


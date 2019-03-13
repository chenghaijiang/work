1、仿JDK动态代理实现原理，自己手写一遍。

```java
public interface MyInvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable;
}
public class MyProxy {

    public static final String ln = "\r\n";

    public static Object newProxyInstance(MyClassLoader classLoader, Class<?> [] interfaces, MyInvocationHandler h){
       try {
           //1、动态生成源代码.java文件
           String src = generateSrc(interfaces);

//           System.out.println(src);
           //2、Java文件输出磁盘
           String filePath = MyProxy.class.getResource("").getPath();
//           System.out.println(filePath);
           File f = new File(filePath + "$Proxy0.java");
           FileWriter fw = new FileWriter(f);
           fw.write(src);
           fw.flush();
           fw.close();

           //3、把生成的.java文件编译成.class文件
           JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
           StandardJavaFileManager manage = compiler.getStandardFileManager(null,null,null);
           Iterable iterable = manage.getJavaFileObjects(f);

          JavaCompiler.CompilationTask task = compiler.getTask(null,manage,null,null,null,iterable);
          task.call();
          manage.close();

           //4、编译生成的.class文件加载到JVM中来
          Class proxyClass =  classLoader.findClass("$Proxy0");
          Constructor c = proxyClass.getConstructor(MyInvocationHandler.class);
          f.delete();

           //5、返回字节码重组以后的新的代理对象
           return c.newInstance(h);
       }catch (Exception e){
           e.printStackTrace();
       }
        return null;
    }

    private static String generateSrc(Class<?>[] interfaces){
            StringBuffer sb = new StringBuffer();
            sb.append("package com.gupaoedu.vip.pattern.proxy.dynamicproxy.yproxy;" + ln);
            sb.append("import com.gupaoedu.vip.pattern.proxy.Person;" + ln);
            sb.append("import java.lang.reflect.*;" + ln);
            sb.append("public class $Proxy0 implements " + interfaces[0].getName() + "{" + ln);
                sb.append("MyInvocationHandler h;" + ln);
                sb.append("public $Proxy0(MyInvocationHandler h) { " + ln);
                    sb.append("this.h = h;");
                sb.append("}" + ln);
                for (Method m : interfaces[0].getMethods()){
                    Class<?>[] params = m.getParameterTypes();

                    StringBuffer paramNames = new StringBuffer();
                    StringBuffer paramValues = new StringBuffer();
                    StringBuffer paramClasses = new StringBuffer();

                    for (int i = 0; i < params.length; i++) {
                        Class clazz = params[i];
                        String type = clazz.getName();
                        String paramName = toLowerFirstCase(clazz.getSimpleName());
                        paramNames.append(type + " " +  paramName);
                        paramValues.append(paramName);
                        paramClasses.append(clazz.getName() + ".class");
                        if(i > 0 && i < params.length-1){
                            paramNames.append(",");
                            paramClasses.append(",");
                            paramValues.append(",");
                        }
                    }

                    sb.append("public " + m.getReturnType().getName() + " " + m.getName() + "(" + paramNames.toString() + ") {" + ln);
                        sb.append("try{" + ln);
                            sb.append("Method m = " + interfaces[0].getName() + ".class.getMethod(\"" + m.getName() + "\",new Class[]{" + paramClasses.toString() + "});" + ln);
                            sb.append((hasReturn(m.getReturnType()) ? "return " : "") + getCaseCode("this.h.invoke(this,m,new Object[]{" + paramValues + "})",m.getReturnType()) + ";" + ln);
                        sb.append("}catch(Error _ex) { }");
                        sb.append("catch(Throwable e){" + ln);
                        sb.append("throw new UndeclaredThrowableException(e);" + ln);
                        sb.append("}");
                        sb.append(getReturnEmptyCode(m.getReturnType()));
                    sb.append("}");
                }
            sb.append("}" + ln);
            return sb.toString();
    }


    private static Map<Class,Class> mappings = new HashMap<Class, Class>();
    static {
        mappings.put(int.class,Integer.class);
    }

    private static String getReturnEmptyCode(Class<?> returnClass){
        if(mappings.containsKey(returnClass)){
            return "return 0;";
        }else if(returnClass == Void.class){
            return "";
        }else {
            return "return null;";
        }
    }
    private static String getCaseCode(String code,Class<?> returnClass){
        if(mappings.containsKey(returnClass)){
            return "((" + mappings.get(returnClass).getName() +  ")" + code + ")." + returnClass.getSimpleName() + "Value()";
        }
        return code;
    }
    private static boolean hasReturn(Class<?> clazz){
        return clazz != Void.class;
    }
    private static String toLowerFirstCase(String src){
        char [] chars = src.toCharArray();
        chars[0] += 32;
        return String.valueOf(chars);
    }
}
public class MyClassLoader extends ClassLoader {
    private File classPathFile;
    public MyClassLoader(){
        String classPath = MyClassLoader.class.getResource("").getPath();
        this.classPathFile = new File(classPath);
    }
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String className = MyClassLoader.class.getPackage().getName() + "." + name;
        if(classPathFile  != null){
            File classFile = new File(classPathFile,name.replaceAll("\\.","/") + ".class");
            if(classFile.exists()){
                FileInputStream in = null;
                ByteArrayOutputStream out = null;
                try{
                    in = new FileInputStream(classFile);
                    out = new ByteArrayOutputStream();
                    byte [] buff = new byte[1024];
                    int len;
                    while ((len = in.read(buff)) != -1){
                        out.write(buff,0,len);
                    }
                    return defineClass(className,out.toByteArray(),0,out.size());
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }
        return null;
    }
}
public class MyMeipo implements MyInvocationHandler {
    private Object target;
    public Object getInstance(Object person) throws Exception{
        this.target = person;
        Class<?> clazz = target.getClass();
        return MyProxy.newProxyInstance(new MyClassLoader(),clazz.getInterfaces(),this);
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object obj = method.invoke(this.target,args);
        after();
        return obj;
    }

    private void before(){
        System.out.println("我是媒婆，我要给你找对象，现在已经确认你的需求");
        System.out.println("开始物色");
    }

    private void after(){
        System.out.println("OK的话，准备办事");
    }
}
public class MyProxyTest {
    public static void main(String[] args) {
        try {
            //JDK动态代理的实现原理
            Person obj = (Person) new MyMeipo().getInstance(new Girl());
            System.out.println(obj.getClass());
            obj.findLove();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```



2、思考：为什么JDK动态代理中要求目标类实现的接口数量不能超过65535个？

因为jvm规范规定一个方法的代码长度不超过65535个字节

3、结合自身的业务场景用代理模式进行重构。

公共类

```java
public interface IBaseDao<T> {
    void add(T o);
}
@Data
public class User {
    Integer id;
    String name;
}
```

重构前

```java
@Slf4j
public class UserDao implements IBaseDao<User> {
    @Autowired
    UserMapper mapper;
    void add(User o){
        log.info("新增开始");
        mapper.insert(o);
        log.info("新增结束");
    }
}

public class TestDemo{
    public static void main(String[] args){
        User user = new User();
        user.setId(1);
        user.setName("张三");
        new UserDao().add(user);
    }
}

```

重构后

```java
@Slf4j
public class ProxyLog implements InvocationHandler{
    private Object target;
    public Object getInstance(Object o) {
        this.target = o;
        Class<?> classz = o.getClass();
        return Proxy.newProxyInstance(classz.getClassLoader(),classz.getInterfaces(),this);
    }
    public Object invoke(Object proxy,Method method,Object[] args){
        before();
        Object o = method.invoke(this.target,args);
        after();
        return o;
    }
    private void before(){
        log.info("新增开始");
    }
    private void after(){
        log.info("新增结束");
    }
}
public class UserDao implements IBaseDao<User> {
    @Autowired
    UserMapper mapper;
    void add(User o){
        mapper.insert(o);
    }
}

public class TestDemo{
    public static void main(String[] args){
        User user = new User();
        user.setId(1);
        user.setName("张三");
        new UserDao().add(user);
    }
}


```


1、自己手写完成高仿真版本的Spring MVC部分。 

```java
@Slf4j
public class MyDispatcherServlet extends HttpServlet{
    private final String LOCATION = "contextConfigLocation";
    private List<MyHandlerMapping> handlerMappings = new ArrayList<>();
    private Map<MyHandlerMapping,MyHandlerAdapter> handlerAdapters = new HashMap<>();
    private List<MyViewResolver> viewResolvers = new ArrayList<>();
    private MyApplicationContext context;
    public void init(ServletConfig config) throws ServletException{
        context = new MyApplicationContext(config.getInitParameter(LOCATION));
        initStrategies(context);
    }
    protected void initStrategies(MyApplicationContext context){
        initMultipartResolver(context);
        initLocaleResolver(context);
        initThemeResolver(context);
        initHandlerMappings(context);
        initHandlerAdapters(context);
        initHandlerExceptionResolvers(context);
        initRequestToViewNameTranslator(context);
        initViewResolvers(context);
        initFlashMapManager(context);
    }
    private void initFlashMapManager(MyApplicationContext context){}
    private void initRequestToViewNameTranslator(MyApplicationContext context){}
    private void initHandlerExceptionResolvers(MyApplicationContext context){}
    private void initThemeResolver(MyApplicationContext context){}
    private void initLocaleResolver(MyApplicationContext context){}
    private void initMultipartResolver(MyApplicationContext context){}
    private void initHandlerMappings(MyApplicationContext context){
        String[] beanNames = context.getBeanDefinitionNames();
        try{
            for(String beanName : beanNames){
                Object controller = context.getBean(beanName);
                Class<?> clazz = controller.getClass();
                if(!clazz.isAnnotationPresent(MyController.class)) continue;
                String baseUrl = "";
                if(clazz.isAnnotationPresent(MyRequestMapping.class)){
                    MyRequestMapping requestMapping = clazz.getAnnotation(MyRequestMapping.class);
                    baseUrl = requestMapping.value();
                }
                Method[] methods = clazz.getMethods();
                for(Method method:methods){
                    if(!method.isAnnotationPresent(MyRequestMapping.class)) continue;
                    MyRequestMapping requestMapping = method.getAnnotation(MyRequestMapping.class);
                    String regex = ("/"+baseUrl+requestMapping.value().replacesAll(
"\\*",".*")).replaceAll("/+","/");
                    Pattern pattern = Pattern.compile(regex);
                    this.handlerMappings.add(new MyHandlerMapping(pattern,controller,method));
            
                }
            }
        }catch(Exception e){
            e.printStackTrace();
        }
    }
    private void initViewResolvers(MyApplicationContext context){
        String templateRoot = context.getConfig().getProperty("templateRoot");
        String templateRootPath = this.getClass().getClassLoader().getResource(templateRoot).getFile();
        File templateRootDir = new File(templateRootPath);
        for(File template:templateRootDir.listFiles()){
            this.viewResolver.add(new MyViewResolver(temlateRoot));
        }
    }
    void doPost(HttpServletRequest req,HttpServletResponse resp) throws ServletException,IOException{
        this.doPost(req,resp);
    }
    void doPost(HttpServletRequest req,HttpServletResponse resp) throws ServletException,IOException{
        try{
            doDispath(req,resp);
        }catch(Exception e){
            resp.getWriter().write("<font size='25' color='blue'>500
            Exception</font><br/>Details:<br/>" + Arrays.toString(e.getStackTrace()).replaceAll("\\[|\\]","")
            .replaceAll("\\s","\r\n") + "<font
            color='green'><i>Copyright@GupaoEDU</i></font>");
            e.printStackTrace();
        }
    }
    void doDispatch(HttpServletRequest req,HttpServletResponse resp) throws Exception{
     	MyHandlerMapping handler = getHandler(req);
        if(handler==null) {
            processDispatchResult(req,resp,new MyModelAndView("404"));
        	return;
        }
        MyHandlerAdapter ha = getHandlerAdapter(handler);
        MyModelAndView mv = ha.handle(req,resp,handler);
        processDispatchResult(req,resp,mv);
    }
    void processDispatchResult(HttpServletRequest request,HttpServletResponse response,MyModelAndView mv)throws Exception{
      	if(null==mv) return;
        if(this.viewResolvers.isEmpty())return;
        if(this.viewResolvers!=null){
            for(MyViewResolver viewResolver:this.viewResolvers){
                MyView view = viewResolver.resolveViewName(mv.getViewName(),null);
                if(view!=null){
                    view.render(mv.getModel(),request,response);
                    return;
                }
            }
        }
    }
    void MyHandlerAdapter getHandlerAdapter(MyHandleMapping handler){
    	if(this.handlerAdapters.isEmpty()) return null;
        String url = req.getRequestURI();
        String contextPath = req.getContextPath();
        url = url.replace(contextPath,"").replaceAll("/+","/");
        for(MyHandlerMapping handler:this.handlerMappings){
            Matcher matcher = handler.getPattern().matcher(url);
            if(!matcher.matches())continue;
            return handler;
        }
        return null;
    }                              
 }
```

```java
@Data
public class MyHandlerMapping{
    private Object controller;
    private Method method;
    private Pattern pattern;
    public MyHandlerMapping(Pattern pattern,Object controller, Method method){
        this.controller = controller;
        this.method = method;
        this.pattern = pattern;
    }
}

public class MyHandlerAdapter{
    public boolean supports(Object handler){
        return (handler instanceof MyHandlerMapping);
    }
    public MyModelAndView handle(HttpServletRequest req,HttpServletResponse resp,Object handler) throws Exception{
        MyHandlerMapping handlerMapping = handler;
        Map<String,Integer> paramMapping = new HashMap<>();
        Annotation[][] pa = handlerMapping.getMethod().getParameterAnnotations();
        for(int i=0;i<pa.length;i++){
            for(Annotation a : pa[i]){
                String paramName = ((MyRequestParam) a).value();
                if(!"".equals(paramName.trim())){
                    paramMapping.put(paramName,i);
                }
            }
        }
        Class<?>[] paramTypes = handlerMapping.getMethod().getParameterTypes();
        for(int i=0;i<paramTypes.length;i++){
            Class<?> type = paramTypes[i];
            if(type==HttpServletRequest.class||type==HttpServletResponse.class){
                paramMapping.put(type.getName(),i);
            }
        }
        Map<String,String[]> reqParamterMap = req.getParameterMap();
        Object[] paramValues = new Object[paramTypes.length];
        for(Map.Entry<String,String[]> param : reqParameterMap.entrySet()){
            String value = Arrays.toString(param.getValue()).replaceAll("\\[|\\]","").replaceAll("\\s","");
            if(!paramMapping.containsKey(param.getKey()))continue;
            int index = paramMapping.get(param.getKey());
            paramValues[index] = caseStringValue(value,paramTypes[index]);
        }
        if(paramMapping.containsKey(HttpServletRequest.class.getName())){
            int reqIndex = paramMapping.get(HttpServletRequest.class.getName());
            paramValues[reqIndex]=req;
        }
        if(paramMapping.containsKey(HttpServletResponse.class.getName())){
            int reqIndex = paramMapping.get(HttpServletResponse.class.getName());
            paramValues[reqIndex]=req;
        }
        Object result = handlerMapping.getMethod().invoke(handlerMapping.getController(),paramValues);
        if(result==null) return null;
        boolean isModelAndView = handlerMapping.getMethod().getReturnType()==MyModelAndView.class;
        if(isModelAndView){
            return (MyModelAndView)result;
        }else{
            return null;
        }
    }
    private Object caseStringValue(String value,Class<?> clazz){
        if(clazz==String.class){
            return value;
        }else if(clazz==Integer.class){
            return Integer.valueOf(value);
        }else if(clazz==int.class){
            return Integer.valueOf(value).intValue();
        }else{
            return null;
        }
    }
}
```

```java
@Data
public class MyModelAndView{
    private String viewName;
    private Map<String,?> model;
    public MyModelAndView(String viewName){
        this(viewName,null);
    }
    public MyModelAndView(String viewName,Map<String,?> model){
        this.viewName =viewName;
        this.model = model;
    }
 }
```

```java
public class MyViewResolver{
    private final String DEFAULT_TEMPLATE_SUFFIX=".html";
    private File templateRootDir;
    private String viewName;
    public MyViewResolver(String templateRoot){
        String templateRootPath = this.getClass().getClassLoader().getResource(templateRoot).getFile();
        this.templateRootDir = new File(templateRootPath);
    }
    public MyView resolveViewName(String viewName,Locale locale)throws Exception{
        this.viewName = viewName;
        if(null==viewName||"".equals(viewName.trim())) return null;
        viewName = viewName.endsWith(DEFAULT_TEMPLATE_SUFFIX)?viewName:(viewName+DEFAULT_TEMPLATE_SUFFIX);
        File templateFile = new File((templateRootDir.getPath()+"/"+viewName).replaceAll("/+","/"));
        return new MyView(templateFile);
    }
    public String getViewName(){
        return viewName;
    }
}
```

```java
public class MyView {

    public final String DEFULAT_CONTENT_TYPE = "text/html;charset=utf-8";

    private File viewFile;

    public GPView(File viewFile) {
        this.viewFile = viewFile;
    }

    public void render(Map<String, ?> model,
                       HttpServletRequest request, HttpServletResponse response) throws Exception{
        StringBuffer sb = new StringBuffer();

        RandomAccessFile ra = new RandomAccessFile(this.viewFile,"r");

        String line  = null;
        while (null != (line = ra.readLine())){
            line = new String(line.getBytes("ISO-8859-1"),"utf-8");
            Pattern pattern = Pattern.compile("￥\\{[^\\}]+\\}",Pattern.CASE_INSENSITIVE);
            Matcher matcher = pattern.matcher(line);
            while (matcher.find()){
                String paramName = matcher.group();
                paramName = paramName.replaceAll("￥\\{|\\}","");
                Object paramValue = model.get(paramName);
                if(null == paramValue){ continue;}
                line = matcher.replaceFirst(makeStringForRegExp(paramValue.toString()));
                matcher = pattern.matcher(line);
            }
            sb.append(line);
        }

        response.setCharacterEncoding("utf-8");
//        response.setContentType(DEFULAT_CONTENT_TYPE);
        response.getWriter().write(sb.toString());
    }


    //处理特殊字符
    public static String makeStringForRegExp(String str) {
        return str.replace("\\", "\\\\").replace("*", "\\*")
                .replace("+", "\\+").replace("|", "\\|")
                .replace("{", "\\{").replace("}", "\\}")
                .replace("(", "\\(").replace(")", "\\)")
                .replace("^", "\\^").replace("$", "\\$")
                .replace("[", "\\[").replace("]", "\\]")
                .replace("?", "\\?").replace(",", "\\,")
                .replace(".", "\\.").replace("&", "\\&");
    }
}
```

业务代码

```java
public interface IQueryService{
    public String query(String name);
}
@MyService
@Slf4j
public class QueryService implements IQueryService {
	public String query(String name) {
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		String time = sdf.format(new Date());
		String json = "{name:\"" + name + "\",time:\"" + time + "\"}";
		log.info("这是在业务方法中打印的：" + json);
		return json;
	}
}
public interface IModifyService {

	/**
	 * 增加
	 */
	public String add(String name, String addr) throws Exception;
	
	/**
	 * 修改
	 */
	public String edit(Integer id, String name);
	
	/**
	 * 删除
	 */
	public String remove(Integer id);
	
}
@MyService
public class ModifyService implements IModifyService {

	/**
	 * 增加
	 */
	public String add(String name,String addr) throws Exception {
		throw new Exception("这是Tom老师故意抛的异常！！");
		//return "modifyService add,name=" + name + ",addr=" + addr;
	}

	/**
	 * 修改
	 */
	public String edit(Integer id,String name) {
		return "modifyService edit,id=" + id + ",name=" + name;
	}

	/**
	 * 删除
	 */
	public String remove(Integer id) {
		return "modifyService id=" + id;
	}
	
}
@MyController
@MyRequestMapping("/web")
public class MyAction {

	@MyAutowired IQueryService queryService;
	@MyAutowired IModifyService modifyService;

	@MyRequestMapping("/query.json")
	public MyModelAndView query(HttpServletRequest request, HttpServletResponse response,
								@MyRequestParam("name") String name){
		String result = queryService.query(name);
		return out(response,result);
	}
	
	@MyRequestMapping("/add*.json")
	public MyModelAndView add(HttpServletRequest request,HttpServletResponse response,
			   @MyRequestParam("name") String name,@MyRequestParam("addr") String addr){
		String result = null;
		try {
			result = modifyService.add(name,addr);
			return out(response,result);
		} catch (Exception e) {
//			e.printStackTrace();
			Map<String,Object> model = new HashMap<String,Object>();
			model.put("detail",e.getCause().getMessage());
//			System.out.println(Arrays.toString(e.getStackTrace()).replaceAll("\\[|\\]",""));
			model.put("stackTrace", Arrays.toString(e.getStackTrace()).replaceAll("\\[|\\]",""));
			return new GPModelAndView("500",model);
		}

	}
	
	@MyRequestMapping("/remove.json")
	public MyModelAndView remove(HttpServletRequest request,HttpServletResponse response,
		   @GPRequestParam("id") Integer id){
		String result = modifyService.remove(id);
		return out(response,result);
	}
	
	@MyRequestMapping("/edit.json")
	public MyModelAndView edit(HttpServletRequest request,HttpServletResponse response,
			@MyRequestParam("id") Integer id,
			@MyRequestParam("name") String name){
		String result = modifyService.edit(id,name);
		return out(response,result);
	}
	
	
	
	private MyModelAndView out(HttpServletResponse resp,String str){
		try {
			resp.getWriter().write(str);
		} catch (IOException e) {
			e.printStackTrace();
		}
		return null;
	}
}
@MyController
@MyRequestMapping("/")
public class PageAction {
	@MyAutowired IQueryService queryService;
	@MyRequestMapping("/first.html")
    public MyModelAndView query(@MyRequestParam("teacher") String teacher){
        String result = queryService.query(teacher);
        Map<String,Object> model = new HashMap<String,Object>();
        model.put("teacher", teacher);
        model.put("data", result);
        model.put("token", "123456");
        return new MyModelAndView("first.html",model);
    }
}
```

```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="utf-8">
    <title>页面去火星了</title>
</head>
<body>
    <font size='25' color='red'>404 Not Found</font><br/><font color='green'><i>Copyright@GupaoEDU</i></font>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="utf-8">
    <title>服务器好像累了</title>
</head>
<body>
    <font size='25' color='blue'>500 服务器好像有点累了，需要休息一下</font><br/>
    <b>Message:￥{detail}</b><br/>
    <b>StackTrace:￥{stackTrace}</b><br/>
    <font color='green'><i>Copyright@GupaoEDU</i></font>
</body>
</html>
```


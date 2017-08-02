# Servlet 工作原理解析

## Servlet 容器
> servlet 与Servlet 容器之间的关系有点像抢和子弹的关系,抢是为子弹而生,而子弹又让抢有了杀伤力.虽然他们的是彼此依存的,但是又相互独立发展.这一切都是为了适应工业化生产的结果.从技术角度上来说是为了解耦,通过标准化接口来相互协作,既然接口是连接Servlet与Servlet 容器的关键,那我们就从他们的连接说起

1. Servlet容器作为一个独立发展的标准化产品,目前他的种类很多,但是他们都是有自己的市场定位,很难说谁优谁劣,各有特点.例如现在比较流行的Jetty,在定制化和移动领域有不错的发展,我们这里还是以大家最为熟悉的tomcat为例来介绍Servlet容器如何管理Servlet.
2. Tomcat本身也很复杂,我们只从Servlet和Servlet容器的接口部分开始介绍,关于Tomcat的详细介绍可以参考<<Tomcat系统架构与模式设计分析>>
3. 在tomcat的容器等级中,Context容器是直接官差Servlet在容器中的包装类Wrapper,所以Context容器如何运行将直接影响Servlet的工作方式.
4. Tomcat容器模型
    1. Tomcat的容器分为四个等级,真正管理Servlet的容器是Context容器,一个Context对应一个Web工程,在Tomcat的配置文件中可以很容易发现这一点:如下
    
    ```
    <Context path="/projectOne" docBase="D:\projects\projectOne" relodable="true" />
    ```

![tomcat容器模型](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image002.jpg)

## Servlet 容器的启动过程
1. Tomcat7 也开始支持嵌入式功能,增加了一个启动类org.apache.catalina.startup.Tomcat.穿件一个实例对夏姑娘并调用start方法就可以很容易的启动一个Tomcat,我们还可以通过这个对象来增加和修改Tomcat的配置参数,如可以动态增加Context/Servlet等.
2. 下面我们就可以利用这个Tomcat类来管理新增加的一个Context容器,我们选择Tomcat7自带的examples Web公车给你,并看看他是如何加到这个Context容器中的.

```
    Tomcat tomcat = getTomcatInstatnce();
    File appDir = new File(getBuildDirectory(),"webapps/examples);
    tomcat.addWebapp(null,"/examples",appDir.getAbsolutePath());
    tomcat.start();
    ByteChunk res = getUrl("http://localhost:"+getPort()+"/examples/servlets/servlet/HelloWorldExample");
    assertTrue(res.toString().indexof("<h1>Hello World!</h1>"))
```
上述代码是创建一个Tomcat实例并新增一个web应用,然后启动tomcat并调用其中的一个HelloWorldexample Servlet,看有没有正确的返回预期的数据.

```
    public Context addwebapp(Host host,String url,String path){
        silence(url);
        Context ctx = new StandardContext();
        ctx.setPath(url);
        ctx.setDocBase(path);
        if(defaultRealm == null){
            initSimpleAuth();
        }
        ctx.setRealm(defaultRealm);
        ctx.addLifecycleListener(new DefaultWebXmlListener));
        ContextConfig ctxCfg = new ContextConfig();
        ctx.addLifeCycleListener(ctxCfg);
        ctxCfg.setDefaultWebXml("org/apache/catalin/startup/NO_DEFAULT_XML");
        if(host == null){
            getHost().addChild(ctx);
        } else{
            host.addChild(ctx);
        }
        return ctx;
    }

```
前面已经介绍了一个Web应用对应一个COntext容器,也就是Servlet 运行时的Servlet容器,添加一个车Web应用时将会创建一个StandardContext(),并且给这个Context设置必要的参数url和path,分别代表这个应用在Tomcat中的访问路径和这个应用的实际物理路径,其中最重要的一个配置是COntextConfig,这个类将会负责整个Web应用配置的解析工作.最后将这个Context容器加到父容器Host中.

tomcat的start 方法的启动逻辑是基于观察者模式设计的,所有的容器都会集成LifeCycle接口,他管理着这个容器的整个生命周期,所有容器的修改和状态的改变都会由他去通知已经注册的观察者(Listener),关于这个设计模式的参考<<tomcat的系统架构与设计莫桑,第二部分:设计模式>>.

![Tomcat启动时序图](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image003.jpg)


添加 examples应用所对应的standardContext容器的启动过程.
打给你context容器初始化为init时,天剑在Context容器的Listener将会被调用.ContextCOnfig继承了LifecycleListener接口,他是在addwebapp的时候被添加到StandardContext容器中.ContextConfig类会负责整个Web应用配置文件的解析任务.

ContextConfig的init方法将完成以下工作:
1. 创建用于解析xml配置文件的contextDigester对象;
2. 读取默认context.xml配置文件.如果存在解析
3. 读取默认的host配置文件,如果存在解析
4. 读取默认的Context自身的配置文件,如果存在则解析
5. 设置context的DocBase


ContextConfig的init方法完成之后,context容器会执行StartInternal方法,这个启动主要完成以下工作:
1. 创建读取资源文件的对象
2. 创建ClassLoader对象
3. 设置应用的工作目录
4. 启动相关的辅助类:logger/realm/resources等
5. 自容器的初始化
6. 获取ServletContext并设置必要的参数
7. 初始化 load on startup的Servlet


## web 应用的初始化工作
1. 主要是要解析web.xml文件,这个文件描述了一个Web应用的关键信息,也是一个Web应用的入口
2. 将servlet包装StandardWrapper并作为子容器添加到context中,.这里的StandardWrapper是tomcat容器的一部分,它具有容器的特,而Servlet是一个独立的Web开发标准,不应该强耦合在Tomcat中.

除了将Servlet包装成StandardWrapper并作为自容器添加到Context中,器他的所有web.xml属性都被解析到Context中.其他的所有web.xml属性都被解析到context中,所以说context容器才是真正运行的Servlet的Servlet容器.一个Web应用对应一个Context容器,容器的配置属性是由web.xml指定,这样我们就能理解web.xml到底起到什么作用了.
```
for (ServletDef servlet : servlets.values()) { 
           Wrapper wrapper = context.createWrapper(); 
           String jspFile = servlet.getJspFile(); 
           if (jspFile != null) { 
               wrapper.setJspFile(jspFile); 
           } 
           if (servlet.getLoadOnStartup() != null) { 
               wrapper.setLoadOnStartup(servlet.getLoadOnStartup().intValue()); 
           } 
           if (servlet.getEnabled() != null) { 
               wrapper.setEnabled(servlet.getEnabled().booleanValue()); 
           } 
           wrapper.setName(servlet.getServletName()); 
           Map<String,String> params = servlet.getParameterMap(); 
           for (Entry<String, String> entry : params.entrySet()) { 
               wrapper.addInitParameter(entry.getKey(), entry.getValue()); 
           } 
           wrapper.setRunAs(servlet.getRunAs()); 
           Set<SecurityRoleRef> roleRefs = servlet.getSecurityRoleRefs(); 
           for (SecurityRoleRef roleRef : roleRefs) { 
               wrapper.addSecurityReference( 
                       roleRef.getName(), roleRef.getLink()); 
           } 
           wrapper.setServletClass(servlet.getServletClass()); 
           MultipartDef multipartdef = servlet.getMultipartDef(); 
           if (multipartdef != null) { 
               if (multipartdef.getMaxFileSize() != null && 
                       multipartdef.getMaxRequestSize()!= null && 
                       multipartdef.getFileSizeThreshold() != null) { 
                   wrapper.setMultipartConfigElement(new 
MultipartConfigElement( 
                           multipartdef.getLocation(), 
                           Long.parseLong(multipartdef.getMaxFileSize()), 
                           Long.parseLong(multipartdef.getMaxRequestSize()), 
                           Integer.parseInt( 
                                   multipartdef.getFileSizeThreshold()))); 
               } else { 
                   wrapper.setMultipartConfigElement(new 
MultipartConfigElement( 
                           multipartdef.getLocation())); 
               } 
           } 
           if (servlet.getAsyncSupported() != null) { 
               wrapper.setAsyncSupported( 
                       servlet.getAsyncSupported().booleanValue()); 
           } 
           context.addChild(wrapper); 
}
```

##　创建Servlet实例
1. 前面已经完成了Servlet的解析工作,并且被包装成StandardWrapper添加在Context容器中,但是它仍然不能为我们工作,它还没有被实例化.
2. 在解析配置文件时对读取默认的globalWebXml,在conf下的web.xml文件中定义了一些默认的配置项,其定义了两个Servlet,分别是org.apache.catalina.servlets.DefaultServlet和org.apahce.jasper.servlet.jspServlet他们的load-on-startup分别是1 和3 也就是当tomcat启动时这两个Servelt就会被启动.

创建Servlet实例的方法是从Wrapper.loadServlet开始的.LoadServlet方法要完成的就桑获取ServletClass然后把它交给InstanceManager去创建一个机遇ServletClass.class的对象.如果这个servlet配置了jsp-file,那么这个servletClass就是conf/web.xml中定义的org.apache.jasper.servlet.JspServlet了.
![servlet创建对象的相关类结构图](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image006.jpg)

##　初始化Servlet 
初始化Servlet在StandardWrapper的initServlet方法中,其详细调用的时序图如下:
![servlet初始化时序图](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image007.jpg)

z在StandardWrapper的initServlet方法中,这个方法很简单就是调用Servlet的init方法,同时吧包装了StandardWrapper对象的StandardWrapperFacade作为ServletConfig传给了Servlet.

## servlet体系结构
我们知道JavaWeb应用是基于Servlet规范运转的,那么Servlet本身又是如何运转的呢?威慑要设计这样的体系结构.
![servlet的体系结构](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image010.jpg)
从上图来看Servlet的规范就是基于这几个类运转的,与Servlet主动关联的是三个类:servletConfig/ServletRequest/ServletResponse.这三个类都是通过容器传给Servlet的.其中ServletConfig实在Servlet初始化的时候就传给Servlet了,而后面两个是在请求达到时调用Servlet时传递过来的.
1. servletconfig 获取Servlet运行时的配置属性
2. servletServletContext对Servlet有什么价值?
3. servlet的运行模式是一个典型的"握手型"的交互式模式.
    1. 两个模块为了交换数据通常都会准备一个交易场景,这更饿场景一直跟随着这个交易过程知道这个交易完成为止.
    2. 这个交易场景的初始化是根据这次交易对象指定的参数来指定的,这些指定的参数通常就会是一个配置类.所以对号入座,交易场景就由ServletContext来描述,而指定的参数集合就由ServletConfig来描述
    3. 而ServletRequest和ServletResponse就是要交互的具体对象了,他们通常都是作为运输工具来传递交互结果.
4. ServletCOnfig是在Servlet init时由容器传过来的,那么ServletConfig到底是什么对象呢?

![ServletConfig在容器中的类关联图](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image012.jpg)

![request相关类的结构图](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image014.jpg)

![Request和Response的转变过程](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image016.jpg)

## Servlet是如何工作的
1. 当用户从浏览器向服务器发起一个请求,通常会包含如下信息:http://hostname:port/contextpath/servletpaht,hostname和port是用来与服务器建立TCP连接,而后面的URL才是用来选择服务器中那个自容器服务用户的请求.
2. 在tomcat7 中映射工作有一个专门的一个类来完成,这就是org.apache.tomcat.util.htpp.mapper,这个类保存了tomcat的Container容器中的所有子容器的信息,当org.apache.catalina.connector.Request类在进入Container容器之前,mapper将会根据这次请求的hostname和contextpaht将host和context容器设置到Request的mappingData属性中,所以当Request进入Container容器之前,他要访问那个自容器这时候已经确定了.

![Request的Mapper类关系](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image018.jpg)

![Request在容器中的路由图](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image020.jpg)

##　Session与cookie
1. 目的:保持访问用户与后端服务器的状态
2. coookie在用的网络带宽较大
3. session不能在多台服务器之间共享.



## Servlet中的listener
1. 提供了六种常见的listener
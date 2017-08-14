---
layout: post_layout
title: TD-Summary for week 4
time: 2017年6月29日 星期四
location: 北京
pulished: true
excerpt_separator: "**以下是我在最近项目中"
---
### 第4周总结  

**1**.httpclient是一个很重的对象，如果在一个项目中多次创建新的httpclient对象，将是一件很消耗资源的事情。**CloseableHttpClient**已经是线程安全的了，所以我们没有必要考虑安全问题。项目中，我们可以考虑将创建httpclient单独提取出来实现单例模式，这样可以大大提高效率和节约资源。以下是我在最近项目中的一个简单实现：
```Java
public class HttpClientUtil {

    private HttpClientUtil(){}

    public static CloseableHttpClient getHttpClient(){
        return SingleHttpClient.httpClient;
    }

    private static class SingleHttpClient{
        private static RequestConfig requestConfig = RequestConfig.custom().setSocketTimeout(5000)
                .setConnectTimeout(5000).build();
        private static final CloseableHttpClient httpClient = HttpClients.custom().setMaxConnTotal(50)
                .setMaxConnPerRoute(25).setDefaultRequestConfig(requestConfig)
                .evictIdleConnections(30, TimeUnit.SECONDS).build();
    }
}
```
我是采用静态内部类的方式实现的单例，当然还有很多种其他的方式。这样，我们只需要在项目中需要创建httpclient的地方调用这个HttpClientUtil工具来创建一个实例即可。
```Java
CloseableHttpClient httpClient = HttpClientUtil.getHttpClient();
```
采用这种方式，我们就避免了重复创建httpclient对象所带来的不必要系统消耗。   

---
**2**.有时候我们需要根据一些请求参数来拼接访问的url，为了避免中文和一些特殊字母的编码解码问题，我们不应该像拼接字符串一样去手动拼接我们需要的url，可以用**URIBuilder**类中提供的方法来解决，这样就不会出现因为转码带来的问题。   

---
**3**.当我们对HttpResponse做处理的时候，可以采用一种**语法糖**的写法，如下：
```Java
try (CloseableHttpResponse response = httpClient.execute(httpGet)) {
    //TODO
}
catch (IOException e) {
    //TODO         
}
```
这样我们就不需要再自己去关闭HttpResponse了。因为这种语法糖的写法会帮我们做这些工作。   

---
**4**.切记一定要关闭HttpClient。接着上面的1来说，采用了节约资源的单例写法，我们需要在你程序的某个合适的地方来关闭HttpCilent，通常是在你创建的**ServerListener**类中来做一些初始化的启动和运行结束的关闭操作。比如我最近项目中的如下所示：
```Java
@Provider
public class ServerListener extends AbstractContainerLifecycleListener {
    private static Logger logger = LoggerFactory.getLogger(ServerListener.class);

	@Override
	public void onShutdown(Container container) {
		super.onShutdown(container);
		try {
			HttpClientUtil.getHttpClient().close();
		}
		catch (IOException e) {
		    //TODO
		}
		Broker.shutdown();
	}

	@Override
	public void onStartup(Container container) {
		super.onStartup(container);
		Broker.start();
	}

}
```
---
**5**.Jackson相关  
最近项目中Json相关处理采用的是Jackson这个框架，因为是第一次接触，所以碰到了些问题，现在做一些记录总结。  
**反序列问题**  
因为项目需求，我需要读取log文件中的内容，对日志内容进行相应对象的映射封装。Jackson对json字符串进行反序列涉及到的内容还挺多的，我这里不展开叙述，因为自己也才刚接触，以后有时间会做系统的总结。我只说明下我遇到的问题，并记录下我的解决办法。
```Java
SingleSmsWrapper singleSms = JsonUtil.readBean(smsRecoveredLog,SingleSmsWrapper.class);
```
如上是我反序列化的代码。
JsonUtil是我提取出来的Json处理工具类，摘取对应readBean代码如下：
```Java
public static <T> T readBean(String json, Class<T> clazz) throws IOException {
		return mapper.readValue(json, clazz);
}
```
**smsRecoveredLog**是我读取的log文件中的一行内容，我在第一次反序列化时程序报错，错误如下：
```
JsonMappingException: No suitable constructor found for type [simple type, class ]: can not instantiate from JSON object
```
解决办法是在我的待反序列化的**SingleSmsWrapper**类中加上了一个默认的构造函数：
```Java
public SingleSmsWrapper(){}
```
其实关于反序列化时还有很多参数匹配等等问题，Jackson也都提供了相应的解决办法。这里就不详细展开了。  

---
**6**.本周主要是实现了Messages模块中的重试策略和定时任务策略，已经开始接触到很多线程池相关的内容。以前都只是在面试中被人问起有没有接触过线程池，现在总算是和这个开始打交道了，很欣喜也很有压力。线程池这块会在未来再进一步的接触实战后抽时间做一次较全面的总结。


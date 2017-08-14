---
layout: post_layout
title: TD-Summary for week 3
time: 2017年6月22日 星期四
location: 北京
pulished: true
excerpt_separator: "**不同的环境可能会有不同"
---
### 第3周总结

**1**.在项目中，像List这种很大的对象能不用尽量就别用。

---
**2**.在项目开发过程中，有时候我们需要将项目部署在不同的服务器上，分别对应不同的开发环境。比如，我们在本地开发环境，测试开发环境，集成测试开发环境，生产环境等等。通常在项目中我们会将一些配置信息等内容放在配置文件中，不同的环境可能会有不同。在Maven项目中，我们可以利用maven profile来构建我们的项目，这样项目的可移植性就很好了，配置好了maven profile，对应不同的环境，程序会加载对应不同的配置文件，完全不需要我们自己在程序中去做处理。比如在我最近的项目中，在pom文件中会配置好profile属性，如下：
```Java
<profiles>
		<profile>
			<id>dev</id>
			<properties>
				<env>dev</env>
			</properties>
		</profile>
		<profile>
			<id>test</id>
			<properties>
				<env>test</env>
			</properties>
		</profile>
		<profile>
			<id>devtest</id>
			<properties>
				<env>devtest</env>
			</properties>
		</profile>
		<profile>
			<id>prod</id>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
			<properties>
				<env>prod</env>
			</properties>
		</profile>
	</profiles>
```
可以看到，我一共定义了四种不同的开发环境，与之对应的，我们在项目工程目录中，会将不同环境下的配置文件放在一起，详细的配置可参考网上教程。  
然后在我们的项目中，我们只需要将配置文件中那些配置信息声明成相应的全局变量，在程序初始化的时候，读取相应的配置文件，对变量进行初始化就可以了。比如：
```Java
public class Configuration {
	private static final Logger logger = LoggerFactory.getLogger(Configuration.class);

	private String mailSenders;
	private String mailDefaultSender;

	private int scheduleWorks;
	private int mailMaxWorkers;
	private int mailNormalWorkers;
	private int taskQueueSize;
	private boolean needTaskMetrics;

	private String smsRequestUrl;

	private Configuration() {
	}

	public static Configuration runtimeConfig() {
		return Singleton.runtimeConfig;
	}

    private static class Singleton {
		private static Configuration runtimeConfig = loadConfig();

		private static Configuration loadConfig() {
			Configuration runtimeConfig = new Configuration();
			InputStream config = Thread.currentThread().getContextClassLoader().getResourceAsStream("server.conf");
			Properties prop = new Properties();
			try {
				prop.load(config);
				runtimeConfig.scheduleWorks = NumberUtils.toInt(prop.getProperty("schedule.worker.count"),5);
                runtimeConfig.smsRequestUrl = prop.getProperty("sms.request.url");
				runtimeConfig.mailSenders = prop.getProperty("mail.senders");
				runtimeConfig.mailDefaultSender = prop.getProperty("mail.default.sender");
				runtimeConfig.mailNormalWorkers = NumberUtils.toInt(prop.getProperty("mail.worker.count"), 10);
				runtimeConfig.mailMaxWorkers = NumberUtils.toInt(prop.getProperty("mail.max.worker.count"), 30);
				runtimeConfig.taskQueueSize = NumberUtils.toInt(prop.getProperty("mail.task.queue.size"), 10000);
				runtimeConfig.needTaskMetrics = BooleanUtils
						.toBoolean(prop.getProperty("mail.pool.needmetrics", "false"));
				return runtimeConfig;
			}
			catch (IOException e) {
				logger.error("Read configuration file error!");
				throw new RuntimeException("Initialize configuration error,please check the configuration file!", e);
			}
			finally {
				try {
					config.close();
				}
				catch (IOException e) {
					logger.error("Close configuration file error!", e);
				}
			}
		}
	}
	...
	...
	...
```
总而言之，用maven profile可以大大的增强项目的可移植性。这里只是抛砖引玉的介绍一下。

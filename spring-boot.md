@参考[http://www.bysocket.com](http://www.bysocket.com)
## 注解
* @RestController=@Controller+@ResponseBody  
* @RequestMapping
* @SpringBootApplication  标记为springboot的启动器 exclude属性可以不加载某个@Configuration
* @ConfigurationProperties(prefix = "home") 可以把属性文件内容注入到实体里
* @PathVariable 作用于函数参数，会取@RequestMapping的value里的{value:regex}的值注入到参数内,可以使用正则表达式
* @Configuration 标记为类是配置类
* @Bean 标记在@Configuration的是函数上，函数名为属性名称，返回值为Bean
* @ControllerAdvice 标记为controller的aop内包含@ExceptionHandler拦截错误用 @InitBinder初始化binder @ModelAttribute默认的属性
* @InitBinder 初始化binder
* @ModelAttribute 标记为ModelAttribute
* @ExceptionHandler(Class<? extends Throwable>[]) 表示拦截标记的异常会默认注入异常到参数如果有的话
* @ResponseStatus(HttpStatus) 用来标记返回的结果是什么http状态
* @RestControllerAdvice=@ControllerAdvice+@ResponseBody
* @Primary 标志这个 Bean 如果在多个同类 Bean 候选时，该 Bean 优先被考虑。
* @MapperScan 扫描 Mapper 接口并容器管理
* @Value("${key}") 获取容器内properties的值进行注入其中#表示spel，$表示properties，一般值则为直接注入
* @Qualifier("name") 通过名字注入，可以写在参数上
* @ImportResource({"classpath:application.xml"}) 导入xml配置
* @EnableConfigurationProperties 开启属性注入

----------------------条件注解
* @ConditionalOnBean 当容器里有Bean的条件
* @ConditionalOnClass 当类路径下有指定的类的条件
* @ConditionalOnExpression 基于SpEL表达式的条件
* @ConditionalOnJava 基于Jvm版本条件
* @ConditionalOnJndi 在JNDI存在的条件下查找指定位置
* @ConditionalOnMissingBean 与ConditionalOnBean相反
* @ConditionalOnClass 与ConditionalOnClass相反
* @ConditionalOnNotWebApplication 当前项目不是web项目条件
* @ConditionalOnProperty 指定的属性是否有指定的值
* @ConditionalOnResource 类路径是否有指定的资源
* @ConditionalOnSingleCandidate 当某Bean在容器只有一个或者有多个但是指定首选(@Primary)Bean
* @ConditionalOnWebApplication 当前项目是web项目条件
* @Conditional 基础条件

## 初始化
pom继承

	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	
并且引入starter依赖（这里是web的starter）

	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	
使用SpringApplication.run(xxx.class,args);来启动，并且xxx是被标记@SpringBootApplication，args是命令行参数

读取配置初始化容器
1. 命令行参数
2. java:comp/env 里的 JNDI 属性
3. JVM 系统属性
4. 操作系统环境变量
5. RandomValuePropertySource 属性类生成的 random.* 属性
6. 应用以外的 application.properties（或 yml）文件
7. 打包在应用内的 application.properties（或 yml）文件
8. 在应用 @Configuration 配置类中，用 @PropertySource 注解声明的属性文件
9. SpringApplication. setDefaultProperties 声明的默认属性

###配置httpencodefilter
HttpEncodingProperties

###关于容器
因为可以切换容器抽象了servlet
可以通过注入ServletRegistrationBean、FilterRegistrationBean、ServletListenerRegistrationBean来实现。
服务器配置可以在server.*下设置



## freemarker
### 引入

	<groupId>org.springframework. boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
	
### 配置

	spring.freemarker.allow-request-override=false # Set whether HttpServletRequest attributes are allowed to override (hide) controller generated model attributes of the same name.
	spring.freemarker.allow-session-override=false # Set whether HttpSession attributes are allowed to override (hide) controller generated model attributes of the same name.
	spring.freemarker.cache=false # Enable template caching.
	spring.freemarker.charset=UTF-8 # Template encoding.
	spring.freemarker.check-template-location=true # Check that the templates location exists.
	spring.freemarker.content-type=text/html # Content-Type value.
	spring.freemarker.enabled=true # Enable MVC view resolution for this technology.
	spring.freemarker.expose-request-attributes=false # Set whether all request attributes should be added to the model prior to merging with the template.
	spring.freemarker.expose-session-attributes=false # Set whether all HttpSession attributes should be added to the model prior to merging with the template.
	spring.freemarker.expose-spring-macro-helpers=true # Set whether to expose a RequestContext for use by Spring's macro library, under the name "springMacroRequestContext".
	spring.freemarker.prefer-file-system-access=true # Prefer file system access for template loading. File system access enables hot detection of template changes.
	spring.freemarker.prefix= # Prefix that gets prepended to view names when building a URL.
	spring.freemarker.request-context-attribute= # Name of the RequestContext attribute for all views.
	spring.freemarker.settings.*= # Well-known FreeMarker keys which will be passed to FreeMarker's Configuration.
	spring.freemarker.suffix= # Suffix that gets appended to view names when building a URL.
	spring.freemarker.template-loader-path=classpath:/templates/ # Comma-separated list of template paths.
	spring.freemarker.view-names= # White list of view names that can be resolved.

## 整合Mybatis
### 引入

	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	
### 配置
	mybatis.config = mybatis 配置文件名称
	mybatis.mapperLocations = mapper xml 文件地址
	mybatis.typeAliasesPackage = 实体类包路径
	mybatis.typeHandlersPackage = type handlers 处理器包路径
	mybatis.check-config-location = 检查 mybatis 配置是否存在，一般命名为 mybatis-config.xml
	mybatis.executorType = 执行模式。默认是 SIMPLE

	spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8
	spring.datasource.username=root
	spring.datasource.password=123456
	spring.datasource.driver-class-name=com.mysql.jdbc.Driver

## 整合redis
### 引入依赖
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-redis</artifactId>
### 配置
	spring.redis.database=0## Redis数据库索引（默认为0）
	spring.redis.host=127.0.0.1## Redis服务器地址
	spring.redis.port=6379## Redis服务器连接端口
	spring.redis.password=## Redis服务器连接密码（默认为空）
	spring.redis.pool.max-active=8## 连接池最大连接数（使用负值表示没有限制）
	spring.redis.pool.max-wait=-1## 连接池最大阻塞等待时间（使用负值表示没有限制）
	spring.redis.pool.max-idle=8## 连接池中的最大空闲连接
	spring.redis.pool.min-idle=0## 连接池中的最小空闲连接
	spring.redis.timeout=0## 连接超时时间（毫秒）
	
## 整合dobbo
### 引入
	<groupId>io.dubbo.springboot</groupId>
    <artifactId>spring-boot-starter-dubbo</artifactId>
### 配置
	spring.dubbo.application.name=##application name
	spring.dubbo.registry.address=zookeeper://127.0.0.1:2181 ##注册中心url 协议://地址:端口
	spring.dubbo.protocol.name=dubbo ##协议名字
	spring.dubbo.protocol.port=20880 ##端口
	spring.dubbo.scan=com.youmu.maven.dubbo ##扫描包
	
# property
	--可以用${}获取其他属性的信息
	--${random.*} 可以随机属性值 比如random.uuid|long|int[min,max]|value
	--注意：
	--application.properties 配置中文值的时候，读取出来的属性值会出现乱码问题。但是 application.yml 不会出现乱码问题。原因是，Spring Boot 是以 iso-8859 的编码方式读取 application.properties 配置文件。
	--如果定义一个键值对 user.name=xxx ,这里会读取不到对应写的属性值。为什么呢？Spring Boot 的默认 StandardEnvironment 首先将会加载 “systemEnvironment” 作为首个PropertySource. 而 source 即为System.getProperties().当 getProperty时,按照读取顺序,返回 “systemEnvironment” 的值.即 System.getProperty(“user.name“)
### 参数
	server.port 本地端口
	spring.profiles.active=xxx 设置读取application-xxx.properties来当配置，切环境用
	
# 注意点
###关于打包
打包的时候必须引入↓，否则打出来的包不能运行

            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

###关于@ComponentScan
 如果不标注，只会扫描当前同级以及其以下级别(./**)
 
###关于@SpringBootApplication
可以使用↓，来替换

        @SpringBootConfiguration
        @EnableAutoConfiguration
        @ComponentScan
        
###关于AutoConfiguration
可以增加启动参数--debug来查看启动和未启动的自动化配置


## 类
	HandlerInterceptorAdapter 拦截适配器，实现HandlerInterceptor 配置

### @Configuration类
	可以继承WebMvcConfigurerAdapter类来完成一些mvc的基本配置
#### 拦截器配置
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(tokenVerifyInterceptor()).addPathPatterns("/test");
		super.addInterceptors(registry);
	}
#### 配置resource-mapping
	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/html/**").addResourceLocations("classpath:/html/");
		registry.addResourceHandler("/js/**").addResourceLocations("classpath:/js/");
		super.addResourceHandlers(registry);
	}
#### ViewController
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            //直接把视图绑定到资源
            registry.addViewController("/index").setViewName("index.htm");
            super.addViewControllers(registry);
        }
#### 路径比较器

        @Override
    	public void configurePathMatch(PathMatchConfigurer configurer) {
    		//当路径有.的时候.后面的会被忽略可以关闭后缀匹配来取消此忽略
    		configurer.setUseSuffixPatternMatch(false);
        	super.configurePathMatch(configurer);
    	}
    	
#### MessageConvert
        extendMessageConverters()重载会追加新convert
        configureMessageConverters()重载会覆盖自带的convert
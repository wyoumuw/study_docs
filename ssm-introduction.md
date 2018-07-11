# 序
本教程针对了解过数据库，servlet的新人入坑传统web项目的，通过学习几个基础框架来快速上手，本文不会讲的非常细，作为引子，现在网络上的教程已经足够多。

传统的web项目架构分为3层(javaEE)，用户访问web层，下面处理业务的为service层，service层处理的数据来源于dao层。

# 一、Spring
Spring是一个解决了许多在J2EE开发中常见的问题的强大框架。 Spring提供了管理业务对象的一致方法并且鼓励了注入对接口编程而不是对类编程的良好习惯。Spring的架构基础是基于使用JavaBean属性的Inversion of Control容器。然而，这仅仅是完整图景中的一部分：Spring在使用IoC容器作为构建完关注所有架构层的完整解决方案方面是独一无二的。 Spring提供了唯一的数据访问抽象，包括简单和有效率的JDBC框架，极大的改进了效率并且减少了可能的错误。Spring的数据访问架构还集成了Hibernate和其他O/R mapping解决方案。Spring还提供了唯一的事务管理抽象，它能够在各种底层事务管理技术，例如JTA或者JDBC事务提供一个一致的编程模型。Spring提供了一个用标准Java语言编写的AOP框架，它给POJOs提供了声明式的事务管理和其他企业事务--如果你需要--还能实现你自己的aspects。这个框架足够强大，使得应用程序能够抛开EJB的复杂性，同时享受着和传统EJB相关的关键服务。

## 1 两个核心
IOC:控制反转

AOP:切面编程

### 1.1 IOC Inversion of Control/别称ID Dependency Injection
控制反转,特指本来javaBean的属性由你来设置
        
        A a=new A();
        a.setName("youmu");

现在使用容器了之后就不需要管这些了只需要配置就好，容器自动按照需求选择并且注入，这里控制反转就是说本来是你控制的现在交给容器控制了。


如何使用呢？

    ApplicationContext context=new ClassPathXmlApplicationContext("config.xml");
    context.getBean("a");
    
其中config.xml如下

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="a" class="com.youmu.first.A">
            <property name="name" value="youmu" />
        </bean>
    </beans>
    
初学者可以吧spring容器看成一个不能after-put的Map，上面的例子就如同

    Map<String,Object> context=new HashMap<>();
    A a=new A();
    a.setName("youmu");
    context.put("a",a);
    context.get("a");
    
但是

    A a=new A();
    a.setName("youmu");
    context.put("a",a);
这部分代码已经是通过配置文件里写好，并且构造容器的时候就会做好的(实际上并不是，初学者可以这么认为)。

每个bean都会有一个name或者id，如果都没指定则会以类的全限定类名作为其名字。

上面看到的都是单例的对象，意思就是只会定义一份不过可以使用scope属性来指定

    <bean id="a" class="com.youmu.first.A" scope="singleton">
         <property name="name" value="youmu" />
    </bean>
    
scope的值有:

1. singleton：单例模式，即该bean对应的类只有一个实例;在spring 中是scope(作用范围)参数的默认值 ;
2. prototype：表示每次从容器中取出bean时，都会生成一个新实例;相当于new出来一个对象;
3. request：基于web，表示每次接受一个请求时，都会生成一个新实例;
4. session：表示在每一个session中只有一个该对象.


同样的当A里面有一个B类型的属性be的时候上面的配置文件就会变成

    <bean id="a" class="com.youmu.first.A">
        <property name="be" ref="b" />
    </bean>
    <bean id="b" class="com.youmu.first.B">
    </bean>
    
OR

    <bean id="a" class="com.youmu.first.A">
        <property name="be">
            <bean class="com.youmu.first.B">
            </bean>
        </property>
    </bean>

_**ps:注意这里的所有注入方式都是setter注入，也就是说他会调用setA和setBe方法(遵循驼峰命名规则)，如果你属性不是a了但是你提供了一个setA方法他也会调用但是类型要和set方法的参数类型一致，反之你有属性a但是没有setA方法也是没用的。**_

第二种注入方式(自动注入)

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
           default-autowire="byName"
           >
    </beans>

default-autowire可以设置自动注入的方式，他将会按照指定的（byName,byType..）方式来进行自动注入，而不用手动来写。

第三种注入方式(注解注入)

1.先配置bean包的扫描路径多个路径可以用,这里会开启注解模式(如果是4.0前的版本要手动开启注解<context:annotation-config />),这些**包以及其子包下**的东西被spring注解注释过的都将自动放入容器,分隔

    <context:component-scan base-package="com.youmu.web,com.youmu.service">
    </context:component-scan>

2.定义要放入容器的类

    @Component//标记成此注解的或者@Service @Controller @Repository的都会交给容器处理不过@Controller还有其他的处理
    //@Scope("prototype")
    public class A{
        @Autowired//标记为b属性是从容器注入的，@Autowired是一种按照类型来注入的注入方式/还有java自带的(javax下的别看错)@Resource注解可以支持在此不介绍
        @Qualifier("be")//标记名字是be，如果配合@Autowired使用则是按照Qualifier的值进行名字注入
        private B b;
    }
    
    @Component("be")//这里可以设置其name
    public class B{}
    
以上就是常见的注入方式。

#### DI 需要jar包

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-bean</artifactId>
    </dependency>

**ps:作者这里引用的方式是使用了maven并且没写版本号读者可以自行选择版本号。**

### 1.2 AOP aspect oriented  programing
aop是一种编程思想，不只是spring独有的，注意。
aop是面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。
上面的概念取自百度百科。代码是顺序执行的，但是我们常常有一些通用的业务处理可能会嵌入到代码里面，这里我们可以使用aop进行代码嵌入。

下面有一份用户注册的代码(并不能跑，有些地方是简写)

    //UserController
    User user=getParamUser(request);
    userService.register(user);
    ......
    
    //UserService#register
    ....
    user.setCreateTime(new Date());
    userMapper.insert(user);//这里使用了主流的mapper技术，mybatis章节会介绍，此处只需知道是调用数据库的插入
    ....

这里其实不只是userMapper,还能有其他的mapper的insert方法，这里其实所有的插入方法都应该需要createTime，update都应该设置updateTime等等，这些东西每次都写其实不利于开发，而且很容易会漏掉(比如作者)。
那我们就可以做一份代码插入在调用*Mapper#insert方法之前，给其参数进行setCreateTime。
建议参考：http://jinnianshilongnian.iteye.com/blog/1418596

aop在spring里面2种配置方式，

第一种:配置文件配置

    <aop:config>
        <!-- 这里是一个切点,即嵌入方法会嵌入在这些方法上 -->
        <aop:pointcut id="pc"
            expression=
            "execution(* com.youmu.*Mapper.insert(..))"/>
        <!-- 这里设定一个切面，引用了上面的切点，并且指定了切点这个切点的处理方法。aop:before说明是切点前置切入，还有很多参考上面的文章 -->
        <aop:aspect id="myAspect" ref="aspect">
            <aop:before pointcut-ref="pc" method="aspectCreateTime"/>
        </aop:aspect>
    </aop:config>
    <bean id="aspect" class="com.youmu.MyAspect" />
    
    
    public class MyAspect{
        public void before(JoinPoint joinPoint){
            BaseObject obj=(BaseObject)joinPoint.getArgs()[0];
            obj.setCreateTime(new Date());
      }
    }
    
第二种:注解配置(这里我们使用aspectJ的切面接口)
    
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
    
这里使用包扫描模式注入切面bean

    @Component
    @Aspect//标注为一个切面
    public class Aspect{
    
          @Before("execution(* com.youmu.*Mapper.insert(..))")
          public void before(JoinPoint joinPoint){
            BaseObject obj=(BaseObject)joinPoint.getArgs()[0];
            obj.setCreateTime(new Date());
          }
    }
    
    
上面的所有嵌入的方法都可以有参数JoinPoint作为上下文,里面可以拿到方法可以拿到方法的所有参数等等。

如果是around的话入参可以是ProceedingJoinPoint，由于around会吧整个方法都拦截下来，只有手动执行ProceedingJoinPoint#proceed才会执行目标方法，所以around可以看做其他的通知(before,after..)的大整合，而且控制很广你甚至可以不去执行原方法。

#### AOP需要jar包

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
    </dependency>

### 数据库事务管理
数据库事务(Database Transaction) ，是指作为单个逻辑工作单元执行的一系列操作，要么完全地执行，要么完全地不执行。

spring里面提供了完成数据库事务的管理，更多内容请参考《跟我学spring3》里的介绍，这里只做最简单介绍。

    <!--mysql配置-->
    <!--首先需要配置一个数据库用来访问数据库的-->
    <bean name="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="username" value="root"></property>
        <property name="password" value=""></property>
        <property name="url" value="jdbc:mysql://localhost:3306/test?serverTimezone=UTC"></property>
    </bean>
    
    <!--然后配置事务管理器-->
    <bean name="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <!--开启注解形式-->
    <tx:annotation-driven />
    
之后就可以使用注解@Transactional来标注事务型方法了
    
    //执行增加文章和修改用户信息的操作，其中有任何一个抛出异常就会回滚，也就是更改用户出现异常，则插入的文章也会回滚到没插入之前，调用者不能捕获此异常，直到丢出到spring容器里才会回滚。
    @Transactional
    public void commit(Article article){
        //add article to database
        //change user info
    }
    
大家应该注意到，这里是开启了注解的形式进行事物配置，当然还有使用xml进行配置，由于篇幅会过长，读者只需要知道事物其实是使用了aop实现即可，具体可以参考《跟我学spring3》此文章针对于快速了解和实战的。
    
 



## 2 spring 整合web

上面介绍了spring的基础，那来说说他是怎么整合到web项目里的，首先web工程启动的时候呢会加载web.xml作为工程的配置，这里要加
  
      <listener>
        <!-- 监听web容器，这里会在启动的时候去加载ApplicationContext -->
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      </listener>
      <!-- 指定配置文件 -->
      <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:context.xml</param-value>
      </context-param>
      
这里的classpath代表的是放class文件的跟路径，也就是和顶级包名同级的路径，一般的maven web工程的classpath是WEB-INF/classes(注意是打包后的不是源工程)
至此web配置完成，不过其实还需要其他的web框架如springmvc和struts等才行。

# spring mvc

springmvc是spring里面提供的一套对web的控制层的框架，用来接收和转发客户端请求。
大意可以认为其流程是这样的:

![Alt text](/resources/ssm-introduction/simple_architecture.jpg)
    
**实际不是的，不过一开始可以那么认为，上面那张图和实际流程差距有点大所以这里特地贴出原执行流程**

![Alt text](/resources/ssm-introduction/full_architecture.jpg) 

从第一个图我们可以了解到springmvc最核心的就是DispatcherServlet
配置可想而知:
    
    //web.xml
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
    
我们看到上面又配了个spring的配置,这里会配置一个新的容器，同时也要配置扫包，并且这个容器会自动关联已有的容器作为父容器(略)，这里配置/*会拦截所有请求包括html,js,css等(配置成/会有所不同)。

这里再附上spring-mvc.xml

    <mvc:annotation-driven/>
    <context:component-scan base-package="com.youmu.web"/>

注意，web包下的东西会放到mvc容器里，如果扫描过多会导致一个bean在mvc容器下有，在父容器下也有的情况。

之后可以使用@Controller来标记一个控制器，使用@RequestMapping来标记一个路径的映射，例子:

    //可以访问/youmu/hello
    @Controller
    @RequestMapping("/youmu")
    public class YoumuController{
        
        @RequestMapping("/hello")
        public String hello(String name){
            return "view/page.htm";
        }
    }
    
不过这里其实并不会跳到page.html，因为并没有配置viewResolver，并不会识别成view。一般来说是不需要的除非特殊情况需要可以配置类似于模板引擎就需要(freemarker之类的)，不然可以使用资源映射将html,css,js的访问权限开一部分出去，类似下面的配置。

    <mvc:resources mapping="/**.html" location="/"/>
    
这样会把所有的html请求都映射到web根目录下的对应名字的资源，比如访问/index.html->src/main/webapp/index.html(maven的web工程)

**ps:\*\*代表复数路径，\*代表当前路径这个通用于spring内部很多地方(execution表达式，路径配置等等，还有classpath和classpath\*的区别等等)**


这里介绍几个新的注解

* @RequestMapping("path",method,....) 请求映射的注解，可以吧一个请求映射到一个方法，如果标记在类上面则这个类下的所有标记@RequestMapping都会加上这个注解的value值作为前缀，参照上面的例子。
* @RequestParam("param",required) 标注一个方法的请求的参数，不写也可以不过会以函数参数名作为请求参数名，写的时候是必要参数，不写则默认是不必要参数。
* @ResponseBody 标记一个类的所有请求方法或者这个被标记的方法所返回的内容会序列化(通过messageConverter)后写到response上(类似于ajax)。
* @PathVariable("pathvar") 标记与方法参数，他会映射路劲参数，比如@RequestMapping("/{name}") public String methodA(@PathVariable("name") String name)。

知道上面的基本够用了，接下看一个简单的例子

    @Controller
    @RequestMapping("/user")
    public class UserController{
        
        // example POST /user -form name:youmu
        @RequestMapping(method=HttpMethod.POST)
        @ResponseBody
        public String add(@RequestParam String name){
            User user=new User(name);
            //save user...
            return "success";
        }
        
        // example GET /user
        @RequestMapping(method=HttpMethod.GET)
        @ResponseBody
        public List<User> getList(){
            List<User> users=//get users
            return users;
        }
        
        // example GET /user/1
        @RequestMapping(value="/{id}", method=HttpMethod.GET)
        @ResponseBody
        public User get(@PathVariable("id") Integer id){
            User user=//get user by id
            return user;
        }
    }

注意如果你返回的是User如上面说的会序列化的话就需要对应的序列化工具，比如说jackson(json)，fastjson(json)等等，或者也可以自己写MessageConverter来实现自己的序列化方法，这里演示实验jackson进行序列化:

    <mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
        </mvc:message-converters>
    </mvc:annotation-driven>


#### mvc所引用的包

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
    </dependency>

# mybatis

mybatis是一个数据库访问框架，提供简单的数据库访问操作等等。由于介绍的是ssm框架，则直接是对接spring的，如果读者有兴趣可以自行查阅资料学习无spring时的配置。

## 基本使用
spring整合配置:
  
    <bean name="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <!-- 暂时不需要过多的配置 -->
        <!-- <property name="configLocation"  value="classpath:mybatis-config.xml"/> -->
        <!-- 自动扫描mapping.xml文件 -->
        property name="mapperLocations" value="classpath:mapper/*.xml" />
    </bean>
    
ps:mybatis-config.xml是配置mybatis的一些属性的，具体参考：http://www.mybatis.org/mybatis-3/zh/configuration.html，
如果没有过多配置也可以不配，如上。

这样可以使用sqlSessionFactory来创建session来执行sql

    提供一下mapper.xml
    //classpath:mapper/UserMapper.xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- 名字空间作为Mapper类的全名-->
    <mapper namespace="userMapper">
        <!-- 这里type写映射类的全名 -->
        <resultMap id="BaseResultMap" type="com.youmu.User">
          <id column="id" jdbcType="BIGINT" property="id" />
          <result column="name" jdbcType="VARCHAR" property="name" />
        </resultMap>
        <!-- 这里id写方法的名字 #{@Param的值}表示这个占位符将会使用入参替换掉，这里还有${}不过不建议使用，原因读者自行了解-->
        <select id="get" resultMap="BaseResultMap">
        select * from user where id=#{id}
        </select>
    </mapper>
    
调用
    
    //UserService.java
    ...
    Map map=new HashMap();
    map.put("id",1);
    //这里第一个参数是mapper的namespace.id组成的，openSession()会得到一个SqlSession里面有各种方法，select,update..他会分别对应其<select><update>...
    sqlSessionFactory.openSession().selectList("userMapper.get",map);
    ...
    
当然也可以使用mapper接口的方式

    <!-- 配置mapper的扫描，将会把basePackage下的所有接口作为mapper放到spring容器里-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.youmu.mapper"></property>
    </bean>

这样定义一个mapper在com.youmu.mapper

    public interface UserMapper{
        //参数要附带@Param注解来指定参数名，否则会引用不到参数问题
        User get(@Param("id") Integer id);
    }

然后在xml里面可以

    //classpath:mapper/UserMapper.xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- 名字空间作为Mapper类的全名-->
    <mapper namespace="com.youmu.mapper.UserMapper">
        <!-- 这里type写映射类的全名 -->
        <resultMap id="BaseResultMap" type="com.youmu.User">
          <id column="id" jdbcType="BIGINT" property="id" />
          <result column="name" jdbcType="VARCHAR" property="name" />
        </resultMap>
        <!-- 这里id写方法的名字 #{@Param的值}表示这个占位符将会使用入参替换掉，这里还有${}不过不建议使用，原因读者自行了解-->
        <select id="get" resultMap="BaseResultMap">
        select * from user where id=#{id}
        </select>
    </mapper>

这样子配置好后就可以直接注入UserMapper对象(从spring容器里)，然后调用UserMapper#get方法即可。

既然有配置的方式，那肯定也有注解的形式啦，这时可以不配置其mapperLocations属性，修改UserMapper类
    
    public interface UserMapper{
        @Select("select * from user where id=#{id}")
        //这里是结果映射
        @Results({
        @Result(column = "id", property = "id", jdbcType = JdbcType.INTEGER, id = true),
        @Result(column = "user_type", property = "userType", jdbcType = JdbcType.INTEGER),
        ...
        })
        User get(@Param("id") Integer id);
    }
    
使用Provider的形式

        //UserMapper.java
        public interface UserMapper{
            //第一个参数是sql的提供类，method是用这个类下的这个方法来提供sql
            @SelectSelectProvider(type=UserMapperProvider.class,method="get")
            //自动映射
            User get(@Param("id") Integer id);
        }
        
        //UserMapperProvider.java
        public class UserMapperProvider{
            
            //入参是所有参数的map
            public String get(Map<Object,Object> param){
                //这里可以使用StringBuilder按照入参条件拼接一条完整sql
                return "select * from user where id=#{id}";
            }
        }
        
还有更多内容参考网上资料吧....

## 逻辑判断
xml的mapper配置可以写逻辑，比如常见的判断if

    <!--UserMapper.xml-->
    ....
    select * from user
    <!--判断入参的type是不是null，如果不是null就用表的user_type列作为sql的查询条件-->
    <if test="type != null">
        user_type=#{type}
    </if>
    ...
    
循环
    
    <!--UserMapper.xml-->
    ....
    select * from user
    <!--使用入参的types(Collection<Integer>类型)找出所有用户类型是types里面的任意一种的-->
    where user_type in
    <foreach item="type" index="index" collection="types"
      open="(" separator="," close=")">
        #{type}
    </foreach>
    ...
    
更多参考:http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html

#### mybatis整合所引用的包

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
    </dependency>
    <!-- 这里还需要使用的数据库相关的JDBC连接 -->

**_至此初学者应该已经会搭出一个SSM框架了。_**

下面我们来介绍一下附加的框架

# 通用Mapper 
这是一个国人写的，对于mybatis的查询提供基本方法的框架，我们来看看其作用。

第一部，配置:
    
    <bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.youmu.maven.mapper"></property>
        <property name="properties">
            <value>
                    mappers=tk.mybatis.mapper.common.Mapper
            </value>
        </property>
    </bean>
    
这里的配置其实只需要把mapper扫描类换成tk的即可(org.xxx->tk.xxx),下面那个properties不写也可以的，作用留给读者自行了解。

接下来需要把mapper改造一下
    
    //增加继承接口tk.mybatis.mapper.common.Mapper(注意包),注意泛型，他将会自动映射到泛型的类的属性上
    public interface UserMapper extends Mapper<User>{
        //参数要附带@Param注解来指定参数名，否则会引用不到参数问题
        User get(@Param("id") Integer id);
    }

在service里就可以直接使用了。

    //UserService
    ...
    //根据性别和用户类型进行查询
    List<User> get(int sex,Set<Integer> types){
        Example example=new Example(User.class);
        Example.Criteria criteria=example.createCriteria();//这是创建where条件
        //in查询,注意第一个参数是属性名不是列名！
        criteria.andIn("userType",types);
        criteria.andEqualTo("sex",sex);
       return userMapper.selectByExample(example);
    }
    
    void insert(User user){
        //这里有两种方法一种是xxx另一种是xxxSelective,这里举xxx=insert的例子，update也是一样的。
        //insert会吧user的所有属性都插入如果属性值为null则会强行插入null。
        //但是insertSelective是会插入非null的字段，这时null的字段会取数据库默认值。
        userMapper.insertSelective(user);
    }
    
    User get(Integer id){
        //根据主键查询
        return userMapper.selectByPrimaryKey(id);
    }

**ps:注意criteria查询函数的第一个参数是属性名不是列名！注意criteria查询函数的第一个参数是属性名不是列名！重要的事情说3遍**

下面还要特别列出User类

    //这些都是jpa注解，javax.persistence下
    //指定表名
    @Table(name="user")
    public class User{
        
        //标记为主键
        @Id
        //标记映射的列名
        @Column(name="id")
        private Integer id;
        
        @Column(name="name")
        private String name;
        
        @Column(name="user_type")
        private Integer userType;
        
        ...
    }

为什么要特地列出User类呢，类似于上面的方法

    userMapper.selectByPrimaryKey(id);
    
这里如果不使用@Id指定主键则**会把所有属性打包作为主键**，至于表名和列名有其自定义转换规则，比如驼峰，读者自行了解。

#### 包
    
    <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper</artifactId>
    </dependency>
    
# 分页插件
由于介绍的是tk的通用mapper(如果是用mybatis plus或者spring data jpa作为数据库层的实现则不需要这个插件)，那下面就介绍其兄弟分页插件。

先看看传统分页怎么做
    
    UserMapper.xml
    ...
    select * from user
    <!-- 从第offset数据开始取，取pageSize条,注意offset不是页号。offset=(页号-1)*页大小-->
    limit #{offset},#{pageSize}
    ...

其实这样做非常麻烦，原因是要自己去计算其offset,还要写一个其对应的count查询，非常复杂。如果有分页插件的话则就会方便很多，下面我们看看是怎么操作的。

配置:

其实现是mybatis的插件，插件可以配置到上面没列出来的**mybatis-config.xml**里，这里选择最简单的配置方式，在配置sqlSessionFactory的时候添加插件

    <bean name="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <!-- 暂时不需要过多的配置 -->
        <!-- <property name="configLocation"  value="classpath:mybatis-config.xml"/> -->
        <!-- 自动扫描mapping.xml文件 -->
        property name="mapperLocations" value="classpath:mapper/*.xml" />
        <!--配置分页插件-->
        <property name="plugins">
            <array>
                <bean class="com.github.pagehelper.PageInterceptor">
                    <property name="properties">
                        <value>
                            helperDialect=mysql
                        </value>
                    </property>
                </bean>
            </array>
        </property>
    </bean>

之后就可以时候了
    
    //UserService
    ...
    //根据性别和用户类型进行查询
    PageInfo<User> get(int sex,Set<Integer> types,int page,int pageSize){
        Example example=new Example(User.class);
        Example.Criteria criteria=example.createCriteria();//这是创建where条件
        //in查询
        criteria.andIn("userType",types);
        criteria.andEqualTo("sex",sex);
        //开始分页
        PageHelper.startPage(page,pageSize);
        //这个时候查询出来的结果其实是个Page<User>只不过Page实现了接口List
        List<User> users=userMapper.selectByExample(example);
        //构建分页对象
        PageInfo<User> userPager=new PageInfo<>(users);
        return userPager;
    }
    
PageInfo对象里包含了很丰富分页的属性。
    
### 包

    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
    </dependency>
        
# 参考:
*   spring是做什么的 https://blog.csdn.net/kangqianshengshi/article/details/44263017
*   跟我学spring3 http://jinnianshilongnian.iteye.com/blog/1482071
*   springmvc结构 https://www.cnblogs.com/hujiapeng/p/5765636.html
## spring boot学习

## spring 基础
### 一、Maven
- groupId: 组织的唯一标识
- artifactId: 项目的唯一标识
- 变量(可在dependency中引用)
- - <properties>
- -    <xx>yy</xx>
- - </properties>
- - <dependency><version>${xx}</dependency>


### 二、 Bean

#### 声明(以下的注解都能注册为bean)
- @Component: 没有明确的角色
- @Service: 在业务逻辑层(service层)使用
- @Repository: 在数据访问层(dao层)使用
- @Controller: 在展现层(MVC->Spring MVC)使用

#### bean注入 （可注解在set方法或者属性上，最好注解在属性上）
- @Autowired：Spring 提供的注解
- @Inject: JSR-330提供的注解
- @Resource: JSR-250提供的注解

#### @ConponentScan("包名")
- 自动扫描包名下所有使用@Service @Component @Repository @Controller的类，并注册为Bean

#### @Configuration
- 声明这是一个配置类，他的作用与在xml文件中<Beans>的作用一样

### 三、AOP

#### @AspectJ 声明一个切面
#### @After @Before @Around 拦截规则 
#### @PointCut 切点(可被上面的拦截规则复用) 

#### 实例
注解式被拦截类
- 先声明一个注解:
```java
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface Action {
        String name();
    }
```

- 编写使用注解的被拦截类

```java
    @Service
    public class DemoAnnotationsService {
        @Action(name="注解式被拦截类的add操作")
        public void add(){}
    }
```

方法规则被拦截类
```java
    @Service
    public class DemoMethodService {
        public void add(){}
    }
```

编写切面
```java
    @Aspect
    @Component
    public class LogAspect {
        //切点-> 上面的Action注解
        @Pointcut("@annotation(包名.Action)")
        public void annotationPointCut() {}

        @After("annotationPointCut()")
        public void after(JoinPoint joinPoint) {
            MethodSignature signature = (MethodSignature) joinPoint.getSignature();
            Method method = signature.getMethod();
            System.out.println(action.name());
        }

        //方法规则拦截
        @Before("execution(*包名.DemoMethodService.*(..))")
        public void before(JoinPoint joinPoint) {
            MethodSignature signature = (MethodSignature) joinPoint.getSignature();
            Method method = signature.getMethod();
            System.out.println(method.getnName());
        }
    }
```

关于[excecution](http://blog.csdn.net/abcd898989/article/details/50809321)

编写配置类
```java
    @Configuration
    @ComponentScan("包名")
    @EnableAspectJAutoProxy // 使用这个注解来开启spring对AspectJ的支持
```

测试类
```java
    public class Main() {
        public static void mian(String[] args) {
            AnnotationConfigApplicationContext = context = new AnnotationConfigApplicationContext(AopConfig.class);
            DemoAnnotationService demoAnnotationService = context.getBean(DemoAnnotationService.class);

            DemoMethodService demoMethodService = context.getBean(DemoAnootationService.class);
            demoAnnotationService.add();
            demoMethodService.add();
            context.close();
        }
    }
```

### 四 、Spring 常用配置

#### @Scope
- Singleton: 一个spring"容器"(多个容器失效)只有一个Bean的实例，是spring的默认配置
- Prototype:每次调用新建一个Bean实例 
- Request: Web项目中，给每个http request新建一个Bean实例
- Session: Web项目中，给每个http session 新建一个Bean实例
- GlobalSession: 这个只在portal应用中有用，给每一个global http session新建一个bean实例
- StepScope: 这个在spring batch里面有用
```java
    @Service
    @Scope("prototype")
    public class DemoClass {}
```

#### Spring EL和资源调用
```java
    //注入字符串
    @Value("xxx")
    private String xx;

    //注入osName（从配置文件）
    @Value("#{systemProperties['os.name']}")
    private String osName;

    //注入随机数
    @Value("#{ T(java.lang.Math).random() * 100.0}")
    private double randomNumber;

    //注入资源
    @Value("classpath:xxx/test.txt")
    private Resource testFile;

    @Value("http://www.baidu.com")
    private Resource testUrl;


    //注入字符串，从配置文件注入，首先需要在类上面写上@PropertySource("classpath:xxx/test.properties")配置文件路径，然后得写PropertySourcesPlaceholderConfigurer的Bean，然后用"$"取值
    @Value("${book.name}")
    private String bookName;

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyConfigure() {
        return new PropertySourcesPlaceholderConfigurer();
    }
```

#### bean的构造与销毁(initMethod, destoryMethod)
```java
    public class Test {
        public void init() {
            System.out.println("init"); 
        }

        public Test() {

        }

        public void destory() {
            System.out.println("destory");
        }
    }

    @Configuration
    @ComponentScan("包名")
    public class PrePostConfig {
        @Bean(initMethod="init", destoryMethod="destory")
        Test test() {
            return new Test();
        }
    }
```

#### Profile (环境切换，比如线上和开发)
```java
    public class DemoBean{
        String content;
        public DemoBean(String content) {
            this.content = content;
        }
    }
    
    @Configuration
    public class ProfileConfig {
        @Bean 
        @Profile("dev")
        public DemoBean devDemo() {
            return new DemoBean("dev")
        }

        @Bean 
        @Profile("prod")
        public DemoBean devDemo() {
            return new DemoBean("prod")
        }
    }

    public class Main {
        public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
            context.getEnvironment().setActiviteProfiles("prod"); //将活动的profile设置为prod
            context.register(ProfileConfig.class);//注册Bean配置类
            context.refresh();//刷新容器
            DemoBean demoBean = context.getBean(DemoBean.class);
            context.close();
        }
    }

```

#### 事件(Application Event)
spring 事件监听流程
- 自定义事件，继承ApplicationEvent
- 定义事件监听器, 实现ApplicationListerner;
- 使用容易发布事件.

```java
    //自定义事件
    public class DemoEvent extends ApplicationEvent {
        private static final long serialVersionUID = 1L;
        private String msg;

        public DemoEvent(Object source, String msg) {
            super(source);
            this.msg = msg;
        }

        //...get set 方法
    }

    //事件监听器

    @Component
    public class DemoListener implements ApplicationListener<DemoEvent> {
        public void onApplicationEvent(DemoEvent event) {
            String msg = event.getMsg();
            System.out.println("收到消息" + msg);
        }
    }

    //事件发布

    @Component
    public class DemoPulisher {
        @Autowired
        ApplicationContext applicationContext;
        public void publish(String mgs) {
            applicationContext.publishEvent(new DemoEvent(this, msg));
        }
    }

    //配置类
    @Configuration
    @ComponentScan("包名")
    public class EventConfig {

    }

    public class Main {
         public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
            DemoPulisher demoPulisher = context.getBean(DemoPublisher.class);
            demoPulisher.publish("xxx");
            context.close();
        }
    }
```

### spring 高级使用
#### 多线程
@EnableAsync， @Async
```java
    @Configuration
    @ComponentScan("包名")
    @EnableAsync
    public class TaskExecutorConfig implements AsyncConfigurer {
        @override
        public Executor getAsyncExecutor() {
            ThreadPoolTaskExecutor taskExecutor = new ThreadPollTaskExecutor();
            taskExecutor.setCorePoolSize(5);
            taskExecutor.setMaxPoolSize(10);
            taskExecutor.setQueueCapactity(25);
            taskExecutor.initialize();
            return taskExecutor;
            //返回ThreadPollTaskExecutor;
        }

        @Override
        public AsyncUncaughtExceptionhandler getAsyncUncaughtExceptionHandler() {

        }
    }

    @Service
    public class AsyncTaskService {
        @Async//表明是一个异步方法，如果注解在类级别，那么整个类的所有方法都是异步的，这里的方法会被自动注入使用ThreadPoolTaskExecutor作为TaskExecutor.
        public void executeAsyncTask(Integer i) {
            System.out.println("xx");
        }

        @Async
        public void executeAsyncTaskPlus(Integer i) {

        }
    }
```

#### 计划任务
@EnableScheduling, @Scheduled(包含cron,fixDelay,fixRate等)

```java
    @Service
    public class ScheduledTaskService {
        private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");
        @Scheduled(fixedRate = 5000)
        public void reportCurrentTime() {
            System.out.println(dateFormat.format(new Date()));
        }

        @Scheduled(cron = "0 28 11 ? * *") //每天11点28分执行  cron是UNIX和类UNIX下同下的定时任务
        public void fixTimeExecutino() {
            System.out.println("在指定时间执行")
        }
    }

    @Configuration
    @ComponentScan
    @EnableScheduling
    public class TaskSchedulerConfig {

    }
```

#### 条件注解@Conditional
通过不同的条件创建不同的Bean对象(例子：不同系统创建不同bean)
- windows
```java
    public class WindowsCondition implements Condition {
        public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
            return context.getEnvironment.getProperty("os.name").contains("Windows");
        }
    }
```
- linux
```java
    public class LinuxCondition implements Condition {
        public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
            return context.getEnvironment.getProperty("os.name").contains("Linux");
        }
    }
```

- 不同系统下的接口
```java
    public interface ListService {
        public String showListCmd();
    }
```
- Windows下要创建的Bean
```java
    public class WindowsListService implements ListService {
        @Override
        public String showListCmd() {
            return "dir";
        }
    }
```

- Linux下要创建的Bean
```java
    public class LinuxListService implements ListService {
        @Override
        public String showListCmd() {
            return "ls";
        }
    }
```

- 配置类
```java
    @Configuration
    public class ConditionConfig {
        @Bean
        @Conditional(WindowsCondition.class)
        public ListService windowsListService() {
            return new WindowsListService();
        }

        @Bean
        @Conditional(LinuxCondition.class)
        public ListService LinuxListService() {
            return new LinuxListService();
        }
    }
```

### Spring MVC 常用注解
#### @Contraller 在spring mvc中声明控制器的时候，只能使用@Controller
#### @RequestMapping 映射 可注解在类或者方法上，并且支持Servlet的request和response参数。
#### @ResponseBody 支持讲返回值放在response体内，而不是返回一个页面，返回json格式等数据，用到这个
#### @RequestBody 允许request的参数在request体类，而不是直接在链接地址后面，这个注解需要放在参数前面
#### @PathVariable 用来接受路径参数,如/news/001，可接收001为参数，此注解需要放在参数前
#### @RestController 组合注解，组合了@COntroller和@ResponseBody
```java
@Controller
@RequestMapping("/anno")
public class DemoAnnoController {

    @RequestMapping(produces = "text/plain;charset=UTF-8") //produces 定制返回的response的媒体类型和字符集，如果需要json，则可以改成produces = "application/json;charset=UTF-8"
    public @ResponseBody String index(HttpServletRequest request) { 
        return "url:" + request.getRequestURL() + " can access";
    }

    @RequestMapping(value = "/pathvar/{str}", produces = "text/plain;charset=UTF-8")// 这里的str 可通过@PathVariable来取
    public @ResponseBody String demoPathVar(@PathVariable String str,
            HttpServletRequest request) {
        return "url:" + request.getRequestURL() + " can access,str: " + str;
    }

    @RequestMapping(value = "/requestParam", produces = "text/plain;charset=UTF-8")
    public @ResponseBody String passRequestParam(Long id,
            HttpServletRequest request) {
        
        return "url:" + request.getRequestURL() + " can access,id: " + id;

    }

    @RequestMapping(value = "/obj", produces = "application/json;charset=UTF-8")//参数解析到对象上，/obj?id=1&name=xx
    @ResponseBody
    public String passObj(DemoObj obj, HttpServletRequest request) {
        
         return "url:" + request.getRequestURL() 
                    + " can access, obj id: " + obj.getId()+" obj name:" + obj.getName();

    }

    @RequestMapping(value = { "/name1", "/name2" }, produces = "text/plain;charset=UTF-8")//不同路径映射到同一个方法上
    public @ResponseBody String remove(HttpServletRequest request) {
        
        return "url:" + request.getRequestURL() + " can access";
    }

}
```

### Spring MVC 基本配置
- 通过一个配置类，继承WebMvcConfigurerAdapter类，并且在类上使用@EnableWebMvc注解，来开启对Spring MVC的配置支持.
#### 静态资源映射与拦截器配置
程序中的静态资源需要直接访问，可以在配置里面重写addResourceHandlers方法来实现。
拦截器: 可让普通的Bean实现HanlderInterceptor借口或者继承HandlerINterceptorAdapter类来实现自定义拦截器
```java
@Configuration
@EnableWebMvc// 开启支持
@EnableScheduling
@ComponentScan("com.wisely.highlight_springmvc4")
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {

        registry.addResourceHandler("/assets/**").addResourceLocations(
                "classpath:/assets/");// addResourceHandler是指暴露在外部的访问路径，adResourceLocations是指文件防止的路径.这里的classpath指的是: src/main/resources  这个可以自己配置

    }

    @Bean
    // 配置自定义拦截器
    public DemoInterceptor demoInterceptor() {
        return new DemoInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {//注册拦截器
        registry.addInterceptor(demoInterceptor());
    }

    //简单的页面转向，比如请求/index，然后直接返回"index"页面，那么可以直接把配置写在下面
    请求/toUpload 直接返回“upload”页面
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/index").setViewName("/index");
        registry.addViewController("/toUpload").setViewName("/upload");
        registry.addViewController("/converter").setViewName("/converter");
        registry.addViewController("/sse").setViewName("/sse");
        registry.addViewController("/async").setViewName("/async");
    }

    //在SpringMvc中，路径参数如果带"."的话， “.”后面的值将被忽略，例如http://localhost:8080/abc/def/xx.yy，那么yy将被忽略，重写下面方法，不忽略
     @Override
     public void configurePathMatch(PathMatchConfigurer configurer) {
     configurer.setUseSuffixPatternMatch(false);
     }

    @Bean
    public MultipartResolver multipartResolver() {
        CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
        multipartResolver.setMaxUploadSize(1000000);
        return multipartResolver;
    }
    
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(converter());
    }
    
    @Bean 
    public MyMessageConverter converter(){
        return new MyMessageConverter();
    }
}

public class DemoInterceptor extends HandlerInterceptorAdapter {//自定义拦截类

    @Override
    public boolean preHandle(HttpServletRequest request, //在请求发生之前执行
            HttpServletResponse response, Object handler) throws Exception {
        long startTime = System.currentTimeMillis();
        request.setAttribute("startTime", startTime);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, //在请求发生之后执行
            HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {
        long startTime = (Long) request.getAttribute("startTime");
        request.removeAttribute("startTime");
        long endTime = System.currentTimeMillis();
        System.out.println("本次请求处理时间为:" + new Long(endTime - startTime)+"ms");
        request.setAttribute("handlingTime", endTime - startTime);
    }

}
```

### Spring Boot 基础
#### 加载XML
@ImportResource({"classpath:some-context.xml", "classpath:anther-context.xml"})

#### 加载外部配置
@PropertySource指明properties文件的位置，通过@Value注入值。
```java
    @Value(${xx.yy})
    private String bookAuthor;
```

####类型安全的配置
在配置文件中加入(如果是新建一个properties文件，则需要在@ConfigurationProperties的locations里指定properties的位置，并且需要在入口类上配置)
```java
author.name = wyf
author.age = 32
```
Bean
```java
    @ComponentScan
    @ConfigurationProperties(prefix="author") //author的前缀
    public class AuthorSettings {
        private String name;
        private Long age;
        //get set method
    }
```
使用：
```java
    @Autowired
    private AuthorSettings author;
```

#### 日志配置(spring boot 默认使用 Logback作为日志框架)
文件配置
- logging.file=D:/mylog/log.log
配置级别
- logging.level.org.springframework.web = DEBUG

#### Profile配置
全局Profile配置
- application-{profile}.properties
- 比如：application-prod.properties  or  application-dev.properties
- 在application.properties中指定:   spring.profiles.active=prod 或者 spring.profiles.active=dev

#### 查看已启用和未启用的自动配置的报告
- java -jar xx.jar --debug
or
- 在 application.properties中设置属性: debug=true


#### 自定义自动配置， starter pom
(1)
```xml
//增加spring boot自身的自动配置作为依赖
<dependency>
<groupId>org.springframework.book</groupId>
<artifactId>spring-boot-autoconfigure</artifactId>
<version>1.3.0.M1</version>
</dependency>
```
(2) 属性配置(在application.properties中通过 hello.msg来指定)
```java
@ConfigurationProperties(prefix="hello")
public class HelloServiceProperties {
    private static final String MSG = "world";
    private String msg = MSG;

    //set get
}

```
(3)判断依据类
```java
    public class HelloService {
        private String msg;

        public String sayHello() {
            return "Hello" + msg;
        }

        public String getMsg() {
            return msg;
        }

        public void setMsg(String msg) {
            this.msg = msg;
        }
    }
```
(4)自动配置类
```java
@Configuration
@EnableConfigurationProperties(HelloServiceProperties.class)//开启属性注入，使用Autowired注入
@ConditionalOnClass(HelloService.class)//当HelloService在类路径下
@ConditionalOnProperty(prefix="hello", value="enabled", matchIfMissing=true)//当设置hello=enabled情况下，没有设置默认为true，即符合条件
public class HelloServiceAutoConfiguration {
    @Autowired
    private HelloServiceProperties hello;
    @Bean
    @ConditionalOnMissingBean(HelloService.class)  //当容器中没有这个bean的时候创建
    public HelloService helloService() {
        HelloService helloService = new HelloService();
        helloService.setMsg(helloServiceProperties.getMsg());
        return helloService;
    }
}
```

(5)注册配置
在resources下新建META-INF/spring.factories
填写注册内容
```xml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=包名.HelloServiceAutoConfiguration 
//若是多个配置可以用“,”隔开,  换行用"\"
```

(6)使用
在pom.xml中添加依赖
```xml
    //新建上面那个start pom 工程的信息
    <dependency>
        <groupId></groupId>
        <artifactId></artifactId>
        <version></version>
    </dependency>
```


```java
    //自动配置
    @Autowired
    HelloService helloService;
```

### spring boot web开发
#### 引入thymeleaf
```html
    <html xmlns="http://www.thymeleaf.org"></html>
```

#### @{}引用web静态资源
#### th：为前缀
#### 访问model中的数据 ${}
```html
    <span th:text="${singlePerson.name}"></span>
```

#### http -> https
```java
@Bean
public EmbeddedServletContainerFactory servletContainer() {
    TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory() {
        @Override
        protected void postProcessContext(Context context) {
            SecurityConstraint securityConstraint = new SecurityConstraint();
            securityConstraint.setUserConstraint("CONFIDENTIAL");
            SecurityCollection collection = new SecurityCollection();
            collection.addPattern("/*");
            securityConstraint.addCollection(collection);
            context.addConstraint(securityConstraint);
        } 
    }
    tomcat.addAdditionalTomcatConnectors(httpConnector());
    return tomcat;
}

@Bean 
public Connector httpConnector() {
    Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    connector.setScheme("http");
    connector.setPort(8080);
    connector.setSecure(false);
    connector.setRedirectPort(8443);
    return connector;
}
```

### JPA and  docker
#### docker常用命令
- docker search 镜像名 //检索镜像
- docker pull 镜像名 //下载镜像
- docker images //查看本地镜像列表
- docker rmi image-id //删除指定镜像
- docker rmi $(docker images -q) 删除所有镜像
- docker run --name container-name -d image-name 运行镜像  //--name 为容器取得名称, -d 表示detached，表示执行完这句命令后控制台将不会被阻碍，可以继续输入命令操作,image-name 是要使用哪个镜像来运行容器.
- 比如  docker run --name test-redis -d redis
- docker ps 查看运行中的容器列表
- docker ps -a 查看运行和停止的状态的容器
- docker stop container-name/container-id 停止容器
- 例如：docker stop test-redis 
- docker start container-name/containner-id 启动容器
- 例如：docker start test-redis
- 端口映射: docker run -d -p 6378:6379 --name port-redis redis //映射容器的6379端口到本机的6378端口
- docker logs container-name/container-id //查看当前容器日志。

#### JPA
- @EnableJpaRepositories("包名") 扫描该包下数据访问定义的接口
- 常规查询
```java
public interface PersonRepository extends JpaRepository<Person, Long> {
    //通过名字相等查询，参数为name, 在数据库中字段名也为name
    List<Person> findByName(String name);
    //通过名字like查询，参数为name, 在数据库中字段名也为name （模糊查询）
    List<Person> findByNameLike(String name);
    //通过名字和地址查询，参数为name和address
    List<Person> findByNameAndAddress(String name, String address);
}
```

-排序和分页
```java
public interface PersonRepository extends JpaRepository<Person, Long> {
    List<Person> findByName(String name, Sort sort); //排序
    Page<Person> findByName(String name, Pageable pageable);
}
```
-使用:
```java
List<Person> people = personRepository.findByName("xx", new Sort(Direction.ASC, "age"));

//page 接口可以获得当前页面的记录，总页数，总记录数，是否有上一页或者下一页.
Page<Person> people = personRepository.findByName("xx", new PageRequest(0, 10));
```

-自定义Repository实现
```java
@NoRepositoryBean // 指明当前这个接口不是我们的领域类的接口
public interface CustomRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID> { //实现pagingAndSortingRepository接口，具备分页和排序的能力.
    public void doSomething(ID id);//要定义的数据操作方法在接口中定义
}
```

-定义接口实现
```java
//实现上面定义的接口，继承SimpleJpaRepository类，让我们可以使用其提供的方法(如:findAll)
public class CustomRepositoryImpl<T, ID extends Serializable>  extends SimpleJpaRepository<T, ID> implements CustomRepository<T, ID> {
    private final EntityManager entityManager;//让数据操作方法中可以使用 entityManager
    public CustomRepositoryImpl(Class<T> domainClass, EntityManager entityManager) {
        super(domainClass, entityManager);
        this.entityManager = entityManager;
    }

    @Override
    public void  doSomething(ID id) {
        //定义数据访问操作

    }
}
```
-自定义RepositoryFactoryBean
```java
public class CustomRepositoryFactoryBean<T extends JpaRepository<S, ID>, S, ID extends Serializable> extends JpaRepositoryFactoryBean<T, S, ID> {//自定义RepositoryFactoryBean, 继承JpaRepositoryFactoryBean
    @Override
    protected RepositoryFactorySupport createRepositoryFactory(EntityManager entityManager) {
        return new CustomRepositoryFactory(entityManager);//重写此方法，用当前的CustomRepositoryFactory创建实例
    }

    private static class CustomRepositoryFactory extends JpaRepositoryFactory {//创建CustomRepositoryFactory ,继承JpaRepositoryFactory

        public CustomRepositoryFactory(EntityManager entityManager) {
            super(entityManager);
        }

        //获得当前自定义的Repository实现
        @Override
        @SuppressWarnings({"unchecked"})
        protected<T, ID extends Serializable> SimpleJpaRepository<?, ?> getTargetRepository(RepositoryInfomation information, EntityManager entityManager) {
            return new CustomRepositoryImpl<T, ID>((Class<T>) information.getDomationType(), entityManager);
        } 

        //获取当前自定义Repository实现的类型
        @Override
        protected Class<?> getRepositoryBaseClass(RepositoryMetadata metadata) {
            return CustomRepositoryImpl.class;
        }
    }
}
```
- 开启自定义支持使用@EnableJpaRepositories的repositoryFactoryBeanClass来指定FactoryBean
```java
@EnableJpaRepositories(repositoryFactoryBeanClass=CustomRepositoryFactoryBean.class)
```
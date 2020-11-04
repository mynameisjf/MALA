# MALA
基于SPRINGBOOT的开发框架

## 第1章 MALA介绍
### 1.1 概述
    mala开发框架（简称 Mala）是一个 基于springBoot 的自定义starter，封装了业务配置和一些常见的util方法，为简化开发、提高效率而生。
### 1.2 框架结构
![](https://s1.ax1x.com/2020/11/04/BcCnV1.md.png)

## 第2章 快速开始
    我们将通过一个简单的 Demo 来阐述 mala 的强大功能，在此之前，我们假设您已经:
    1. 拥有 Java 开发环境(1.8)以及相应 IDE
    2. 熟悉 Spring Boot
    3. 熟悉 Maven
现有一张 Demo 表，其表结构如下：
![](https://s1.ax1x.com/2020/11/04/BcCVKJ.png)

### 2.1 初始化工程
     创建一个空的 Maven或者Spring Boot(推荐)工程（工程将以 oracle 作为默认数据库进行演示）
     TIP:
     可以使用 Spring Initializr 快速初始化一个 Spring Boot 工程。

### 2.2 添加依赖
```maven
<dependency>
   <groupId>com.mammut.mala</groupId>
   <artifactId>mala-spring-boot-starter</artifactId>
   <version>${version}</version>
</dependency>
```

    以下为可选依赖。推荐引入
  ```maven
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-test</artifactId>
   <scope>test</scope>
</dependency>

<dependency>
   <groupId>org.projectlombok</groupId>
   <artifactId>lombok</artifactId>
   <optional>true</optional>
</dependency>
```
### 2.3 配置
#### 2.3.1 配置文件Application.yml配置
    在 resources/application.yml 配置文件(如果文件夹下不存在,则自行添加)中添加相关配置：以下为常用配置(前期不理解的时候，必填配置按照指定规则配置.可选配置按照个人实际情况配置。)，更多配置可以自行查阅springBoot文档.
##### 2.3.1.1 Mala框架配置
```yml
mala:
  config:
    validate: true                                                   #是否开启TOKEN验证。默认开启.开发阶段可以设置为关闭
    configUrl: http://192.168.110.202:8898/parkingManager/noAuth/    #获取用户相关的消息地址,一般为ZZJG
    swagger:                                                      # swagger配置
      swaggerBasePackage: com.mammut.parking.controller           # 扫描的路径
      swaggerTitle: 智慧停车                                       # 文档的标题
      swaggerDesc: 智慧停车。所有接口请求必带TOKEN。例如:http://ip:port/projectName?jws=xxxxx        # 文档的说明
      swaggerVer: 0.1                                          # 文档版本号
      openSwagger: true                                    #该请求不开启。则无法使用API文档。上线阶段必须取消该配置
```
##### 2.3.1.2 tomcat配置
    如果不配置，则默认请求URL:http://localhost:8080/
    是否必填:否
    配置含义:
    指定端口为9089.项目名默认为parking
    URL：http://localhost:9089/parking
```yml
server:
  port: 9089
  context-path: /parking
```
##### 2.3.1.3 兼容性配置
    是否必填:是
    配置含义: 见注释
```yml
spring:
  jmx:
    enabled: false  #解决多个WAR包放在同一个TOMCAT下，导致数据源初始化不正确的BUG
  cache:            #增加EHCACHE的缓存配置
    ehcache:
      config: classpath:config/ehcache.xml
```
##### 2.3.1.4 数据源配置
		目前提供2种用法，开发阶段和上线阶段。开发阶段为了方便调试。可以使用P6SPY。上线阶段必须切换为数据库连接池方式提供性能
p6spy配置如下:
```yml
spring:
  datasource:
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    url: jdbc:p6spy:oracle:thin:@127.0.0.1:1521/orcl
    username: username
    password: password
```

druid资源连接池配置:
```yml
spring:
  datasource:
    druid:
		url: jdbc:oracle:thin:@127.0.0.1:1521/orcl
		username: username
		password: password
		initial-size: 1
		max-active: 5
		min-idle: 1
		# 等待超时时间
		max-wait: 60000
		#打开PSCache，并且指定每个连接上PSCache的大小
		pool-prepared-statements: true
		max-pool-prepared-statement-per-connection-size: 20
		validation-query: SELECT 1 FROM DUAL
		test-on-borrow: false
		test-on-return: false
		test-while-idle: true
		#配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
		time-between-eviction-runs-millis: 60000
		#配置一个连接在池中最小生存的时间，单位是毫秒
		min-evictable-idle-time-millis: 300000
		filters: stat,wall,log4j
		stat-view-servlet:
		enabled: true
		url-pattern: /druid/*
		login-username: admin
		login-password: admin
		allow:
```
##### 2.3.1.5 编码集配置
```yml
	http:
		encoding:
		  charset: UTF-8
		  enabled: true
		  force: true
		  force-request: true
		  force-response: true
```
##### 2.3.1.6 页面默认图标
```yml
  #页面默认图标
  	mvc:
		favicon:
			enabled: true
```
##### 2.3.1.7 mybatis-plus配置
```yml
#mybatis-plus配置
mybatis-plus:
  mapper-locations: classpath:mapper/**/*Dao.xml
  # MyBaits 别名包扫描路径，通过该属性可以给包中的类注册别名，注册后在 Mapper 对应的 XML 文件中可以直接使用类名，而不用使用全限定的类名(即 XML 中调用的时候不用包含包名)
  type-aliases-package: com.mammut.*.entity
  global-config:
    db-config:
      #主键类型  0:"数据库Id自增", 1:"用户输入Id",2:"全局唯一Id (数字类型唯一Id)", 3:"全局唯一Id UUID";
      id-type: INPUT
      #大写命名,对表名和字段名均生效
      capital-mode: true
      #逻辑删除配置（下面2个配置）
      logic-delete-value: 0
      logic-not-delete-value: 1
```
#### 2.3.2 Spring Boot 启动类配置
```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@MapperScan("com.mammut.*.dao")  // mybatis的扫描路径
@EnableTransactionManagement     // 开启事务
@SpringBootApplication          // springBoot的配置
@EnableCaching                  // 开启缓存
@EnableSwagger2                 // 开启Swagger
public class ParkingApplication {

	public static void main(String[] args) {
		SpringApplication.run(ParkingApplication.class, args);
	}

}
```
### 2.4 编码
直接使用MyBatisPlusCodeGenerator.generator 一键生成通用代码。放到指定的目录下即可
```java
        // 文件生成路径
        String filePath = "C:\\test";
        // 需要生成的表名
        String modelName = "T_IP_DEMO";
        // 表名前缀，不配置这个，则生成的实体则为TIpEmployee，配置之后，生成的实体对象为Employee
        String TablePrefix = "T_IP_";
        // 生成的包名
        String packAgeName  = "com.mammut.demo";
        // 作者
        String author = "jiangf";
        // 是否使用REST风格.REST就是前后端分离的方式。会带SWAGGER。如果false,则不会显示SWAGGER。并且控制层不会标记REST
        boolean isRest = true;
        // 构造JDBC参数
        JdbcProperty jdbcProperty = new JdbcProperty(DbType.ORACLE);
        // 执行方法
        MyBatisPlusCodeGenerator.generator(filePath,
	          modelName,
			  TablePrefix,
			  packAgeName,
			  author,
			  isRest,
			  jdbcProperty);
```
### 2.5 开始使用
添加测试类，进行功能测试：
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class IDemoServiceTest {
    @Autowired
    private IDemoService demoService;

	@Test
    public void testList(){
        demoService.list();
    }

    @Test
    public void testSave(){
        Demo demo = new Demo();
        demo.setName("测试用户");
        demoService.save(demo);
    }


    @Test
    public void testDelete(){
        String demoId= "1";
        demoService.removeById(demoId);
    }


    @Test
    public void testUpadate(){
        demoService.list();
    }
}
```
控制台输出：  

![BcCZr9.png](https://s1.ax1x.com/2020/11/04/BcCZr9.png)
![BcCebR.md.png](https://s1.ax1x.com/2020/11/04/BcCebR.md.png)
### 2.6 注意事项及问题
+	basePackage的命名规范必须为com.mammut
+	springboot的启动类必须放在com.mammut目录下

# SpringCloud-Zuul
SpringCloud Zuul
---
&emsp;&emsp;**本文介绍的是SpringCloud（基于SpringBoot 2.X，Finchley.SR2版）中的路由网关SpringCloud Zuul的简单使用，戳[这里](https://github.com/butalways1121/SpringCloud-Zuul)下载源码。**
<!--more-->
## 一、SpringCloud Zuul简介
### 1.SpringCloud Zuul是什么
&emsp;&emsp;SpringCloud Zuul是Netflix设计用来为所有面向设备、web网站提供服务的所有应用的门面，Zuul可以提供动态路由、监控、弹性扩展、安全认证等服务，还可以根据需求将请求路由到多个应用中。**通俗一点来说，就是对服务提供一层保护，对外界的请求进行过滤转发到后端服务中。**

&emsp;&emsp;这里我们可以通过几张简单的示例图来进行了解，不使用路由网关的服务请求与响应示例图:

![](https://raw.githubusercontent.com/butalways1121/img-Blog/master/102.png)
使用路由网关的示例图：

![](https://raw.githubusercontent.com/butalways1121/img-Blog/master/103.png)
使用路由网关及注册中心的示例图:

![](https://raw.githubusercontent.com/butalways1121/img-Blog/master/104.png)
&emsp;&emsp;从上述图中，可以看出加了路由网关之后，实际上是将一对多的关系转变成了一对一的关系，这样的好处是可以在网关层进行数据合法校验、权限认证、负载均衡等统一处理，这样可以在很大的程度上节省的人力和物力，但是这种方式也有一定的弊端，就是以后新增了服务或者在服务中新增方法，就会使得网关层可能需要进行改动。幸好在SpringCloud有Zuul这个组件，通过Eureka将网关和微服务之间相互关联起来，所有的微服务都会在Eureka上进行注册，这样Zuul就能感知到哪些服务在线，并且可以通过配置路由规则将请求自动转发到指定的后端微服务上，这样即使后续新增了服务或者在服务中新增了某些方法，那么只需在Zuul层进行简单配置即可。
### 2.SpringCloud Zuul的功能
Zuul可以通过加载动态过滤机制，从而实现以下各项功能：
> **验证与安全保障：**识别面向各类资源的验证要求并拒绝那些与要求不符的请求；

> **审查与监控：**在边缘位置追踪有意义数据及统计结果，从而为我们带来准确的生产状态结论；

> **动态路由：**以动态方式根据需要将请求路由至不同后端集群处；

> **压力测试：**逐渐增加指向集群的负载流量，从而计算性能水平；

> **负载分配：**为每一种负载类型分配对应容量，并弃用超出限定值的请求；

> **静态响应处理：**在边缘位置直接建立部分响应，从而避免其流入内部集群；

> **多区域弹性：**跨越AWS区域进行请求路由，旨在实现ELB使用多样化并保证边缘位置与使用者尽可能接近。

### 3.为什么需要Zuul
* Zuul和Ribbon以及Eureka相结合，可以实现智能路由和负载均衡的功能，可以将流量按照某种策略分发到集群中的多个实例；
* 统一了对外暴露接口，外界系统不需要知道微服务系统中各服务之间调用的复杂性，也保护了内部微服务的api接口；
* 可以统一做用户身份认证，权限验证，这样就不用在每个微服务中进行认证了；
* 可以统一实现监控、日志的输出；
* 客户端请求多个微服务时，可以只请求Zuul一次，在Zuul中请求多个微服务，减少客户端和微服务的交互次数。
## 二、SpringCloud Zuul示例
本次项目同样分为注册中心、服务端、客户端，详细示例如下：
### 1.注册中心
新建springcloud-zuul-eureka项目作为注册中心，pom.xml依赖完整配置：
```bash
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>1.0.0</groupId>
	<artifactId>springcloud-zuul-eureka</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>
	<name>springcloud-eureka</name>
	<url>http://maven.apache.org</url>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.6.RELEASE</version>
		<relativePath />
	</parent>
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Finchley.SR2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```
application.properties配置信息：
```bash
spring.application.name=springcloud-zuul-eureka
server.port=8006
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
```
启动类代码：
```bash
@EnableEurekaServer
@SpringBootApplication
public class ZuulApplicaton 
{
    public static void main( String[] args )
    {
    	SpringApplication.run(ZuulApplicaton.class, args);
        System.out.println("zuul注册中心服务启动...");
    }
}
```
### 2.服务端
创建springcloud-zuul-gateway项目作为Zuul服务端，pom.xml文件配置：
```bash
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>1.0.0</groupId>
	<artifactId>springcloud-zuul-eureka</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>
	<name>springcloud-eureka</name>
	<url>http://maven.apache.org</url>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.6.RELEASE</version>
		<relativePath />
	</parent>
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Finchley.SR2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<!-- zuul依赖 -->
		<dependency>
        	<groupId>org.springframework.cloud</groupId>
        	<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    	</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```
application.properties文件:
```bash
spring.application.name=springcloud-zuul-gateway
server.port = 9009
eureka.client.serviceUrl.defaultZone=http://localhost:8006/eureka/
zuul.routes.hello.path = /hello/**
zuul.routes.hello.url = http://localhost:9010/hello
zuul.routes.hi.path = /hi/**
zuul.routes.hi.url = http://localhost:9011/hi
```
**其中`zuul.routes.{route}.path`和`uul.routes.{route}.url`是自定义路由的规则，通过path配置路径进行过滤，访问该路径会转发到url对应的地址，是属于一对一的配置方式。例如，如果访问`http://localhost:9009/hello/butalways`的话就会跳转到`http://localhost:9010/hello/butalways`地址。**
启动类代码示例：
```bash
@EnableDiscoveryClient
@SpringBootApplication
@EnableZuulProxy
public class ZuulGateway 
{
    public static void main( String[] args )
    {
    	SpringApplication.run(ZuulGateway.class, args);
        System.out.println("zuul 服务启动...");
    }
}
```
**其中，`@EnableZuulProxy`注解表示开启网关路由服务功能，是`@EnableZuulServer`注解的增强版，当Zuul与Eureka、Ribbon等组件配合使用时，我们使用`@EnableZuulProxy`。**
### 3.客户端
&emsp;&emsp;客户端这边需要创建两个服务来验证Zuul路由网关是否生效，新建两个项目springcloud-zuul-server1和springcloud-zuul-server2：
#### （1）springcloud-zuul-server1
pom.xml完整配置：
```bash
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>1.0.0</groupId>
	<artifactId>springcloud-zuul-eureka</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>
	<name>springcloud-eureka</name>
	<url>http://maven.apache.org</url>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.6.RELEASE</version>
		<relativePath />
	</parent>
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Finchley.SR2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```
application.properties配置信息：
```bash
spring.application.name=springcloud-zuul-server1
server.port=9010
eureka.client.serviceUrl.defaultZone=http://localhost:8006/eureka/
```
启动类：
```bash
@EnableDiscoveryClient
@SpringBootApplication
public class ZuulServer1 
{
    public static void main( String[] args )
    {
    	SpringApplication.run(ZuulServer1.class, args);
        System.out.println("zuul 第一个服务启动....");
    }
}
```
控制层代码示例：
```bash
@RestController
public class ConsumerController {

	@RequestMapping("/hello/{name}")
	public String index(@PathVariable String name) {
	    return name+",Hello World!";
	}
}
```
#### （2）springcloud-zuul-server2
pom.xml配置同springcloud-zuul-server1，application.properties配置信息：
```bash
spring.application.name=springcloud-zuul-server2
server.port=9011
eureka.client.serviceUrl.defaultZone=http://localhost:8006/eureka/
```
启动类：
```bash
@EnableDiscoveryClient
@SpringBootApplication
public class ZuulServer2 
{
    public static void main( String[] args )
    {
    	SpringApplication.run(ZuulServer2.class, args);
        System.out.println("zuul 第二个服务启动....");
    }
}
```
控制层：
```bash
@RestController
public class ConsumerController {

	@RequestMapping("/hi")
    public String index(@RequestParam String name) {
        return name+",hi!";
    }
}
```
**注:这里故意将两个客户端服务的接口参数请求和返回值设置成不一样，以便对Zull的功能进行多方面测试。**
### 4.测试
&emsp;&emsp;完成上述的代码开发后，就来测试springcloud-zuul是否可以实现地址过滤转发功能。首先依次启动springcloud-zuul-eureka、springcloud-zuul-gateway、springcloud-zuul-server1和springcloud-zuul-server2这四个项目，其中8006是注册中心springcloud-zuul-eureka的接口，9009是服务springcloud-zuul-gateway的端口，9010是第一个客户端springcloud-zuul-server1的端口，9011是第二个客户端springcloud-zuul-server2的端口。


启动成功之后，在浏览器中输入`http://localhost:9010/hello/butalways`，返回：
```
butalways,hello world!!
```
接着输入`http://localhost:9011/hi?name=butalways`，返回：
```
butalways,hi!
```
以上说明程序正常启动，并且两个客户端的接口返回正确。
***
接下来进行访问Zuul，将前两个访问地址的端口都改成服务端的即可，先在浏览器中请求`http://localhost:9009/hello/butalways`，返回：
```
butalways,Hello World!
```
再输入`http://localhost:9009/hi?name=butalways`，返回：
```
butalways,hi!
```
若结果如上，则说明zuul路由网关已经生效了，成功地将请求进行了转发！

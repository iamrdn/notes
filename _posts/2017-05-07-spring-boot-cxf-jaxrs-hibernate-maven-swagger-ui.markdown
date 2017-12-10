---
layout: post
title:  "Spring Boot, Apache CXF, Swagger under JAX-RS"
date:   2017-06-22 11:00:40 +0330
categories: spring-boot spring-dans apache-cxf restful jax-rs maven swagger-ui
---

This tutorial is about: Making a standard and enterprise JAX-RS Web Service with Spring Boot, Apache CXF, MySQL, and Swagger UI.

### Audience
Java Developers who know about Spring and REST standards.

### What is a Modular Application?
An Enterprise Software always will change after the version 1 (MVP) we should consider the software growth. It's absolutely better to separate the Tiers. In this case, I put Data Access and Service layers in [Spring-DANS][Spring-DANS] and RESTful Module as a part of Web Tier in another Module.

### Why used Spring Boot?
1. I'm not a fan of XML configuration and I like to configure Java Applications with Java Code.
2. Spring will manage the Application Server which is an Embedded Tomcat in Spring Boot and you don't need to be involved with an Application Server like Tomcat, JBoss, or etc.
3. Most of the Companies in the World using Spring Boot for creating RESTful Applications because it's easy to Develop and Deploy an Enterprise Application with Spring Boot.

### Why should we use Swagger UI on our Web Services?
Swagger UI is a very cool tool that can be placed on Back-End developers and Front-End developers. You just need to configure it with Spring Boot and see Swagger will generate some JSON, HTML, CSS, and JavaScript lines to show what Resources you have on your RESTful Web Service.

### What is the Killer Option of CXF rather than Spring Rest?
We have a standard in Java that called JAX-RS which is an API for making RESTful Web Services, and we have some implementations of this API like Jersey, Apache CXF, RESTEasy, and etc. It's highly recommended to develop RESUful Web Services with these standard Frameworks because Spring Framework has its own implementation of REST which is completely different than standard ones.

### And why used Jackson not Gson?
It's compatible with all JAX-RS frameworks like Jersey, Apache CXF, RESTEasy, Restlet, and Spring framework and the another thing is that [since Spring 4.x Jackson support has been improved][since-Spring-4.x-Jackson-support-has-been-improved] That means you just need to put Jackson dependency in your project then it will automatically do the Parsing.

### Step 1: Add Maven Dependencies to our project
It's the time to create a Maven Project and put the below dependencies on your POM.xml file.
{% highlight xml %}
  <properties>
    <spring-boot.version>1.5.3.RELEASE</spring-boot.version>
    <spring-dans.version>0.0.1</spring-dans.version>
    <tomcat-dbcp.version>8.5.14</tomcat-dbcp.version>
    <mysql-driver.version>6.0.6</mysql-driver.version>
    <cxf.version>3.1.11</cxf.version>
    <swagger-ui.version>3.0.7</swagger-ui.version>
    <jackson.version>2.8.8</jackson.version>
    <junit.version>4.12</junit.version>
  </properties>
  <dependencies>
    <!-- spring dependencies -->
    <dependency>
      <groupId>com.massoudafrashteh.code.spring.dans</groupId>
      <artifactId>spring-dans</artifactId>
      <version>${spring-dans.version}</version>
      </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>${spring-boot.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
      <version>${spring-boot.version}</version>
    </dependency>
    <!-- Tomcat Connection Pooling - used for cache the Database Connection -->
    <dependency>
      <groupId>org.apache.tomcat</groupId>
      <artifactId>tomcat-dbcp</artifactId>
      <version>${tomcat-dbcp.version}</version>
    </dependency>
    <!-- MySQL Database Connector -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>${mysql-driver.version}</version>
    </dependency>
    <!-- Apache CXF dependencies -->
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-spring-boot-starter-jaxrs</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-rt-rs-service-description-swagger</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <dependency>
      <groupId>org.webjars</groupId>
      <artifactId>swagger-ui</artifactId>
      <version>${swagger-ui.version}</version>
    </dependency>
    <!-- Jackson dependency - used for convert Java Objects to JSON and vice versa -->
    <dependency>
      <groupId>com.fasterxml.jackson.jaxrs</groupId>
      <artifactId>jackson-jaxrs-json-provider</artifactId>
      <version>${jackson.version}</version>
    </dependency>
    <!-- unit testing dependencies -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>${junit.version}</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
{% endhighlight %}

### Step 2: Database configurations
It's so easy to configure Database connection in Spring Boot the only thing that you need is that put your configurations on src/main/resources/application.properties
{% highlight properties %}
# Datasource
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/spring_cxf?createDatabaseIfNotExist=true
spring.datasource.username=username # Never use root user in production
spring.datasource.password=password

# Hibernate
spring.jpa.hibernate.ddl-auto=create # Change if after first run to "update", will drop your tables every time otherwise
spring.jpa.properties.hibernate.show_sql=true
{% endhighlight %}

[Spring Boot uses Tomcat Pooling][Spring-Boot-uses-Tomcat-Pooling] by default which is the best Pooling software in Java. 

### Step 3: Configuration of Swagger UI
{% highlight java %}
@Configuration
@EnableAutoConfiguration
@ComponentScan(basePackages = "com")
@EntityScan(basePackages = "com.massoudafrashteh.code.spring.dans.domain")
@EnableJpaRepositories(basePackages = "com.massoudafrashteh.code.spring.dans.repository")
public class CxfConfig {

  @Autowired
  private Bus bus;

  @Bean
  public Server rsServer() {
    final JAXRSServerFactoryBean endpoint = new JAXRSServerFactoryBean();
    endpoint.setProvider(new JacksonJsonProvider());
    endpoint.setProvider(getExceptionHandler());
    endpoint.setBus(bus);
    endpoint.setAddress("/");
    endpoint.setServiceBeans(Arrays.<Object>asList(indexController(), userController()));
    endpoint.setFeatures(Arrays.asList(new Swagger2Feature()));
    return endpoint.create();
  }

  @Bean
  public ExceptionHandler getExceptionHandler() {
    return new ExceptionHandler();
  }

  @Bean
  public IndexController indexController() {
    return new IndexController();
  }

  @Bean
  public UserController userController() {
    return new UserController();
  }
{% endhighlight %}
The default address of API is **/services** to [change the default CXF API address][change-the-default-CXF-API-address] from */services* to */api* (or anything that you like) just check my Github profile :)  

### Step 4: Spring Boot starter method
I prefer to keep Spring Boot starter method out of other configuration classes and put your Beans on other Classes.

{% highlight java %}
@SpringBootApplication
public class Starter {
  public static void main(final String[] args) {
    SpringApplication.run(CxfConfig.class, args);
  }
}
{% endhighlight %}

### Step 5: Run Spring Boot
It's time to see what we have done, we can just run a Boot project with **mvn spring-boot:run** or use Eclipse or any other IDE that you like. If your Application Is running on PORT 8080 just open **http://localhost:8080/services/services** then you will see your API links and click the Swagger Link.

{% include image.html url="/images/spring-boot-cxf-swagger-ui.png" description="Final view of running Swagger UI with Spring Boot and Apache CXF" %}

### Any problem?
Just [git clone Spring CXF project][git-clone-Spring-CXF-project] and run it on your Machine.

Good Luck :)

[Spring-DANS]: https://github.com/massoudAfrashteh/code/tree/master/java/spring-maven-modules/spring-dans
[since-Spring-4.x-Jackson-support-has-been-improved]: https://spring.io/blog/2014/12/02/latest-jackson-integration-improvements-in-spring
[Spring-Boot-uses-Tomcat-Pooling]: https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-sql.html#boot-features-connect-to-production-database
[change-the-default-CXF-API-address]: https://github.com/massoudAfrashteh/code/blob/master/java/spring-maven-modules/spring-boot-cxf/restful/src/main/java/starter/CxfConfig.java
[git-clone-Spring-CXF-project]: https://github.com/massoudAfrashteh/code/tree/master/java/spring-maven-modules/spring-boot-cxf

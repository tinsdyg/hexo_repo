---
title: SSM 整合笔记
tags: study
---
## 1.依赖导入

使用Maven进行导入，在pom.xml下添加以下依赖

```xml-dtd

<dependencies>

        <!-- 包含了许多spring常用依赖 -->

        <dependency>

            <groupId>org.springframework</groupId>

            <artifactId>spring-context</artifactId>

            <version>5.3.23</version>

        </dependency>


        <!-- SpringMvc -->

        <dependency>

            <groupId>org.springframework</groupId>

            <artifactId>spring-web</artifactId>

            <version>5.3.23</version>

        </dependency>

        <dependency>

            <groupId>org.springframework</groupId>

            <artifactId>spring-webmvc</artifactId>

            <version>5.3.23</version>

        </dependency>


        <!-- Spring JDBC -->

        <dependency>

            <groupId>org.springframework</groupId>

            <artifactId>spring-jdbc</artifactId>

            <version>5.3.23</version>

        </dependency>

  

        <!-- Mybatis -->

        <dependency>

            <groupId>org.mybatis</groupId>

            <artifactId>mybatis</artifactId>

            <version>3.5.11</version>

        </dependency>

  

        <!-- Mybatis和Spring整合的依赖 -->

        <dependency>

            <groupId>org.mybatis</groupId>

            <artifactId>mybatis-spring</artifactId>

            <version>2.0.7</version>

        </dependency>

  

        <!-- fastjson springmvc实现RestController的原理就是依赖jackson实现 -->

        <dependency>

            <groupId>com.fasterxml.jackson.core</groupId>

            <artifactId>jackson-databind</artifactId>

            <version>2.14.0-rc2</version>

        </dependency>

  

        <!-- Mysql驱动 -->

        <dependency>

            <groupId>mysql</groupId>

            <artifactId>mysql-connector-java</artifactId>

            <version>8.0.30</version>

        </dependency>

  

        <!-- Druid数据源 -->

        <dependency>

            <groupId>com.alibaba</groupId>

            <artifactId>druid</artifactId>

            <version>1.2.14</version>

        </dependency>

  

        <!-- Lombok 免于写实体类的getter和setter等常用方法 -->

        <dependency>

            <groupId>org.projectlombok</groupId>

            <artifactId>lombok</artifactId>

            <version>1.18.24</version>

        </dependency>

  

        <!-- Servlet -->

        <dependency>

            <groupId>javax.servlet</groupId>

            <artifactId>servlet-api</artifactId>

            <version>2.5</version>

        </dependency>

  

    </dependencies>

```

## 2.配置Spring及Mybatis

在resources文件夹下创建applicationContext.xml (Spring配置) 、database.properties (数据库连接信息)

```xml-dtd

    <!-- 扫描Service -->

    <context:component-scan base-package="com.demo.service"/>

  

    <!-- 加载数据库连接配置文件 -->

    <context:property-placeholder location="classpath:database.properties"/>

    <!-- 配置Druid数据源 -->

    <bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">

        <property name="url" value="${jdbc.url}"/>

        <property name="driverClassName" value="${jdbc.drivername}"/>

        <property name="username" value="${jdbc.username}"/>

        <property name="password" value="${jdbc.password}"/>

    </bean>

    <!-- Mybatis配置 -->

    <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">

        <property name="dataSource" ref="dataSource"/>

        <property name="typeAliasesPackage" value="com.demo.entity.Student"/>

        <!--扫描 *Mapper.xml

        <property name="mapperLocations" value="classpath:mapperxmlLocation"/>

        -->

    </bean>

    <!-- Mybatis 扫描Mapper -->

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer" id="mapperScannerConfigurer">

        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>

        <property name="basePackage" value="com.demo.mapper"/>

    </bean>

```

## 3.SpringMvc配置

在resources目录下创建spring-mvc.xml


```xml-dtd

    <!-- 扫描控制器Controller -->

    <context:component-scan base-package="com.demo.controller"/>

    <!-- 启用驱动注解 -->

    <mvc:annotation-driven/>

```

## 4.配置web.xml

```xml-dtd

    <!-- Spring配置文件导入 -->

    <context-param>

        <param-name>contextConfigLocation</param-name>

        <param-value>classpath:applicationContext.xml</param-value>

    </context-param>

    <!-- 添加监听器，在web项目驱动时加载 -->

    <listener>

        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>

    </listener>

    <!-- SpringMvc 配置文件导入 -->

    <servlet>

        <servlet-name>dispatcherServlet</servlet-name>

        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <init-param>

            <param-name>contextConfigLocation</param-name>

            <param-value>classpath:spring-mvc.xml</param-value>

        </init-param>

    </servlet>

    <!-- 将所有请求交给SpringMvc处理 -->

    <servlet-mapping>

        <servlet-name>dispatcherServlet</servlet-name>

        <url-pattern>/</url-pattern>

    </servlet-mapping>

    <!-- 添加编码过滤器，强制把请求和响应编码设置为UTF-8 -->

    <filter>

        <filter-name>encodingFilter</filter-name>

        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

        <init-param>

            <param-name>encoding</param-name>

            <param-value>UTF-8</param-value>

        </init-param>

        <init-param>

            <param-name>forceRequestEncoding</param-name>

            <param-value>true</param-value>

        </init-param>

        <init-param>

            <param-name>forceResponseEncoding</param-name>

            <param-value>true</param-value>

        </init-param>

    </filter>

    <filter-mapping>

        <filter-name>encodingFilter</filter-name>

        <url-pattern>/*</url-pattern>

    </filter-mapping>

```

至此，达成了SSM的简单整合。

## Tip1：*Mapper.xml文件扫描问题

由于Maven项目在编译中会忽略java目录下的xml文件，因此需要手动设置Maven使其不扫描指定文件

```xml-dtd

<!-- pom.xml文件末尾添加 -->

<build>

    <resources>

        <resource>

            <directory>src/main/java</directory>

            <includes>

                <include>**/*.xml</include>

            </includes>

            <filtering>false</filtering>

        </resource>

    </resources>

</build>

```

或者在resources目录下创建和mapper文件相同目录放置*Mapper.xml文件 


## Tip2: Maven添加仓库

```xml-dtd

<!--    阿里云搭建的国内镜像http://maven.aliyun.com，跑起来速度很快，可以进行配置-->

    <repositories>

        <repository>

            <id>nexus-aliyun</id>

            <name>nexus-aliyun</name>

            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>

            <releases>

                <enabled>true</enabled>

            </releases>

            <snapshots>

                <enabled>false</enabled>

            </snapshots>

        </repository>

    </repositories>

```
# LKHmovie_admin 프로젝트
<span style="color:red">추후 확인해봐야 할것</span>
- 4) 처음 서버 시작 시 로그가 두번씩 중복으로 출력 됨
- 4) 로깅 설정 이후 src/test/java에서 junit 테스트 getConnection에서 에러 남
- 4) 쿼리 로그 출력 되는지는 추후에 기능 구현 해봐야 확인이 됨

---

## 1. 프로젝트 생성
- help > Eclipse MarketPlace > sts 검색 > Spring Tools 3 머시기 release 설치
- new > spring legacy project > project name 입력 > templates : spring MVC project > next > top-level package : "com.SampleBoard.khlim" > finish > 프로젝트 생성<br><br>

---

## 2. 프로젝트 설정 변경
- project 우클릭 > properties 
    - project facets > java : 1.8
    - runtimes : apache tomcat v8.0
    - java build path > libraries > 톰캣 및 자바 버전 확인

- pom.xml에서 `<org.springframework-version>`태그에 스프링 최신버전 입력 (이 당시껀 4.3.25)

- WEB-INF 폴더 하위의 spring 폴더 삭제
    - root-context.xml과 servlet-context.xml 들어있음

- root-context.xml > applicationContext.xml 대체
  - src/main/resources 우클릭 > new > other > spring > spring bean configuration file > file name : applicationContext.xml
  - 네임스페이스에서 `context` 체크 후 자동 스캔 추가
  - src/main/java에 `com.LKHmovie.biz`패키지 생성
```xml
<!-- applicationContext.xml -->

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

	<!-- 자동 스캔 추가 -->
	<context:component-scan base-package="com.LKHmovie.biz"></context:component-scan>

</beans>
```

- servlet-context.xml > servlet-config.xml 대체
  - WEB-INF폴더 하위에 config 폴더 생성 후 우클릭 > new > other > spring > spring
bean configuration file > file name : servlet-config.xml 
  - 네임스페이스에서 `context` 체크 후 자동 스캔 추가
  - src/main/java에 `com.LKHmovie.controller`패키지 생성
  - resolver 정보 추가
```xml
<!-- servlet-config.xml -->

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

	<!-- 자동스캔 추가 -->
	<context:component-scan base-package="com.LKHmovie.controller"></context:component-scan>

	<!-- resolver 설정 추가 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
</beans>
```

- web.xml 수정
  - 변경된 root-context와 servlet-context 경로 수정
  - url패턴 수정
  - 인코딩  설정 추가
```xml
<!-- web.xml -->

<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

	<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</context-param>
	
	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!-- Processes application requests -->
	<servlet>
		<servlet-name>dispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/config/servlet-config.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
		
	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>
	
	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

</web-app>
```

---

## 3. DB 연동 및 설정
- Maven Dependencies 추가
```xml
<!-- pom.xml -->

<!-- MyBatis -->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.5.2</version>
</dependency>

<!-- MyBatis-Spring -->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>2.0.2</version>
</dependency>

<!-- apache commons dbcp -->
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-dbcp2</artifactId>
	<version>2.5.0</version>
</dependency>

<!-- mysql -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.46</version>
</dependency>

<!-- spring-jdbc -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>${org.springframework-version}</version>
</dependency>

<!-- spring-test -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
	<version>${org.springframework-version}</version>
</dependency>
```

- DB 정보 파일 생성
  - src/main/resources에 config 폴더 생성
  - database.properties 파일 생성
```
(db.properties)

jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3307/movie?useSSL=false
jdbc.username=root
jdbc.password=[자기가 설정한 비밀번호]
```

- applicationContext에 DB 정보 설정 입력
```xml
<!-- applicationContext.xml -->

<!-- Database info -->
<context:property-placeholder location="classpath:config/database.properties"/>

<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
	<property name="driverClassName" value="${jdbc.driver}"></property>
	<property name="url" value="${jdbc.url}"></property>
	<property name="username" value="${jdbc.username}"></property>
	<property name="password" value="${jdbc.password}"></property>
</bean>
```

- DB 연동 테스트 파일 작성
  - src/test/java의 com.LKHmovie.admin에 DataSourceTest.java 파일 생성
  - 우클릭 > Run as > JUnit test 하면 연결 성공 시 아래와 같은 문구 나옴
    - `1229161065, URL=jdbc:mysql://127.0.0.1:3307/movie?useSSL=false, UserName=root@localhost, MySQL Connector Java`
```java
// DataSourceTest.java

package com.LKHmovie.admin;

import java.sql.Connection;

import javax.inject.Inject;
import javax.sql.DataSource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("file:src/main/resources/applicationContext.xml")

public class DataSourceTest {

	@Inject
	private DataSource ds;
	
	@Test
	public void testConnection() throws Exception {
		try (Connection con = ds.getConnection()) {
			System.out.println(con);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```



- 웹상에서 DB 연동 확인
  - (DB 연동 확인 구별을 위해 예제 화면도 조금 수정)
  - servlet-config에 `<context:component-scan base-package="com.LKHmovie.admin"></context:component-scan>` 추가
  - HomeController.java에 DB 연동 테스트 함수 추가
  - home.jsp 복사해서 home_dbTest.jsp 파일 생성
```java
// HomeController.java

package com.LKHmovie.admin;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.text.DateFormat;
import java.util.Date;
import java.util.Locale;

import org.apache.commons.dbcp2.BasicDataSource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

/**
 * Handles requests for the application home page.
 */
@Controller
public class HomeController {
	
	private static final Logger logger = LoggerFactory.getLogger(HomeController.class);
	
	@Autowired
	BasicDataSource dataSource;
	
	/**
	 * Simply selects the home view to render by returning its name.
	 */
	@RequestMapping(value = "/home.do", method = RequestMethod.GET)
	public String home(Locale locale, Model model) {
		logger.info("Welcome home! The client locale is {}.", locale);
		
		Date date = new Date();
		DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);
		
		String formattedDate = dateFormat.format(date);
		
		model.addAttribute("serverTime", formattedDate );
		
		return "home";
	}
	
	@RequestMapping(value = "/dbTest.do", method = RequestMethod.GET)
	public String dbTest(Model model) {
		Connection conn = null;
		Statement st = null;
		
		try {
			conn = dataSource.getConnection();
			st = conn.createStatement();
			ResultSet rs = st.executeQuery("select now() as now;");
			
			while(rs.next() ) {
				model.addAttribute("dbTime", rs.getString("now"));
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if(st != null) st.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
		
		return "home_dbTest";
	}
}
```

```html
<!-- home_dbTest.jsp -->

<%@ page contentType="text/html; charset=UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
<head>
	<title>Home</title>
</head>
<body>
<h1>
	Hello world!  
</h1>

<p>  The time on the database time is ${dbTime } </p>

</body>
</html>
```

- 매핑 설정
  - help > eclipse market palce > orm 검색 > java orm plugin for eclipse beta 설치
  - 프로젝트 우클릭 > new > other > java orm plugin > mybatis configuration xml > 
    - contatiner : 해당 프로젝트 선택
    - mybatis configuration name : sql-map-config.xml 입력
  - src폴더에 생성된 db.properties는 삭제, sql-map-config.xml은 src/main/resource로 이동
```xml
<!-- sql-map-config.xml -->

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

	<typeAliases>
		<!-- <typeAlias type="user" alias="com.LKHmovie.biz.user.UserVO"></typeAlias> -->
	</typeAliases>

	<mappers>
		<!-- <mapper resource="mappings/useres-mapping.xml" /> -->
	</mappers>
</configuration>
```
  - 프로젝트 우클릭 > new > other > java orm plugin > mybatis mapper xml > 
    - container : 해당 프로젝트 선택
    - mybatis mapper name : [기능명]-mapping.xml 입력
  - 생성된 파일을 src/main/resources의 mappings 패키지 생성 후 이동
```xml
<!-- [기능명]-mapping.xml -->

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- <mapper namespace="UseresDAO">

	<select/insert/update/delete id="getUserInfo" resultType="user">
		<![CDATA[
			SELECT 
				* 
			FROM 
				USERES 
			WHERE 
				USERID=#{userId};
		]]>
	</select>
</mapper> -->
<mapper namespace="">

</mapper>
```
- Spring, JDBC 설정
  - JDBC, session 관련 설정 추가
```xml
<!-- applicationContext.xml -->

<!-- ~~~ -->
	<!-- Spring JDBC 설정 -->
	<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<!-- Spring과 Mybatis 연동 설정  -->
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
		<property name="configLocation" value="classpath:sql-map-config.xml"></property>
	</bean>
	
	<bean class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg ref="sqlSession"></constructor-arg>
	</bean>
```

---

## 4. 로깅 설정 (logback)
- dependency 추가
  - slf4j-version은 1.7.25 이후 버전으로 수정
    - logback dependency의 버전과 호환 문제가 있는것 같음
    - NoClassDefFoundError: org/slf4j/event/LoggingEvent : Logback-classic version 1.1.4 and later require slf4j-api version 1.7.15 or later
  - Class path contains multiple SLF4J bindings 도 로그 중엔 나오지만 이건 딱히 지장은 없는 듯
    - 거슬린다면 artifactId : slf4j-log4j12인거 주석처리하면 해결 되기는 함
```xml
<!-- pom.xml -->

<!-- ~~~ -->
<properties>
	<java-version>1.8</java-version>
	<org.springframework-version>4.3.25.RELEASE</org.springframework-version>
	<org.aspectj-version>1.6.10</org.aspectj-version>
	<org.slf4j-version>1.7.25</org.slf4j-version>
</properties>
<!-- ~~~ -->

	<!-- ~~~ -->
	<!-- Logging -->
	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-classic</artifactId>
		<version>1.2.3</version>
	</dependency>
	<dependency>
		<groupId>org.lazyluke</groupId>
		<artifactId>log4jdbc-remix</artifactId>
		<version>0.2.7</version>
	</dependency>
	<!-- ~~~ -->
```

- 로깅 파일 작성
  - src/main/resources의 log4j.xml 파일 삭제 후 logback.xml 파일 생성
  - 파일로 로그 관리하려면 '<!-- <appender-ref ref="file" /> -->' 주석 해제 하면 됨
```xml
<!-- logback.xml -->

<?xml version="1.0" encoding="UTF-8"?>

<configuration>
	<property name="LOG_DIR" value="C:/eng/log/LKHmovie_admin"></property>

	<!-- Appenders -->
	<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<charset>UTF-8</charset>
			<Pattern>
				%d{HH:mm:ss.SSS} [%thread] %-5level %logger{5} - %msg%n
			</Pattern>
		</encoder>
	</appender>
	
	<appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${LOG_DIR}/LKHmovie_admin.log</file>
		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<Pattern>
				%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{5} - %msg%n
			</Pattern>
		</encoder>
		
		<!-- 일자에 따른 로그파일 처리 -->
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<fileNamePattern>${LOG_DIR}/LKHmovie_admin-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
			<maxFileSize>10MB</maxFileSize>
			<maxHistory>30</maxHistory>
		</rollingPolicy>
		
		<!-- 용량에 따른 로그파일 처리 -->
		<!-- <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
			<MaxFileSize>10MB</MaxFileSize>
		</triggeringPolicy>
		
		<rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
			<FileNamePattern>${LOG_DIR}/LKHmovie_admin.%i.log.zip</FileNamePattern>
			<MinIndex>1</MinIndex>
			<MaxIndex>10</MaxIndex>
		</rollingPolicy> -->
	</appender>
	
	<!-- Application Loggers -->
	<logger name="com.LKHmovie" level="info" additivity="false">
		<appender-ref ref="console" />
		<!-- <appender-ref ref="file" /> -->
	</logger>
	
	<!-- Query Loggers -->
	<logger name="jdbc.sqlonly" level="info" additivity="false">
		<appender-ref ref="console" />
		<!-- <appender-ref ref="file" /> -->
	</logger>
	
	<logger name="jdbc.resultsettable" level="info" additivity="false">
		<appender-ref ref="console" />
		<!-- <appender-ref ref="file" /> -->
	</logger>

	<!-- 3rdparty Loggers -->
	<logger name="org.springframework.core" level="info">
		<appender-ref ref="console" />
		<!-- <appender-ref ref="file" /> -->
	</logger>
	
	<logger name="org.springframework.beans" level="info">
		<appender-ref ref="console" />
		<!-- <appender-ref ref="file" /> -->
	</logger>
	
	<logger name="org.springframework.context" level="info">
		<appender-ref ref="console" />
		<!-- <appender-ref ref="file" /> -->
	</logger>

	<logger name="org.springframework.web" level="info">
		<appender-ref ref="console" />
		<!-- <appender-ref ref="file" /> -->
	</logger>

	<!-- Root Logger -->
	<root level="warn">
		<appender-ref ref="console" />
		<!-- <appender-ref ref="file" /> -->
	</root>
	
</configuration>
```

- root-context에 로깅 설정 추가
```xml
<!-- applicationContext.xml -->

<!-- ~~~ -->
<context:property-placeholder location="classpath:config/database.properties"/>

<bean id="dataSourceSpied" class="org.apache.commons.dbcp2.BasicDataSource">
	<property name="driverClassName" value="${jdbc.driver}"></property>
	<property name="url" value="${jdbc.url}"></property>
	<property name="username" value="${jdbc.username}"></property>
	<property name="password" value="${jdbc.password}"></property>
</bean>

<bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
	<constructor-arg ref="dataSourceSpied"></constructor-arg>
	<property name="logFormatter">
		<bean class="net.sf.log4jdbc.tools.Log4JdbcCustomFormatter">
			<property name="loggingType" value="MULTI_LINE"></property>
			<property name="sqlPrefix" value="SQL : "></property>
		</bean>
	</property>
</bean>

<!-- Spring JDBC 설정 -->
<!-- ~~~ -->
```

- logger 템플릿 지정
  - windows > preferences > java > editor > templates > new
    -  name : LOGGER
    -  context : java
    -  pattern : private static final Logger LOGGER = LoggerFactory.getLogger(${primary_type_name}.class); ${imp:import(org.slf4j.Logger, org.slf4j.LoggerFactory)}
       - LOGGER.info(), LOGGER.error() 등 사용 가능
  - HomeController 파일로 테스트
```java
// HomeController.java

// LOGGER 위주로 수정 됨!!
// ~~~
private static final Logger LOGGER = LoggerFactory.getLogger(HomeController.class);

@Autowired
BasicDataSource dataSource;

/**
	* Simply selects the home view to render by returning its name.
	*/
@RequestMapping(value = "/home.do", method = RequestMethod.GET)
public String home(Locale locale, Model model) {
	LOGGER.info("Welcome home! The client locale is {}.", locale);
	
	Date date = new Date();
	DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);
	
	String formattedDate = dateFormat.format(date);
	
	model.addAttribute("serverTime", formattedDate );
	
	return "home";
}

@RequestMapping(value = "/dbTest.do", method = RequestMethod.GET)
public String dbTest(Model model) {
	Connection conn = null;
	Statement st = null;
	
	try {
		conn = dataSource.getConnection();
		st = conn.createStatement();
		ResultSet rs = st.executeQuery("SELECT ID FROM ADMIN WHERE ID='ROOT';");
		
		LOGGER.info("logger test");
		
		while(rs.next() ) {
		// ~~~
```


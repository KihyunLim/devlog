# LKHmovie_admin 프로젝트
<span style="color:red">추후 확인해봐야 할것</span>
- 4) 처음 서버 시작 시 로그가 두번씩 중복으로 출력 됨
- 4) 로깅 설정 이후 src/test/java에서 junit 테스트 getConnection에서 에러 남
  - dataSourceSpied, dataSource 이 두개가 NoUniqueBeanDefinitionException 에러남
  - 참고 : https://www.lesstif.com/pages/viewpage.action?pageId=20774954

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

	<!-- HandlerMapping, HandlerAdapter를 등록 -->
	<mvc:annotation-driven></mvc:annotation-driven>

	<!-- resolver 설정 추가 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
</beans>
```

- web.xml 수정
  - 변경된 root-context와 servlet-context 경로 수정
  - url패턴 설정
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
		<url-pattern>/</url-pattern>
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

---

## 5. test MVC 작성
- VO 작성
  - 사용할 변수 작성 후 `ctrl + shift + s`로 getter/setter 및 toString() 생성
```java
// TestListVO

package com.LKHmovie.biz.test.list;

import java.text.SimpleDateFormat;
import java.util.Date;

public class TestListVO {

	private String id;
	private String password;
	private String name;
	private Date createDate;
	private String phone;
	
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getCreateDate() {
		SimpleDateFormat simpleCreateDate = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		return simpleCreateDate.format(createDate);
	}
	public void setCreateDate(Date createDate) {
		this.createDate = createDate;
	}
	public String getPhone() {
		return phone;
	}
	public void setPhone(String phone) {
		this.phone = phone;
	}
	
	@Override
	public String toString() {
		return "ListVO [id=" + id + ", password=" + password + ", name=" + name + ", createDate=" + createDate
				+ ", phone=" + phone + "]";
	}
}
```

- sql mapper 추가
  - 테스트용 typeAlias와 mapper 추가
```xml
<!-- sql-map-config.xml -->

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

	<typeAliases>
		<typeAlias alias="testList" type="com.LKHmovie.biz.test.list.TestListVO"></typeAlias>
	</typeAliases>

	<mappers>
		<mapper resource="mappings/test-mapping.xml" />
	</mappers>
</configuration>
```

- sql mapping 파일 추가
  - src/main/resources/mappings에 test-mapping.xml 추가
```xml
<!-- test-mapping.xml -->

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  
<mapper namespace="TestListDAO">
	<select id="getTestList" resultType="testList">
		SELECT * FROM USERLIST;
	</select>
</mapper>
```

- DAO 추가
  - src/main/java/com.LKHmovie.biz.test.list.impl에 TestListDAOMybatis.java 추가
```java
// TestListDAOMybatis.java

package com.LKHmovie.biz.test.list.impl;

import java.util.List;

import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import com.LKHmovie.biz.test.list.TestListVO;

@Repository
public class TestListDAOMybatis {

	@Autowired
	private SqlSessionTemplate mybatis;
	
	public List<TestListVO> getTestList() {
		return mybatis.selectList("TestListDAO.getTestList");
	}
}
```

- service(interface) 추가
  - src/main/java/com.LKHmovie.biz.test.list에 TestListService.java 추가
```java
// TestListService

package com.LKHmovie.biz.test.list;

import java.util.List;

public interface TestListService {

	List<TestListVO> getTestList();

}
```

- serviceImpl 추가
  - src/main/java/com.LKHmovie.biz.test.list.impl에 TestListServiceImpl.java 추가
```java
// TestListServiceImpl.java

package com.LKHmovie.biz.test.list.impl;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.LKHmovie.biz.test.list.TestListService;
import com.LKHmovie.biz.test.list.TestListVO;

@Service("testListService")
public class TestListServiceImpl implements TestListService{

	@Autowired
	private TestListDAOMybatis listDAO;
	
	public List<TestListVO> getTestList() {
		return listDAO.getTestList();
	}
}
```

- controller 추가
  - src/main/java/com.LKHmovie.controller.test에 GetTestListController.java 추가
```java
// GetTestListController.java

package com.LKHmovie.controller.test;

import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import com.LKHmovie.biz.test.list.TestListService;
import com.LKHmovie.biz.test.list.TestListVO;

@Controller
public class GetTestListController {
	private static final Logger LOGGER = LoggerFactory.getLogger(GetTestListController.class);
	
	@Autowired
	private TestListService testListService;
	
	@RequestMapping(value="/getTestList.do")
	public String getTestList(Model model) {
		try {
			List<TestListVO> testList = testListService.getTestList();
			
			model.addAttribute("testList", testList);
		} catch(Exception e) {
			LOGGER.error("error message : " + e.getMessage());
			LOGGER.error("error trace : ", e);
		}
		
		return "home_getListTest";
	}
}
```

- 테스트용 웹페이지 추가
  - src/main/webapp/WEB-INF/views에 home_getListTest.jsp 추가
```jsp
<!-- home_getListTest.jsp -->

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

<c:forEach items="${testList }" var="test">
	<p>${test.name }</p>
</c:forEach>

</body>
</html>
```

---

## 6. AOP 추가
- dependency 추가
```xml
<!-- pom.xml -->

		<!-- ~~~ -->
		<!-- AspectJ -->
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjrt</artifactId>
			<version>${org.aspectj-version}</version>
		</dependency>	
		<dependency>
		    <groupId>org.aspectj</groupId>
		    <artifactId>aspectjweaver</artifactId>
		    <version>1.9.4</version>
		</dependency>
		
		<!-- Logging -->
		<!-- ~~~ -->
```

- AOP 설정 등록
  - applicationContext.xml에서 Namespaces 탭 클릭 > aop 선택 후 저장
```xml
<!-- applicationContext.xml -->

	<!-- ~~~ -->
	<!-- 자동 스캔 추가 -->
	<context:component-scan base-package="com.LKHmovie.biz"></context:component-scan>
	
	<!-- AOP -->
	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>

	<!-- Database info -->
	<!-- ~~~ -->
```

- AOP 패키지 생성
  - com.LKHmovie.biz.common.aop 패키지 생성

- AOP 설정 클래스 생성
```java
// PointcutCommon.java

package com.LKHmovie.biz.common.aop;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class PointcutCommon {

	@Pointcut("execution(* com.LKHmovie.biz..*Impl.*(..))")
	public void allPointcut() {}
	
	@Pointcut("execution(* com.LKHmovie.biz..*Impl.get*(..))")
	public void getPointcut() {}
}
```

- AOP before 추가
```java
// BeforeAdvice.java

package com.LKHmovie.biz.common.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
@Aspect
public class BeforeAdvice {

	private static final Logger LOGGER = LoggerFactory.getLogger(BeforeAdvice.class);
	
	@Before("PointcutCommon.allPointcut()")
	public void beforeLog(JoinPoint jp) {
		String method = jp.getSignature().getName();
		Object[] args = jp.getArgs();
		
		LOGGER.info("-------------------------------------------------------------------------------------------------------");
		if(args.length > 0) {
			LOGGER.info("[Before] " + method + " 메소드 args 정보 : " + args[0].toString());
		} else {
			LOGGER.info("[Before] " + method + " 메소드");
		}
	}
}
```

- AOP around 추가
```java
// AroundAdvice.java

package com.LKHmovie.biz.common.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;
import org.springframework.util.StopWatch;

@Service
@Aspect
public class AroundAdvice {

	private static final Logger LOGGER = LoggerFactory.getLogger(AroundAdvice.class);
	
	@Around("PointcutCommon.allPointcut()")
	public Object aroundLog(ProceedingJoinPoint pjp) throws Throwable {
		String method = pjp.getSignature().getName();
		
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		
		Object returnObj = pjp.proceed();
		
		stopWatch.stop();
		
		LOGGER.info("[Around] " + method + " 메소드 수행에 걸린 시간 : " + stopWatch.getTotalTimeMillis() + "(ms)초");
		
		return returnObj;
	}
}
```

- AOP afterReturning 추가
```java
// AfterReturningAdvice.java

package com.LKHmovie.biz.common.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
@Aspect
public class AfterReturningAdvice {

	private static final Logger LOGGER = LoggerFactory.getLogger(AfterReturningAdvice.class);
	
	@AfterReturning(pointcut="PointcutCommon.getPointcut()", returning="returnObj")
	public void afterLog(JoinPoint jp, Object returnObj) {
		String method = jp.getSignature().getName();
		
		if(returnObj != null) {
			LOGGER.info("[AfterReturning] " + method + " 메소드 리턴값 : " + returnObj.toString());
		} else {
			LOGGER.info("[AfterReturning] " + method + " 메소드 리턴값 : null");
		}
	}
}
```

- AOP after 추가
```java
// AfterAdvice.java

package com.LKHmovie.biz.common.aop;

import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
@Aspect
public class AfterAdvice {

	private static final Logger LOGGER = LoggerFactory.getLogger(AfterAdvice.class);
	
	@After("PointcutCommon.allPointcut()")
	public void finallyLog() {
		LOGGER.info("[After] 비즈니스 로직 수행 끝");
		LOGGER.info("-------------------------------------------------------------------------------------------------------");
	}
}
```

- AOP transaction 인터페이스 추가
```java
// PlatformTransactionManager.java

package com.LKHmovie.biz.common.aop;

import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionException;
import org.springframework.transaction.TransactionStatus;

public interface PlatformTransactionManager {

	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
	void commit(TransactionStatus status) throws TransactionException;
	void rollback(TransactionStatus status) throws TransactionException;
}
```

- AOP transaction 설정 추가
```xml
<!-- applicationContext.xml -->

	<!-- ~~~ -->
		<constructor-arg ref="sqlSession"></constructor-arg>
	</bean>

	<!-- Transaction 설정 -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<tx:advice id="txAdvice" transaction-manager="txManager">
		<tx:attributes>
			<tx:method name="get*" read-only="true"/>
			<tx:method name="*"/>
		</tx:attributes>
	</tx:advice>
	
	<aop:config>
		<aop:pointcut expression="execution(* com.LKHmovie.biz..*(..))" id="txPointcut"/>
		<aop:advisor pointcut-ref="txPointcut" advice-ref="txAdvice"/>
	</aop:config>
</beans>
```

---

## 7. restful 적용
- postman 설치
  - 인터넷어 postman 검색해서 프로그램 설치
  - 로그인 필요

- dependency 추가
```xml
<!-- pom.xml -->

		<!-- ~~~ -->
		<dependencies>
		<!-- use json -->
		<dependency>
		    <groupId>org.codehaus.jackson</groupId>
		    <artifactId>jackson-mapper-asl</artifactId>
		    <version>1.9.13</version>
		</dependency>
		<dependency>
		    <groupId>com.fasterxml.jackson.core</groupId>
		    <artifactId>jackson-databind</artifactId>
		    <version>2.9.8</version>
		</dependency>
		
		<!-- MyBatis -->
		<!-- ~~~ -->
```

- json과 restful 설정 추가
```xml
<!-- servlet-config.xml -->

	<!-- ~~~ -->
	<context:component-scan base-package="com.LKHmovie.controller"></context:component-scan>
	
	<!-- HandlerMapping, HandlerAdapter를 등록 -->
	<mvc:annotation-driven></mvc:annotation-driven>
	<!-- tomcat의 server.xml에 정의돈 url-pattern "/" 무시하고 현재 DispatcherServlet의 url-pattern으로 설정 -->
	<mvc:default-servlet-handler/>
	
	<!-- 부트스트랩 추가 -->
	<!-- ~~~ -->
```

- restful 테스트를 위한 기능 함수 추가
```xml
<!-- servlet-config.xml -->

	<!-- ~~~ -->
			USERLIST;
	</select>
	
	<select id="getTestUser" resultType="testList">
		SELECT 
			* 
		FROM 
			USERLIST 
		WHERE 
			ID = #{id};
	</select>
	
	<insert id="insertTestUser">
		INSERT
		INTO
			USERLIST (
				ID,
				PASSWORD,
				NAME,
				PHONE
			)
			VALUES (
				#{id},
				#{password},
				#{name},
				#{phone}
			);
	</insert>
	
	<update id="updateTestUser">
		UPDATE
			USERLIST
		SET
			PHONE = #{phone}
		WHERE
			ID = #{id};
	</update>
	
	<delete id="deleteTestUser">
		DELETE 
		FROM
			USERLIST
		WHERE
			ID = #{id};
	</delete>
</mapper>
```
```java
// TestListDAO.java
	// ~~~
		return mybatis.selectList("TestListDAO.getTestList");
	}
	
	public TestListVO getTestUser(String id) {
		return mybatis.selectOne("TestListDAO.getTestUser", id);
	}
	
	public void insertTestUser(TestListVO vo) {
		mybatis.insert("TestListDAO.insertTestUser", vo);
	}
	
	public void updateTestUser(TestListVO vo) {
		mybatis.update("TestListDAO.updateTestUser", vo);
	}
	
	public void deleteTestUser(String id) {
		mybatis.delete("TestListDAO.deleteTestUser", id);
	}
}
```
```java
// TestListService.java

	// ~~~
	List<TestListVO> getTestList();

	TestListVO getTestUser(String id);
	
	void insertTestUser(TestListVO vo);
	
	void updateTestUser(TestListVO vo);
	
	void deleteTestUser(String id);
}
```
```java
// TestListServiceImpl.java

	// ~~~
		return listDAO.getTestList();
	}

	@Override
	public TestListVO getTestUser(String id) {
		return listDAO.getTestUser(id);
	}

	@Override
	public void insertTestUser(TestListVO vo) {
		listDAO.insertTestUser(vo);
	}
	
	@Override
	public void updateTestUser(TestListVO vo) {
		listDAO.updateTestUser(vo);
	}

	@Override
	public void deleteTestUser(String id) {
		listDAO.deleteTestUser(id);
	}
}
```

- restful 테스트용 컨트롤러 추가
  - `com.LKHmovie.controller.test` 패키지 하위에 컨트롤러 파일 추가
```java
// GetTestRestfulController.java
package com.LKHmovie.controller.test;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import com.LKHmovie.biz.test.list.TestListService;
import com.LKHmovie.biz.test.list.TestListVO;

@Controller
@RequestMapping(value="/restful")
public class GetTestRestfulController {
	private static final Logger LOGGER = LoggerFactory.getLogger(GetTestRestfulController.class);
	
	@Autowired
	private TestListService testListService;
	
	@RequestMapping(value="/testList", method=RequestMethod.GET)
	@ResponseBody
	public Map<String, Object> getTestList() {
		Map<String, Object> result = new HashMap<String, Object>();

		try {
			List<TestListVO> testList = testListService.getTestList();
			
			result.put("result", Boolean.TRUE);
			result.put("data", testList);
		} catch(Exception e) {
			result.put("result", Boolean.FALSE);
			
			LOGGER.error("error message : " + e.getMessage());
			LOGGER.error("error trace : ", e);
		}
		
		return result;
	}
	
	@RequestMapping(value="/testList/{id}", method=RequestMethod.GET)
	@ResponseBody
	public Map<String, Object> getTestUser(@PathVariable String id) {
		Map<String, Object> result = new HashMap<String, Object>();
		
		try {
			TestListVO user = testListService.getTestUser(id);
			
			result.put("result", Boolean.TRUE);
			result.put("data", user);
		} catch(Exception e) {
			result.put("result", Boolean.FALSE);
			
			LOGGER.error("error message : " + e.getMessage());
			LOGGER.error("error trace : ", e);
		}
		
		return result;
	}
	
	@RequestMapping(value="/testList", method=RequestMethod.POST, headers={"Content-type=application/json"})
	@ResponseBody
	public Map<String, Object> insertTestUser(@RequestBody TestListVO user) {
		Map<String, Object> result = new HashMap<String, Object>();
		
		try {
			if(user != null) {
				testListService.insertTestUser(user);
			}
			
			result.put("result", Boolean.TRUE);
		} catch(Exception e) {
			result.put("result", Boolean.FALSE);
			
			LOGGER.error("error message : " + e.getMessage());
			LOGGER.error("error trace : ", e);
		}
		
		return result;
	}
	
	@RequestMapping(value="/testList", method=RequestMethod.PUT, headers= {"Content-type=application/json"})
	@ResponseBody
	public Map<String, Object> updateTestUser(@RequestBody TestListVO user) {
		Map<String, Object> result = new HashMap<String, Object>();
		
		try {
			if(user != null) {
				testListService.updateTestUser(user);
			}
			
			result.put("result", Boolean.TRUE);
		} catch(Exception e) {
			result.put("result", Boolean.FALSE);
			
			LOGGER.error("error message : " + e.getMessage());
			LOGGER.error("error trace : ", e);
		}
		
		return result;
	}
	
	@RequestMapping(value="/testList/{id}", method=RequestMethod.DELETE)
	@ResponseBody
	public Map<String, Object> deleteTestUser(@PathVariable String id) {
		Map<String, Object> result = new HashMap<String, Object>();
		
		try {
			testListService.deleteTestUser(id);
			
			result.put("result", Boolean.TRUE);
		} catch(Exception e) {
			result.put("result", Boolean.FALSE);
			
			LOGGER.error("error message : " + e.getMessage());
			LOGGER.error("error trace : ", e);
		}
		
		return result;
	}
}
```

- postman으로 테스트
  - launchpad > create a request
  - 전체 목록 조회 (GET)
    - GET 선택
    - url : http://localhost:8080/admin/restful/testList
    - send 클릭
  - 단일 조회 (GET)
    - GET 선택 
    - url : http://localhost:8080/admin/restful/testList/user4
    - send 클릭
  - 등록 (POST)
    - POST 선택
    - url : http://localhost:8080/admin/restful/testList
    - Headers > key : Content-Type
    - Headers > value : application/json
    - Body > raw 선택 및 우측의 json 확인
      - `{"id" : "user4", "password" : "4444", "name" : "사유저", "phone" : "01044445555"}` 입력
    - send 클릭
  - 수정 (PUT)
    - PUT 선택
    - url : http://localhost:8080/admin/restful/testList
    - Headers > key : Content-Type
    - Headers > value : application/json
    - Body > raw 선택 및 우측의 json 확인
      - `{"id" : "user4", "phone" : "01044446666"}` 입력
    - send 클릭
  - 삭제 (DELETE)
    - DELETE 선택
    - url : http://localhost:8080/admin/restful/testList/user4
    - send 클릭

- web상에서 테스트
  - 컨트롤러에 restful 테스트 페이지 매핑 함수 추가
```java
// HomeController.java

	// ~~~
		return "home_bootstrapTest2";
	}
	
	@RequestMapping(value = "/testRestful.do", method = RequestMethod.GET)
	public String testRestful(Model model) {
		return "home_testRestful";
	}
}
```
  - ajax통신을 위한 jquery 라이브러리 추가
    - `src/main/webapp/resources/js/lib` 경로에 jquery-3.3.1.js 추가
  - `src/main/webapp/WEB-INF/views` 경로에 restful 테스트용 페이지 추가
```jsp
<!-- home_testRestful.jsp -->

<%@ page contentType="text/html; charset=UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<html>
<head>
	<title>Home</title>
	<script type="text/javascript" src="<c:url value="/resources/js/lib/jquery-3.3.1.js"/>"></script>
	<script type="text/javascript" src="<c:url value="/resources/js/home_testRestful.js"/>"></script>
</head>
<body>
<h1>
	Hello world!  
</h1>

<div>
	<p>
		<span>id : </span>
		<span>
			<input type="text" class="flagDisabled" id="id" />
		</span>
	</p>
	<p>
		<span>password : </span>
		<span>
			<input type="text" class="flagDisabled" id="password" />
		</span>
	</p>
	<p>
		<span>name : </span>
		<span>
			<input type="text" class="flagDisabled" id="name" />
		</span>
	</p>
	<p>
		<span>date : </span>
		<span>
			<input type="text" class="flagDisabled" id="createDate" />
		</span>
	</p>
	<p>
		<span>phone : </span>
		<span>
			<input type="text" id="phone" />
		</span>
	</p>
	<p>
		<input type="button" id="btnInsert" value="insert" />
		<input type="button" id="btnUpdate" value="update" />
		<input type="button" id="btnReset" value="reset" />
	</p>
</div>

<div>
	<table>
		<thead>
			<tr>
				<th>id</th>
				<th>name</th>
			</tr>
		</thead>
		<tbody></tbody>
	</table>
</div>

</body>
</html>
```
  - ajax 통신을 위한 js 파일 추가
    - `src/main/webapp/resources/js` 경로에 home_testRestful.js 추가
```js
// home_testRestful.js

/**
 * 
 */

$(function(){
	
	init();
	bindEvent();
	
	function init() {
		userList();
	};
	
	function bindEvent() {
		$("body").on("click", "#btnInsert", function() {
			var reqData = {
				id : $("#id").val(),
				password : $("#password").val(),
				name : $("#name").val(),
				phone : $("#phone").val()
			};
			
			$.ajax({
				url : "restful/testList",
				type : "POST",
				dataType : "json",
				data : JSON.stringify(reqData),
				contentType : "application/json",
				mimeType : "application/json",
				error : function(xhr, status, msg) {
					alert("status : " + status + "/nHttp error msg : " + msg);
				},
				success : function(res) {
					if(res.result) {
						userList();
					}
				}
			});
		});
		
		$("body").on("click", "#btnUpdate", function() {
			var reqData = {
					id : $("#id").val(),
					phone : $("#phone").val()
				};
				
				$.ajax({
					url : "restful/testList",
					type : "PUT",
					dataType : "json",
					data : JSON.stringify(reqData),
					contentType : "application/json",
					mimeType : "application/json",
					error : function(xhr, status, msg) {
						alert("status : " + status + "/nHttp error msg : " + msg);
					},
					success : function(res) {
						if(res.result) {
							userList();
						}
					}
				});
		});
		
		$("body").on("click", "#btnReset", function() {
			resetInput(false);
		});
		
		$("body").on("click", ".btnSearch", function(){
			var id = ($(this).parents("tr").data("info")).id;
			
			$.ajax({
				url : "restful/testList/" + id,
				type : "GET",
				contentType : "application/json;charset=utf-8;",
				dataType : "json",
				error : function(xhr, status, msg) {
					alert("status : " + status + "/nHttp error msg : " + msg);
				},
				success : function(res) {
					console.log(res);
					
					resetInput(true);
					
					$("#id").val(res.data.id);
					$("#password").val(res.data.password);
					$("#name").val(res.data.name);
					$("#createDate").val(res.data.createDate);
					$("#phone").val(res.data.phone);
				}
			});
		});
		
		$("body").on("click", ".btnDelete", function() {
			var id = ($(this).parents("tr").data("info")).id;
			var result = confirm(id + "사용자를 삭제하시겠습니까?");
			
			if(result) {
				$.ajax({
					url : "restful/testList/" + id,
					type : "DELETE",
					contentType : "application/json;charset=utf-8;",
					dataType : "json",
					error : function(xhr, status, msg) {
						alert("status : " + status + "/nHttp error msg : " + msg);
					},
					success : function(res) {
						console.log(res);
						
						if(res.result) {
							userList();
						}
					}
				});
			}
		});
	};
	
	function resetInput(isDisabled) {
		if(isDisabled) {
			$(".flagDisabled").prop("disabled", true);
		} else {
			$(".flagDisabled").prop("disabled", false);
		}
		
		$("#id").val("");
		$("#password").val("");
		$("#name").val("");
		$("#createDate").val("");
		$("#phone").val("");
	};
	
	function userList() {
		$.ajax({
			url : "restful/testList",
			type : "GET",
			contentType : "application/json;charset=utf-8",
			dataYtpe : "json",
			error : function(xhr, status, msg) {
				alert("status : " + status + "\nHttp error msg : " + msg);
			},
			success : userListRender
		});
	};
	
	function userListRender(xhr) {
		console.log(xhr.data);
		$("tbody").empty();
		
		$.each(xhr.data, function(idx, item) {
			$("tbody").append(
				$("<tr>").data("info", item).append(
					$("<td>").append(item.id),
					$("<td>").append(item.name),
					$("<td>").append(
						$("<input>").attr({type : "button", value : "search"}).addClass("btnSearch")
					),
					$("<td>").append(
						$("<input>").attr({type : "button", value : "delete"}).addClass("btnDelete")
					),
					// 난 히든은 나중에 활용하기에도 html상에도 보여서 좀 아닌거 같기도 하다아
					$("<td style={display:none}>").append(
						$("<input>").attr({type : "hidden", value : item.id}).addClass("inpHidden")
					)
				)
			);
		});
	};
});
```


---

## 8. bootstrap 적용
- bootstrap 프레임워크 다운
  - adminLTE.io 사이트 접속해서 소스파일 다운로드
  - 압축해제 후 `src/main/sebapp/resources`에 dist, pages, plugins 폴더 복사

- bootstrap 파일 매핑 설정 추가
```xml
<!-- servlet-config.xml -->

	<!-- ~~~ -->
	<mvc:annotation-driven></mvc:annotation-driven>
	
	<!-- 부트스트랩 추가 -->
	<mvc:resources mapping="/resources/**" location="/resources/" />

	<!-- resolver 설정 추가 -->
	<!-- ~~~ -->
```

- 테스트 페이지 추가
  - `src/main/webapp/resources/pages/examples/blank.html` 파일 복사해서 views폴더에 테스트 페이지(home_bootstrapTest.jsp)로 생성
  - views폴더 하위에 include 폴더 생성 후 아래 파일 생성
    - footer.jsp
    - meta.jsp
    - navbar.jsp
    - script.jsp
    - sidebar.jsp

- meta.jsp 작성
  - home_bootstrapTest.jsp에서 `<head>` 부분 잘라내기
  - `../../` 코드는 `resources/`로 수정
```jsp
<!-- meta.jsp -->

<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>

<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<title>LKHmovie_admin</title>
	<!-- Tell the browser to be responsive to screen width -->
	<meta name="viewport" content="width=device-width, initial-scale=1">
	
	<!-- Font Awesome -->
	<link rel="stylesheet" href="resources/plugins/fontawesome-free/css/all.min.css">
	<!-- Ionicons -->
	<link rel="stylesheet" href="https://code.ionicframework.com/ionicons/2.0.1/css/ionicons.min.css">
	<!-- overlayScrollbars -->
	<link rel="stylesheet" href="resources/dist/css/adminlte.min.css">
	<!-- Google Font: Source Sans Pro -->
	<link href="https://fonts.googleapis.com/css?family=Source+Sans+Pro:300,400,400i,700" rel="stylesheet">
</head>
```

- navbar.jsp 작성
  - home_bootstrapTest.jsp에서 `<nav>` 부분 잘라내기
  - `../../` 코드는 `resources/`로 수정
```jsp
<!-- meta.jsp -->

<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>

<nav class="main-header navbar navbar-expand navbar-white navbar-light">
	<!-- Left navbar links -->
	<ul class="navbar-nav">
		<li class="nav-item"><a class="nav-link" data-widget="pushmenu"
			href="#"><i class="fas fa-bars"></i></a></li>
		<li class="nav-item d-none d-sm-inline-block"><a
			href="resources/index3.html" class="nav-link">Home</a></li>
		<li class="nav-item d-none d-sm-inline-block"><a href="#"
			class="nav-link">Contact</a></li>
	</ul>

	<!-- SEARCH FORM -->
	<form class="form-inline ml-3">
		<div class="input-group input-group-sm">
			<input class="form-control form-control-navbar" type="search"
				placeholder="Search" aria-label="Search">
			<div class="input-group-append">
				<button class="btn btn-navbar" type="submit">
					<i class="fas fa-search"></i>
				</button>
			</div>
		</div>
	</form>

	<!-- Right navbar links -->
	<ul class="navbar-nav ml-auto">
		<!-- Messages Dropdown Menu -->
		<li class="nav-item dropdown"><a class="nav-link"
			data-toggle="dropdown" href="#"> <i class="far fa-comments"></i>
				<span class="badge badge-danger navbar-badge">3</span>
		</a>
			<div class="dropdown-menu dropdown-menu-lg dropdown-menu-right">
				<a href="#" class="dropdown-item"> <!-- Message Start -->
					<div class="media">
						<img src="resources/dist/img/user1-128x128.jpg" alt="User Avatar"
							class="img-size-50 mr-3 img-circle">
						<div class="media-body">
							<h3 class="dropdown-item-title">
								Brad Diesel <span class="float-right text-sm text-danger"><i
									class="fas fa-star"></i></span>
							</h3>
							<p class="text-sm">Call me whenever you can...</p>
							<p class="text-sm text-muted">
								<i class="far fa-clock mr-1"></i> 4 Hours Ago
							</p>
						</div>
					</div> <!-- Message End -->
				</a>
				<div class="dropdown-divider"></div>
				<a href="#" class="dropdown-item"> <!-- Message Start -->
					<div class="media">
						<img src="resources/dist/img/user8-128x128.jpg" alt="User Avatar"
							class="img-size-50 img-circle mr-3">
						<div class="media-body">
							<h3 class="dropdown-item-title">
								John Pierce <span class="float-right text-sm text-muted"><i
									class="fas fa-star"></i></span>
							</h3>
							<p class="text-sm">I got your message bro</p>
							<p class="text-sm text-muted">
								<i class="far fa-clock mr-1"></i> 4 Hours Ago
							</p>
						</div>
					</div> <!-- Message End -->
				</a>
				<div class="dropdown-divider"></div>
				<a href="#" class="dropdown-item"> <!-- Message Start -->
					<div class="media">
						<img src="resources/dist/img/user3-128x128.jpg" alt="User Avatar"
							class="img-size-50 img-circle mr-3">
						<div class="media-body">
							<h3 class="dropdown-item-title">
								Nora Silvester <span class="float-right text-sm text-warning"><i
									class="fas fa-star"></i></span>
							</h3>
							<p class="text-sm">The subject goes here</p>
							<p class="text-sm text-muted">
								<i class="far fa-clock mr-1"></i> 4 Hours Ago
							</p>
						</div>
					</div> <!-- Message End -->
				</a>
				<div class="dropdown-divider"></div>
				<a href="#" class="dropdown-item dropdown-footer">See All
					Messages</a>
			</div></li>
		<!-- Notifications Dropdown Menu -->
		<li class="nav-item dropdown"><a class="nav-link"
			data-toggle="dropdown" href="#"> <i class="far fa-bell"></i> <span
				class="badge badge-warning navbar-badge">15</span>
		</a>
			<div class="dropdown-menu dropdown-menu-lg dropdown-menu-right">
				<span class="dropdown-item dropdown-header">15 Notifications</span>
				<div class="dropdown-divider"></div>
				<a href="#" class="dropdown-item"> <i
					class="fas fa-envelope mr-2"></i> 4 new messages <span
					class="float-right text-muted text-sm">3 mins</span>
				</a>
				<div class="dropdown-divider"></div>
				<a href="#" class="dropdown-item"> <i class="fas fa-users mr-2"></i>
					8 friend requests <span class="float-right text-muted text-sm">12
						hours</span>
				</a>
				<div class="dropdown-divider"></div>
				<a href="#" class="dropdown-item"> <i class="fas fa-file mr-2"></i>
					3 new reports <span class="float-right text-muted text-sm">2
						days</span>
				</a>
				<div class="dropdown-divider"></div>
				<a href="#" class="dropdown-item dropdown-footer">See All
					Notifications</a>
			</div></li>
		<li class="nav-item"><a class="nav-link"
			data-widget="control-sidebar" data-slide="true" href="#"> <i
				class="fas fa-th-large"></i>
		</a></li>
	</ul>
</nav>
```

- sidebar.jsp 작성
  - home_bootstrapTest.jsp에서 `<aside>` 부분 잘라내기
  - `../../` 코드는 `resources/`로 수정
```jsp
<!-- sidebar.jsp -->

<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>

<aside class="main-sidebar sidebar-dark-primary elevation-4">
	<!-- Brand Logo -->
	<a href="resources/index3.html" class="brand-link"> <img
		src="resources/dist/img/AdminLTELogo.png" alt="AdminLTE Logo"
		class="brand-image img-circle elevation-3" style="opacity: .8">
		<span class="brand-text font-weight-light">AdminLTE 3</span>
	</a>

	<!-- Sidebar -->
	<div class="sidebar">
		<!-- Sidebar user (optional) -->
		<div class="user-panel mt-3 pb-3 mb-3 d-flex">
			<div class="image">
				<img src="resources/dist/img/user2-160x160.jpg"
					class="img-circle elevation-2" alt="User Image">
			</div>
			<div class="info">
				<a href="#" class="d-block">Alexander Pierce</a>
			</div>
		</div>

		<!-- Sidebar Menu -->
		<nav class="mt-2">
			<ul class="nav nav-pills nav-sidebar flex-column"
				data-widget="treeview" role="menu" data-accordion="false">
				<!-- Add icons to the links using the .nav-icon class
               with font-awesome or any other icon font library -->
				<li class="nav-item has-treeview"><a href="#" class="nav-link">
						<i class="nav-icon fas fa-tachometer-alt"></i>
						<p>
							Dashboard <i class="right fas fa-angle-left"></i>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="resources/index.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Dashboard v1</p>
						</a></li>
						<li class="nav-item"><a href="resources/index2.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Dashboard v2</p>
						</a></li>
						<li class="nav-item"><a href="resources/index3.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Dashboard v3</p>
						</a></li>
					</ul></li>
				<li class="nav-item"><a href="../widgets.html" class="nav-link">
						<i class="nav-icon fas fa-th"></i>
						<p>
							Widgets <span class="right badge badge-danger">New</span>
						</p>
				</a></li>
				<li class="nav-item has-treeview"><a href="#" class="nav-link">
						<i class="nav-icon fas fa-copy"></i>
						<p>
							Layout Options <i class="fas fa-angle-left right"></i> <span
								class="badge badge-info right">6</span>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="../layout/top-nav.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Top Navigation</p>
						</a></li>
						<li class="nav-item"><a href="../layout/boxed.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Boxed</p>
						</a></li>
						<li class="nav-item"><a href="../layout/fixed-sidebar.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Fixed Sidebar</p>
						</a></li>
						<li class="nav-item"><a href="../layout/fixed-topnav.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Fixed Navbar</p>
						</a></li>
						<li class="nav-item"><a href="../layout/fixed-footer.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Fixed Footer</p>
						</a></li>
						<li class="nav-item"><a
							href="../layout/collapsed-sidebar.html" class="nav-link"> <i
								class="far fa-circle nav-icon"></i>
								<p>Collapsed Sidebar</p>
						</a></li>
					</ul></li>
				<li class="nav-item has-treeview"><a href="#" class="nav-link">
						<i class="nav-icon fas fa-chart-pie"></i>
						<p>
							Charts <i class="right fas fa-angle-left"></i>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="../charts/chartjs.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>ChartJS</p>
						</a></li>
						<li class="nav-item"><a href="../charts/flot.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Flot</p>
						</a></li>
						<li class="nav-item"><a href="../charts/inline.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Inline</p>
						</a></li>
					</ul></li>
				<li class="nav-item has-treeview"><a href="#" class="nav-link">
						<i class="nav-icon fas fa-tree"></i>
						<p>
							UI Elements <i class="fas fa-angle-left right"></i>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="../UI/general.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>General</p>
						</a></li>
						<li class="nav-item"><a href="../UI/icons.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Icons</p>
						</a></li>
						<li class="nav-item"><a href="../UI/buttons.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Buttons</p>
						</a></li>
						<li class="nav-item"><a href="../UI/sliders.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Sliders</p>
						</a></li>
						<li class="nav-item"><a href="../UI/modals.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Modals & Alerts</p>
						</a></li>
						<li class="nav-item"><a href="../UI/navbar.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Navbar & Tabs</p>
						</a></li>
						<li class="nav-item"><a href="../UI/timeline.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Timeline</p>
						</a></li>
						<li class="nav-item"><a href="../UI/ribbons.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Ribbons</p>
						</a></li>
					</ul></li>
				<li class="nav-item has-treeview"><a href="#" class="nav-link">
						<i class="nav-icon fas fa-edit"></i>
						<p>
							Forms <i class="fas fa-angle-left right"></i>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="../forms/general.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>General Elements</p>
						</a></li>
						<li class="nav-item"><a href="../forms/advanced.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Advanced Elements</p>
						</a></li>
						<li class="nav-item"><a href="../forms/editors.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Editors</p>
						</a></li>
					</ul></li>
				<li class="nav-item has-treeview"><a href="#" class="nav-link">
						<i class="nav-icon fas fa-table"></i>
						<p>
							Tables <i class="fas fa-angle-left right"></i>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="../tables/simple.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Simple Tables</p>
						</a></li>
						<li class="nav-item"><a href="../tables/data.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>DataTables</p>
						</a></li>
						<li class="nav-item"><a href="../tables/jsgrid.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>jsGrid</p>
						</a></li>
					</ul></li>
				<li class="nav-header">EXAMPLES</li>
				<li class="nav-item"><a href="../calendar.html"
					class="nav-link"> <i class="nav-icon far fa-calendar-alt"></i>
						<p>
							Calendar <span class="badge badge-info right">2</span>
						</p>
				</a></li>
				<li class="nav-item"><a href="../gallery.html" class="nav-link">
						<i class="nav-icon far fa-image"></i>
						<p>Gallery</p>
				</a></li>
				<li class="nav-item has-treeview"><a href="#" class="nav-link">
						<i class="nav-icon far fa-envelope"></i>
						<p>
							Mailbox <i class="fas fa-angle-left right"></i>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="../mailbox/mailbox.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Inbox</p>
						</a></li>
						<li class="nav-item"><a href="../mailbox/compose.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Compose</p>
						</a></li>
						<li class="nav-item"><a href="../mailbox/read-mail.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Read</p>
						</a></li>
					</ul></li>
				<li class="nav-item has-treeview"><a href="#" class="nav-link">
						<i class="nav-icon fas fa-book"></i>
						<p>
							Pages <i class="fas fa-angle-left right"></i>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="../examples/invoice.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Invoice</p>
						</a></li>
						<li class="nav-item"><a href="../examples/profile.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Profile</p>
						</a></li>
						<li class="nav-item"><a href="../examples/e_commerce.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>E-commerce</p>
						</a></li>
						<li class="nav-item"><a href="../examples/projects.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Projects</p>
						</a></li>
						<li class="nav-item"><a href="../examples/project_add.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Project Add</p>
						</a></li>
						<li class="nav-item"><a href="../examples/project_edit.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Project Edit</p>
						</a></li>
						<li class="nav-item"><a
							href="../examples/project_detail.html" class="nav-link"> <i
								class="far fa-circle nav-icon"></i>
								<p>Project Detail</p>
						</a></li>
						<li class="nav-item"><a href="../examples/contacts.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Contacts</p>
						</a></li>
					</ul></li>
				<li class="nav-item has-treeview menu-open"><a href="#"
					class="nav-link active"> <i class="nav-icon far fa-plus-square"></i>
						<p>
							Extras <i class="fas fa-angle-left right"></i>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="../examples/login.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Login</p>
						</a></li>
						<li class="nav-item"><a href="../examples/register.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Register</p>
						</a></li>
						<li class="nav-item"><a
							href="..examples/forgot-password.html" class="nav-link"> <i
								class="far fa-circle nav-icon"></i>
								<p>Forgot Password</p>
						</a></li>
						<li class="nav-item"><a
							href="..examples/recover-password.html" class="nav-link"> <i
								class="far fa-circle nav-icon"></i>
								<p>Recover Password</p>
						</a></li>
						<li class="nav-item"><a href="../examples/lockscreen.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Lockscreen</p>
						</a></li>
						<li class="nav-item"><a
							href="../examples/legacy-user-menu.html" class="nav-link"> <i
								class="far fa-circle nav-icon"></i>
								<p>Legacy User Menu</p>
						</a></li>
						<li class="nav-item"><a href="../examples/language-menu.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Language Menu</p>
						</a></li>
						<li class="nav-item"><a href="../examples/404.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Error 404</p>
						</a></li>
						<li class="nav-item"><a href="../examples/500.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Error 500</p>
						</a></li>
						<li class="nav-item"><a href="../examples/pace.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Pace</p>
						</a></li>
						<li class="nav-item"><a href="../examples/blank.html"
							class="nav-link active"> <i class="far fa-circle nav-icon"></i>
								<p>Blank Page</p>
						</a></li>
						<li class="nav-item"><a href="resources/starter.html"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>Starter Page</p>
						</a></li>
					</ul></li>
				<li class="nav-header">MISCELLANEOUS</li>
				<li class="nav-item"><a href="https://adminlte.io/docs/3.0"
					class="nav-link"> <i class="nav-icon fas fa-file"></i>
						<p>Documentation</p>
				</a></li>
				<li class="nav-header">MULTI LEVEL EXAMPLE</li>
				<li class="nav-item"><a href="#" class="nav-link"> <i
						class="fas fa-circle nav-icon"></i>
						<p>Level 1</p>
				</a></li>
				<li class="nav-item has-treeview"><a href="#" class="nav-link">
						<i class="nav-icon fas fa-circle"></i>
						<p>
							Level 1 <i class="right fas fa-angle-left"></i>
						</p>
				</a>
					<ul class="nav nav-treeview">
						<li class="nav-item"><a href="#" class="nav-link"> <i
								class="far fa-circle nav-icon"></i>
								<p>Level 2</p>
						</a></li>
						<li class="nav-item has-treeview"><a href="#"
							class="nav-link"> <i class="far fa-circle nav-icon"></i>
								<p>
									Level 2 <i class="right fas fa-angle-left"></i>
								</p>
						</a>
							<ul class="nav nav-treeview">
								<li class="nav-item"><a href="#" class="nav-link"> <i
										class="far fa-dot-circle nav-icon"></i>
										<p>Level 3</p>
								</a></li>
								<li class="nav-item"><a href="#" class="nav-link"> <i
										class="far fa-dot-circle nav-icon"></i>
										<p>Level 3</p>
								</a></li>
								<li class="nav-item"><a href="#" class="nav-link"> <i
										class="far fa-dot-circle nav-icon"></i>
										<p>Level 3</p>
								</a></li>
							</ul></li>
						<li class="nav-item"><a href="#" class="nav-link"> <i
								class="far fa-circle nav-icon"></i>
								<p>Level 2</p>
						</a></li>
					</ul></li>
				<li class="nav-item"><a href="#" class="nav-link"> <i
						class="fas fa-circle nav-icon"></i>
						<p>Level 1</p>
				</a></li>
				<li class="nav-header">LABELS</li>
				<li class="nav-item"><a href="#" class="nav-link"> <i
						class="nav-icon far fa-circle text-danger"></i>
						<p class="text">Important</p>
				</a></li>
				<li class="nav-item"><a href="#" class="nav-link"> <i
						class="nav-icon far fa-circle text-warning"></i>
						<p>Warning</p>
				</a></li>
				<li class="nav-item"><a href="#" class="nav-link"> <i
						class="nav-icon far fa-circle text-info"></i>
						<p>Informational</p>
				</a></li>
			</ul>
		</nav>
		<!-- /.sidebar-menu -->
	</div>
	<!-- /.sidebar -->
</aside>
```

- footer.jsp 작성
  - home_bootstrapTest.jsp에서 `<footer>` 부분 잘라내기
  - `../../` 코드는 `resources/`로 수정
```jsp
<!-- footer.jsp -->

<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>

<footer class="main-footer">
	<div class="float-right d-none d-sm-block">
		<b>Version</b> 3.0.0
	</div>
	<strong>Copyright &copy; 2014-2019 <a
		href="http://adminlte.io">AdminLTE.io</a>.
	</strong> All rights reserved.
</footer>
```

- script.jsp 작성
  - home_bootstrapTest.jsp에서 `<script>` 부분 잘라내기
  - `../../` 코드는 `resources/`로 수정
```jsp
<!-- script.jsp -->

<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>

<!-- jQuery -->
<script src="resources/plugins/jquery/jquery.min.js"></script>
<!-- Bootstrap 4 -->
<script src="resources/plugins/bootstrap/js/bootstrap.bundle.min.js"></script>
<!-- AdminLTE App -->
<script src="resources/dist/js/adminlte.min.js"></script>
<!-- AdminLTE for demo purposes -->
<script src="resources/dist/js/demo.js"></script>
```

- 잘라내기 후 남은 home_bootstrapTest.jsp
  - 서버 통신 테스트를 위한 testText 변수 추가 해둠
```jsp
<!-- home_bootstrapTest.jsp -->

<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
	
<html>

<!-- meta -->
<%@ include file="./include/meta.jsp" %>

<body class="hold-transition sidebar-mini">
	<!-- Site wrapper -->
	<div class="wrapper">
		<!-- Navbar -->
		<%@ include file="./include/navbar.jsp" %>

		<!-- Main Sidebar Container -->
		<%@ include file="./include/sidebar.jsp" %>

		<!-- Content Wrapper. Contains page content -->
		<div class="content-wrapper">
			<!-- Content Header (Page header) -->
			<section class="content-header">
				<div class="container-fluid">
					<div class="row mb-2">
						<div class="col-sm-6">
							<h1>Blank Page</h1>
						</div>
						<div class="col-sm-6">
							<ol class="breadcrumb float-sm-right">
								<li class="breadcrumb-item"><a href="#">Home</a></li>
								<li class="breadcrumb-item active">Blank Page</li>
							</ol>
						</div>
					</div>
				</div>
				<!-- /.container-fluid -->
			</section>

			<!-- Main content -->
			<section class="content">

				<!-- Default box -->
				<div class="card">
					<div class="card-header">
						<h3 class="card-title">Title</h3>

						<div class="card-tools">
							<button type="button" class="btn btn-tool"
								data-card-widget="collapse" data-toggle="tooltip"
								title="Collapse">
								<i class="fas fa-minus"></i>
							</button>
							<button type="button" class="btn btn-tool"
								data-card-widget="remove" data-toggle="tooltip" title="Remove">
								<i class="fas fa-times"></i>
							</button>
						</div>
					</div>
					<div class="card-body">
						${testText }
					</div>
					<!-- /.card-body -->
					<div class="card-footer">Footer</div>
					<!-- /.card-footer-->
				</div>
				<!-- /.card -->

			</section>
			<!-- /.content -->
		</div>
		<!-- /.content-wrapper -->

		<!-- footer -->
		<%@ include file="./include/footer.jsp" %>

		<!-- Control Sidebar --> <!-- 쓰이는데가 없는 듯하다 -->
		<!-- <aside class="control-sidebar control-sidebar-dark">
			Control sidebar content goes here
		</aside> -->
		<!-- /.control-sidebar -->
	</div>
	<!-- ./wrapper -->

	<!-- script -->
	<%@ include file="./include/script.jsp" %>

</body>
</html>
```

- 테스트 페이지 호출용 컨트롤러 코드 추가
```java
// HomeController.java

	// ~~~
	@RequestMapping(value = "/bootstrapTest.do", method = RequestMethod.GET)
	public String bootStrapTest(Model model) {
		model.addAttribute("testText", "hello world!!");
		
		return "home_bootstrapTest";
	}
}
```

---

## 9. 
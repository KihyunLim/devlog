# LKHmovie_admin 프로젝트

## 1. 프로젝트 생성
- help > Eclipse MarketPlace > sts 검색 > Spring Tools 3 머시기 release 설치
- new > spring legacy project > project name 입력 > templates : spring MVC project > next > top-level package : "com.SampleBoard.khlim" > finish > 프로젝트 생성<br><br>


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

## 3. DB 연동
- Maven Dependencies 추가
```xml
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
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3307/movie?useSSL=false
jdbc.username=root
jdbc.password=[자기가 설정한 비밀번호]
```

- applicationContext에 DB 정보 설정 입력
```xml
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
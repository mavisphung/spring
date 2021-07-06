### Hướng dẫn cấu hình 1 project maven
1. Thêm các dependencies của Maven cho app
2. Tạo file cấu hình cho app a.k.a `@Configuration`
3. Tạo Spring Dispatcher Servlet Initializer được kế thừa từ lớp cha `AbstractAnnotationConfigDispatcherServletInitializer`
4. Tạo một Spring controller
5. Tạo view jsp


##### 1. Thêm các dependencies của Maven cho app
File pom.xml:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.luv2code</groupId>
	<artifactId>spring-security-demo</artifactId>
	<version>1.0</version>
	<packaging>war</packaging>

	<name>spring-security-demo</name>

	<properties>
		<springframework.version>5.2.9.RELEASE</springframework.version>

		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
	</properties>

	<dependencies>

		<!-- Spring MVC support -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${springframework.version}</version>
		</dependency>

		<!-- Servlet, JSP and JSTL support -->
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
		</dependency>

		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>javax.servlet.jsp-api</artifactId>
			<version>2.3.1</version>
		</dependency>

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>

		<!-- to compensate for java 9+ not including jaxb -->
		<dependency>
			<groupId>javax.xml.bind</groupId>
			<artifactId>jaxb-api</artifactId>
			<version>2.3.0</version>
		</dependency>

		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>

	</dependencies>

	<!-- TO DO: Add support for Maven WAR Plugin -->

	<build>
		<finalName>spring-security-demo</finalName>

		<pluginManagement>
			<plugins>
				<!-- Thêm Maven coordinates (GAV) for: maven-war-plugin -->
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-war-plugin</artifactId>
					<version>3.3.1</version>
				</plugin>
			</plugins>
		</pluginManagement>

	</build>

</project>

```

##### 2 .Tạo file cấu hình cho app a.k.a `@Configuration`
Tạo một package mới với tên như sau trong đường dẫn `src/main/java`: `me.huypc.springsecurity.config` với `me.huypc` là group id, từ `springsecurity.*` trở đi là artifact id.

Trong package vừa tạo, tạo một java class `AppConfig.java`. Sau đó cấu hình như bên dưới:

```java
package me.huypc.springsecurity.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "me.huypc.springsecurity")
public class AppConfig {
	
	//Phải có ViewResolver để render ra view
	@Bean
	public ViewResolver viewResolver() {
		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
		//url: /WEB-INF/view/<tên trang>.jsp
		viewResolver.setPrefix("/WEB-INF/view/");
		viewResolver.setSuffix(".jsp");
		return viewResolver;
	}
}
```

##### 3. Tạo Spring Dispatcher Servlet Initializer được kế thừa từ lớp cha `AbstractAnnotationConfigDispatcherServletInitializer`
Chuột phải vào package của file config, new class với tên đặt sao cho gọn là được và kế thừa lớp `AbstractAnnotationConfigDispatcherServletInitializer`

```java
package me.huypc.springsecurity.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class DispatcherServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		// TODO Auto-generated method stub
		return new Class[] { AppConfig.class };
	}

	@Override
	protected String[] getServletMappings() {
		// TODO Auto-generated method stub
		return new String[] { "/" };
	}

}

```

Ảnh bên dưới là so sánh với cấu hình bằng xml:
![[Pasted image 20210628183334.png]]



### Hướng dẫn cấu hình Spring Security cơ bản trong 1 project
* Để thực hiện hướng dẫn này thì phải cấu hình 1 project maven cơ bản, trong đó file pom.xml phải có khai báo dependency `spring-security-web` và `spring-security-config`

1. Khởi tạo lớp khởi động cho Spring Security (Lớp Initializer)
2. Tạo lớp Configuration `@Configuration`
3. Thêm users, password và role (cơ bản)

##### 1. Khởi tạo lớp khởi động cho Spring Security (Lớp Initializer)
![[Pasted image 20210629132033.png]]

##### 2. Tạo lớp Configuration `@Configuration`
![[Pasted image 20210629132122.png]]

##### 3. Thêm users, password và role (cơ bản)
![[Pasted image 20210629132639.png]]

### ASD
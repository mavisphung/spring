### Kể từ đây...
Tạo project theo phong cách maven và sử dụng toàn bộ code java để config, không dùng xml nữa.
[[huong-dan-config|Hướng dẫn config]]

### Overview
- Secure Spring MVC web app
- Phát triển trang login (mặc định hoặc custom)
- Phân quyền, phân trang dựa vào role
- Xử lý password

Khá tương đồng với 2 đối tượng UserManager và RoleManager trong .NET 5

Vể bản chất, Spring Security sử dụng Servlet filter để xử lí trước/sau khi web request được gửi đến

![[Pasted image 20210628155146.png]]

Sơ lược về cách thức hoạt động của Spring Security, không khác gì bên .NET

![[Pasted image 20210628155351.png]]

### Các kiểu security
**Declarative Security**

Định nghĩa các ràng buộc bảo mật của app trong trong file java có `@Configuration` (không xml) hoặc file xml, tách rời phần application code và security

**Programmatic Security**

Cung cấp API để sử dụng cho việc bảo mật, flexible hơn loại bảo mật trên


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


### Chức năng login
Trong quá trình code và test, sửa đổi 1 trang jsp rồi bấm refresh nhưng server vẫn log in mình vào là do dựa trên session của browser (web browser session) vẫn còn mở. Chỉ cần session còn mở thì dù mở tab mới vẫn được log in vào. Vấn đề này chỉ xảy ra trong quá trình dev hay testing, còn lên production thì không còn vấn đề này nữa.

Lưu ý: Chỉ áp dụng khi chỉ sử dúng 1 loại trình duyệt. Không áp dụng cho 2 trình duyệt khác nhau trở lên.

Mặc định nếu không khai báo trang login thì Spring Security tự lấy trang login của nó để login.

Để có thể sử dụng trang login ngoài hay bất kì trang nào khác thì phải thông qua method `configure(HttpSecurity)` của lớp `WebSecurityConfigurerAdapter`.

![[Pasted image 20210630150428.png]]

Override lại method `configure(HttpSecurity)`

![[Pasted image 20210701105628.png]]

Đoạn code sau khi config, lưu ý là parameter phải là `HttpSecurity`

![[Pasted image 20210701110832.png]]

Sau khi config xong đối tượng của lớp Adapter, giờ đến lượt config controller cho việc login. Ở package chứa controller > new > đặt tên file là LoginController.java

![[Pasted image 20210701112714.png]]

Và copy hoặc new một trang login ở thư mục /WEB-INF/view

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Login page</title>
</head>
<body>
	<h3>My Custom Login Page</h3>
	<hr>
	<form:form action="${pageContext.request.contextPath}/authenticateUser" 
			   method="POST">
		<p>
			Username: <input type="text" name="username">
		</p>
		<p>
			Password: <input type="password" name="password">
		</p>
		<input type="submit" value="Login">
	</form:form>
</body>
</html>
```

Lưu ý: tên biến của username và password phải đặt như trong code trên vì đây là tên mặc định. Spring Security sẽ tự động tìm tên biến username và password.

Tới đây mọi thứ vẫn chạy bình thường nhưng cái link tới method controller /authenicateUser chưa hề có trong file LoginController.java. Tức là lúc này vẫn đang xài đường dẫn mặc định của Spring. Và để ý kĩ nữa thì sẽ thấy là lúc mình nhập sai username hay password thì sẽ không có error nào hiện ra cho biết mình nhập đúng hay sai.

Nhập tài khoản bừa nào đó và password cũng không đúng

![[Pasted image 20210701113720.png]]

Sau khi nhấn nút login

![[Pasted image 20210701113811.png]]

Để giải quyết vấn đề này, lúc login failed, Spring Security trả về trang login và một biến tên `error` chứa message trả về, nếu biến `error` tồn tại thì thì hiển thị message. Sử dụng JSTL để làm điều này

### Chức năng logout
Giải quyết vẫn như hồi bên Java EE, thay vì phải tự mình gọi hàm invalidate() của đối tượng session thì Spring Security sẽ gọi giùm mình. Dev chỉ cần config, còn lại Spring Security sẽ lo.

Cơ chế: user nhấn nút log out, invalidate() được gọi, trả về trang login kèm với parameter tên `logout`

Config cực kì đơn giản, vào đối tượng thừa kế `WebSecurityConfigurerAdapter` thêm vào config logout().

![[Pasted image 20210701180327.png]]

### Cross Site Request Forgery (CSRF)
**Vậy thì CSRF (đọc là see surf) là gì?**
- Là một kiểu tấn công bảo mật dùng một website lừa đảo để lừa bạn thực thi một chức năng nào đó trên cái web app mà bạn vừa đăng nhập. Nói nôm na dễ hiểu nhất là lừa user đăng nhập hay sử dụng một chức năng sau đó redirect đến một trang web khác và đánh cắp thông tin của user.

Ví dụ: Khi bạn đăng nhập vào một web bán hàng, bạn bị lừa mua những món đồ mà bạn không muốn

**Làm thế nào để bảo vệ user khỏi CSRF?**
- Nhúng thêm các dữ liệu hoặc token xác thực vào tất cả HTML Form.
- Trước khi các request thực sự được xử lí, web app sẽ verify các dữ liệu hay token

May mắn là chế độ bảo vệ CSRF của Spring Security luôn được bật (mặc định). Cơ chế bảo mật như sau: Spring Security sử dụng Synchronizer Token Pattern, mỗi request luôn kèm theo 1 session cookie và một token sinh ra ngẫu nhiên, Spring Security sẽ verify các những token này trước khi request được xử lí. Tất cả đều được xử lí bởi *Spring Security Filters*

**Làm thế nào để sử dụng CSRF protection?**
- Như đã nói ở trên, CSRF protection luôn được bật thông qua việc dev sử dụng thẻ `form:form`.

**Nếu dev không sử dụng thẻ form:form thì sao?**
- Cũng được luôn nhưng trong thẻ form thường phải có thêm 1 thẻ input theo cú pháp định trước của Spring Security.

```html
<form action="..." method="POST">
	<input hidden name="${_csrf.parameterName}"
		   value="${_csrf.token}">
	...
</form>

<!-- hoặc -->
<form:form action="..." method="POST">
	
	...
</form:form>
```

**Sẽ ra sao nếu không có CSRF token?**

Kết quả là hình bên dưới
![[Pasted image 20210701182312.png]]

Server hiểu request mà user gửi đến nhưng sẽ không xử lí vì server đã check tham số _csrf.token có kết quả là null

**Đây là kết quả khi sử dụng thẻ form:form**
![[Pasted image 20210701182819.png]]
### Phân quyền timeee
Spring Security cung cấp thẻ JSP riêng cho việc truy cập đến user id và role của user. Để sử dụng được thẻ JSP do Spring Security cung cấp thì phải cấu hình trong file pom.xml
```xml
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-taglibs</artifactId>
	<version>${springsecurity.version}</version>
</dependency>

```

và cả trong trang jsp
```html
<%@ taglib prefix="security" 
	uri="http://www.springframework.org/security/tags" %>

```

Cách truy xuất như sau ở trang home:
```html
User: <security:authentication property="principal.username" />

Roles(s): <security:authentication 
			property="principal.authorities" />
```

Lưu ý: khi display ra Spring Security sẽ tự động thêm tiền tố ROLE_ khi display trên browser. Nhưng không cần phải lo, tiền tố này có thể config được. Ngoài ra, một user cũng có thể có nhiều role.

##### Hạn chế truy cập dựa trên role của user
Phân quyền bằng cách thêm config trong hàm configure(HttpSecure) của đối tượng `WebSecurityConfigurerAdapter`. Spring Security cung cấp sẵn các hàm hỗ trợ để xử lí phân quyền như là: antMatchers(), hasRole()

Giải thích đã có trong hình
![[Pasted image 20210702135937.png]]

Nhưng nếu user không có role mà cố tình access thì Spring Security sẽ bắn ra lỗi 403 Forbidden. Vì vậy dev chúng mình phải có 1 trang access denied để bắt lỗi này. Tất cả đều bắt nguồn từ method `configure(HttpSecurity)`

Xử lí bằng 2 dòng cuối
![[Pasted image 20210702141415.png]]

##### Hạn chế context dựa vào role
Sử dụng thẻ <security:authorize> để hiển thị. Còn bên C# thì check xem trong principle của user vừa đăng nhập có role này không bằng câu lệnh if else thông thường.

```html
<security:authorize access="hasRole('MANAGER')">

	...
</security:authorize>
```

### Thêm JDBC Database Authentication để lấy user account từ database
Các bước thực hiện:
![[Pasted image 20210702143505.png]]

##### Từ bước 1 đến 3 rất ez nên sẽ bắt đầu từ bước 4
Spring Security cung cấp trước (pre-define) 2 bảng mặc định cho việc login đó là bảng `users` và bảng `authorities == roles`

![[Pasted image 20210702143858.png]]

Để có thể override lại 2 bảng này do chính dev viết thì phải viết đúng chính xác tên bảng lẫn tên cột. Và trên hình là các cột tối thiểu **(bắt buộc phải có)** trong 2 table trên.

**Lưu ý**:
- Trong Spring Security 5, password có thể được lưu dưới dạng sau `{id}password-của-người-dùng`. Ví dụ như {noop}123456789 hay {bcrypt}abcdefgh. Các id ở đây chính là các prefix để Spring Security biết được là password nào sẽ dùng thuật toán BCrypt để hash hay không có operation nào xảy ra cả (noop)
- Cần phải config thêm mysql connector và C3P0 trong file pom.xml

**Khai báo DataSource để trong file spring config**
```java
package me.huypc.springsecurity.config;

import java.beans.PropertyVetoException;
import java.util.logging.Logger;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

import com.mchange.v2.c3p0.ComboPooledDataSource;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "me.huypc.springsecurity")
//Đọc file properties
@PropertySource("classpath:persistence-mysql.properties")
public class AppConfig {
	
	//Thêm đối tượng Environment để đọc file .properties
	@Autowired
	private Environment env;
	
	//thêm cả logger
	private Logger logger = Logger.getLogger(getClass().getName());
	
	//Phải có ViewResolver để render ra view
	@Bean
	public ViewResolver viewResolver() {
		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
		//url: /WEB-INF/view/<tên trang>.jsp
		viewResolver.setPrefix("/WEB-INF/view/");
		viewResolver.setSuffix(".jsp");
		return viewResolver;
	}
	
	//khai báo bean cho datasource
	@Bean
	public DataSource getDataSource() {
		
		//Tạo connection pool
		ComboPooledDataSource ds = new ComboPooledDataSource();
		
		//set jdbc driver class
		try {
			ds.setDriverClass(env.getProperty("jdbc.driver"));
		} catch (PropertyVetoException e) {
			throw new RuntimeException(e);
		}
		
		//log connection properties
		logger.info(">>>> jdbc.url=" + env.getProperty("jdbc.url"));
		logger.info(">>>> jdbc.user=" + env.getProperty("jdbc.user"));
		
		//set database connection properties
		ds.setJdbcUrl(env.getProperty("jdbc.url"));
		ds.setUser(env.getProperty("jdbc.user"));
		ds.setPassword(env.getProperty("jdbc.password"));
		
		//set connection pool properties
		ds.setInitialPoolSize(
				getIntProperty(env.getProperty("connection.pool.initialPoolSize"))
				);
		ds.setMinPoolSize(
				getIntProperty(env.getProperty("connection.pool.minPoolSize"))
				);
		ds.setMaxPoolSize(
				getIntProperty(env.getProperty("connection.pool.maxPoolSize"))
				);
		ds.setMaxIdleTime(
				getIntProperty(env.getProperty("connection.pool.maxIdleTime"))
				);
		
		
		return ds;
	}
	
	private int getIntProperty(String value) {
		int result = Integer.parseInt(value);
		return result;
	}
	
}
```

**Config trong file security config**
```java
package me.huypc.springsecurity.config;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

	//Tiêm datasource vào đây
	@Autowired
	private DataSource datasource;
	
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		// Thêm user của mình khi khởi động chạy ứng dụng
		//code cứng
//		UserBuilder userBuilder = User.withDefaultPasswordEncoder();
//		
//		auth.inMemoryAuthentication()
//			.withUser(userBuilder.username("huy").password("123").roles("ADMIN", "EMPLOYEE"))
//			.withUser(userBuilder.username("zaan").password("123").roles("EMPLOYEE", "MANAGER"))
//			.withUser(userBuilder.username("susan").password("123").roles("EMPLOYEE"));
		
		//Lấy dữ liệu từ datasource
		auth.jdbcAuthentication().dataSource(datasource);
	
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		//Dưới dây được gọi là expression, khác với C#, java sử dụng method như các câu điều kiện
		http.authorizeRequests() //hạn chế truy cập dựa trên đối tượng HttpServletRequest
				.antMatchers("/").hasRole("EMPLOYEE") //trang chính cho những user có Role là EMPLOYEE
				.antMatchers("/leaders/**") //link bắt đầu bằng /leaders và tất cả các link con
					.hasRole("MANAGER") //role manager mới truy cập được link /leader/**
				.antMatchers("/systems/**").hasRole("ADMIN") //role admin mới truy cập được /systems/**
			.and()
			.formLogin() //custom lại login form
				.loginPage("/showLoginForm") //show trang login
				.loginProcessingUrl("/authenticateUser") //method controller xử lí POST
				.permitAll()
			.and()
			.logout().permitAll()
			.and()
			.exceptionHandling()
				.accessDeniedPage("/access-denied");
	}
	
}

```

### Mã hoá password
Sử dụng thuật toán Bcrypt để mã hoá password. Sau khi mã hoá, chuỗi mã hoá luôn có độ dài mặc định là 68 vì cấu trúc của nó `{id}password-đã-mã-hoá`. Phần `{id}` chiếm 8 kí tự, phần còn lại chiếm 60 kí tự cho nên điều này dẫn đến phải sửa lại số lượng kí tự trong thuộc tính password của bảng table.

Vì Spring Security đã hỗ trợ sẵn hết nên chỉ cần encrypt password ở website hay bên backend trước, rồi add vô database với id của Spring Security được cung cấp sẵn thì mọi thứ đều đã được giải quyết

# Hibernate

### Hibernate là gì?

Là một framework nổi tiếng dành cho ngôn ngữ java, persist (bền bỉ?? Tồn tại?? Kiên trì??) và lưu các đối tượng (objects) trong 1 cơ sở dữ liệu.

Download free tại trang chủ [hibernate](http://www.hibernate.org)

### Lợi ích của Hibernate là gì?

- Xử lí tất cả các truy vấn SQL bậc thấp

- Giảm tải lượng sql code của JDBC

- Cung cấp mô hình Object-to-Relational Mapping (ORM)

 - Mô hình ORM: mapping 1 đối tượng java vào 1 bảng nằm trong cơ sở dữ liệu (giống như dapper và Entity Framework Core của .NET)

  

**Mối quan hệ giữa Hibernate và JDBC**

Về logic xử lí, Hibernate vẫn sử dụng JDBC để kết nối đến database, cho nên khi sử dụng Hibernate trong project java thì có nghĩa là sử dụng Hibernate APIs được cung cấp sẵn. Hibernate làm thay hết tất cả các công việc của dev.

### Các phần mềm cần thiết để xây dựng Hibernate Application

1. IDE (Eclipse)

2. Database Server (MySQL)

3. Hibernate JAR files và JDBC Driver

**Lưu ý: Vì đây là file kiến thức nên sẽ tập trung vào lý thuyết nhiều hơn là coding.**

  

### Làm việc với Hibernate (DEMO)

**Tạo sẵn database**

Copy những dòng dưới và thực thi trong database

```sql

CREATE USER 'hbstudent'@'localhost' IDENTIFIED BY 'hbstudent';

  

GRANT ALL PRIVILEGES ON * . * TO 'hbstudent'@'localhost';

#

# Starting with MySQL 8.0.4, the MySQL team changed the 

# default authentication plugin for MySQL server 

# from mysql_native_password to caching_sha2_password.

#

# The command below will make the appropriate updates for your user account.

#

# See the MySQL Reference Manual for details: 

# https://dev.mysql.com/doc/refman/8.0/en/caching-sha2-pluggable-authentication.html

#

ALTER USER 'hbstudent'@'localhost' IDENTIFIED WITH mysql_native_password BY 'hbstudent';

```

```sql

CREATE DATABASE IF NOT EXISTS `hb_student_tracker`;

USE `hb_student_tracker`;

  

--

-- Table structure for table `student`

--

  

DROP TABLE IF EXISTS `student`;

  

CREATE TABLE `student` (

 `id` int(11) NOT NULL AUTO_INCREMENT,

 `first_name` varchar(45) DEFAULT NULL,

 `last_name` varchar(45) DEFAULT NULL,

 `email` varchar(45) DEFAULT NULL,

 PRIMARY KEY (`id`)

) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;

  

```

**Setup project**

Đầu tiên tạo 1 project java application (console), sử dụng hệ cơ sở dữ liệu nào cũng được, phụ thuộc vào jdbc url đã khai báo.

Chuột phải vào project, tạo 1 folder tên lib, copy toàn bộ thành file jar nằm trong folder required trong folder `hibernate-release` và `mysql-connector-java.jar` đã download từ trang chủ.

Sau khi copy, chuột phải vào project, chọn **properties**, nhìn sang trái của menu mới hiện lên chọn **Java Build Path**, chọn tab **libraries** (nếu chưa nằm ở tab này), tiếp đến chọn **Add jars**, lúc này sẽ hiện lên 1 popup để tìm, đè shift chuột trái chọn hết mọi thành phần trong folder **lib** rồi OK và Apply.

**Test thử xem đã kết nối được chưa**

Chuột phải vào folder **src**, **New** > **Package**, tạo tên package theo ý mình hoặc theo phong cách maven.

Chuột phải vào package vừa tạo, **New** > **Class**, đặt tên là TestJdbc.java với nội dung bên dưới.

```java

package me.mavis.jdbc;

  

import java.sql.Connection;

import java.sql.DriverManager;

  

public class TestJdbc {

  

 public static void main(String[] args) {

 // TODO Auto-generated method stub

 //câu lệnh kết nối tới mysql

 String jdbcUrl = "jdbc:mysql://localhost:3306/hb_student_tracker?useSSL=false&serverTimezone=UTC";

 String user = "hbstudent";

 String pass = "hbstudent";

 try {

 System.out.println("Connecting to database: " + jdbcUrl);

 Connection con = DriverManager.getConnection(jdbcUrl, user, pass);

 System.out.println("Connection successfully");

 } catch (Exception e) {

 e.printStackTrace();

 }

 }

  

}

```

Chạy thử nếu không lỗi gì tức là đã **kết nối đến database thành công**

**Tạo file configuration của Hibernate**

Như đã nói ở trên, Hibernate sử dụng jdbc để giao tiếp với database nên trong file config sẽ khai báo/ cấu hình để Hibernate biết cần kết nối như thế nào

Tạo 1 file tên là `hibernate.cfg.xml` cùng cấp với package với nội dung bên dưới

```xml

<!DOCTYPE hibernate-configuration PUBLIC

 "-//Hibernate/Hibernate Configuration DTD 3.0//EN"

 "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

  

<hibernate-configuration>

  

 <session-factory>

  

 <!-- JDBC Database connection settings -->

 <property name="connection.driver_class">com.mysql.cj.jdbc.Driver</property>

 <property name="connection.url">jdbc:mysql://localhost:3306/hb_student_tracker?useSSL=false&amp;serverTimezone=UTC</property>

 <property name="connection.username">hbstudent</property>

 <property name="connection.password">hbstudent</property>

  

 <!-- JDBC connection pool settings ... using built-in test pool -->

 <property name="connection.pool_size">1</property>

  

 <!-- Select our SQL dialect -->

 <property name="dialect">org.hibernate.dialect.MySQLDialect</property>

  

 <!-- Echo the SQL to stdout -->

 <property name="show_sql">true</property>

  

 <!-- Set the current session context -->

 <property name="current_session_context_class">thread</property>

 </session-factory>

  

</hibernate-configuration>

```

  

**Map đối tượng java vào table**

Có 2 cách để làm việc này:

1. Config trong file xml (truyền thống)

2. Sử dụng Java Annotation (hiện đại)

  

Nhưng hiện tại nhà nhà đều xài cách 2 nên sẽ tập trung vào cách 2 nhiều hơn

  

Gồm có 2 bước:

1. Map java class thành table trong database

Ví dụ:

```java

@Entity //để hibernate biết đây là entity class

@Table(name="student") //Tên bảng trong database

public class Student {

  

}

```

2. Map các thuộc tính của class thành cột trong database

Ví dụ:

```java

@Entity 

@Table(name="student")

public class Student {

  

 @Id //đánh dấu đây là khoá chính

 @Column(name="id") //tên bảng

 private int id;

  

 @Column(name="first_name")

 private String firstName;

}

```

  

Sau khi map hết 2 bước trên sẽ có file Student.java hoàn chỉnh như bên dưới

```java

package me.mavis.hibernate.demo.entity;

  

import javax.persistence.Column;

import javax.persistence.Entity;

import javax.persistence.Id;

import javax.persistence.Table;

  

@Entity

@Table(name="student")

public class Student {

 @Id

 @Column(name="id")

 private int id;

 @Column(name="first_name")

 private String firstName;

 @Column(name="last_name")

 private String lastName;

 @Column(name="email")

 private String email;

 public Student() {

 }

  

 public Student(String firstName, String lastName, String email) {

 super();

 this.firstName = firstName;

 this.lastName = lastName;

 this.email = email;

 }

  

 public int getId() {

 return id;

 }

  

 public void setId(int id) {

 this.id = id;

 }

  

 public String getFirstName() {

 return firstName;

 }

  

 public void setFirstName(String firstName) {

 this.firstName = firstName;

 }

  

 public String getLastName() {

 return lastName;

 }

  

 public void setLastName(String lastName) {

 this.lastName = lastName;

 }

  

 public String getEmail() {

 return email;

 }

  

 public void setEmail(String email) {

 this.email = email;

 }

  

 //dành cho mục debugging

 @Override

 public String toString() {

 return "Student [id=" + id + ", firstName=" + firstName + ", lastName=" + lastName + ", email=" + email + "]";

 }

}

```

Tại sao lại phải dùng JPA annotation thay vì org.hibernate.annotations ?

- JPA là 1 chuẩn chung, Hibernate implements toàn bộ các JPA annotations.

**Trước khi bắt đầu code first thì phải phân biệt được 2 lớp `SessionFactory` và `Session`**

**SessionFactory**
- Đọc file hibernate.cfg.xml
- Tạo kết nối đến database
- Tạo là 1 đối tượng `Session` để sử dụng
- Chỉ khởi tạo 1 lần khi chạy nên có thể tái sử dụng

**Session**
- Là đối tượng chứa JDBC connection
- Là đối tượng thực thi hành động CRUD
- Là đối tượng short-lived tức là xài xong dispose, muốn xài nữa phải new mới
- New đối tượng session bằng cách sử dụng phương thức nằm trong SessionFactory

### Mapping trong Hibernate
Như cơ bản, có 3 loại quan hệ:
1. One-to-One
2. One-to-Many hay Many-to-One
3. Many-to-Many

##### Cascade operation

Khi có tác động ở table cha thì ở table con cũng bị tác động.

Khi thực hiện hành động thêm xoá sửa ở table cha thì hành động đó cũng sẽ được áp dụng cho table con. Nhưng ngược lại khi tác động ở table con thì table cha thì không

Nhưng vấn đề nghiêm trọng sẽ phát sinh nếu xoá một bảng có mối quan hệ Many-to-Many với bảng khác.

Giả sử giữa bảng Course và bảng Student, nếu xoá đi 1 row trong Course thì sẽ xoá luôn các phần tử có liên quan bên Student. Như vậy là toang.

**Uni-Directional**

Mối quan hệ một chiều

Giả sử lớp A và B, biết được 1 đối tượng của A thì sẽ biết được đối tượng của B nhưng ngược lại thì không.

**Bi-Directional**

Mối quan hệ 2 chiều

Giả sử lớp A và B, chỉ cần biết được 1 trong 2 thì sẽ biết cả 2.

###### Entity lifecycle
Entity lifecycle là gì?
- Là một tập hợp các trạng thái mà các Hibernate entity (thực thể) có được khi coding

Gồm có các state như sau:
- **Detach:** Nếu thực thể bị detached (tách ra), nó sẽ không được liên kết với hibernate session
- **Merge:** Nếu instance (đối tượng) bị tách ra từ session thì merge sẽ attach (buộc | dính chặt) vào session
- **Persist:** Chuyển trạng thái của các instance mới vào trạng thái được quản lý. Lượt commit kế tiếp sẽ lưu vào database
- **Remove:** Chuyển trạng thái của thực thể được quản lý cần xoá. Lượt commit kế tiếp sẽ xoá khỏi database
- **Refresh:** Reload hoặc đồng bộ lại dữ liệu từ database. Ngăn chặn việc dữ liệu bị lỗi thời

Ngoài ra các trạng thái này còn áp dụng cho các annotation mapping như @OneToOne, @OneToMany, @ManyToMany

##### @OneToOne
Mối quan hệ 1 - 1  thường hay xuất hiện trong cơ sở dữ liệu. Điều này là không thể tránh khỏi. Và bên Hibernate code hơi khác bên EntityFramework Core một vài chỗ.

Giả sử có 2 lớp A và B, lớp B có khoá chính là id của lớp A (tức là B tham chiếu tới A hay còn gọi B là con của A) thì bên lớp A sẽ có 1 thuộc tính là
```java
//các thuộc tính khác
...
//ở đây sẽ bắt đầu những annotation cơ bản
@OneToOne(cascade = CascadeType.All)
//dòng dưới sẽ được xem là một cột của thực thể A
@JoinColumn(name = "b_id")
private B b;

public void setB(B b) {
	this.b = b;
}

public B getB() {
	return b;
}

//constructor/getters/setters
...
```

Lúc này sẽ hình thành uni-directional vì đây là mối quan hệ một chiều, biết A thì biết B nhưng biết B thì lại không biết A. Để trở thành bi-directional thì bên lớp B cũng config như sau:
```java
//các thuộc tính khác
...

//config bắt buộc để trở thành mối quan hệ 2 chiều
//mappedBy = "tên thuộc tính có annotation @JoinColumn"
@OneToOne(mappedBy = "b", cascade = CasecadeType.All)
private A a;

public void setA(A a) {
	this.a = a;
}

public A getA() {
	return a;
}


//constructor/getters/setters
```

Từ config trên, mỗi khi tác động một trong 2 bảng A hay B thì đều sẽ ảnh hưởng cả 2

**Trong trường hợp chỉ muốn xoá thực thể con nhưng không xoá thực thể cha**

Nếu muốn như vậy thì trong lớp B phải sửa lại cascade

```java
//các thuộc tính khác
...

//config tuỳ chọn xoá thực thể con không xoá thực thể cha
//loại cascade: detach, merge, persist, refresh
//mappedBy = "tên thuộc tính có annotation @JoinColumn"
@OneToOne(mappedBy = "b", 
	cascade ={CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST, CascadeType.REFRESH})
private A a;

public void setA(A a) {
	this.a = a;
}

public A getA() {
	return a;
}


//constructor/getters/setters
```
À mà khoan, dừng khoảng chừng là 2 giây. Sau khi đoạn code này thực thi, nếu xoá 1 đối tượng B nhưng bên A thì sao. Chắc chắn sẽ bắn lỗi `EntityNotFoundException` vì `@JoinColumn` bên lớp A không biết phải tìm ai để map -> bị gãy bi-directional link. cho nên trước khi xoá thực thể B thì phải set thuộc tính B của đối tượng A là `null` thì mới không bị lỗi nữa.


##### @OneToMany
**A có nhiều B**

Mối liên kết là bi-directional. Thêm nữa, `@OneToMany` và `@ManyToOne` giống nhau, chỉ là ngược nhau. `@OneToMany` nằm ở lớp cha, `@ManyToOne` nằm ở lớp con

Trong dự án thực tế, nếu xoá thực thể con thì không xoá thực thể cha, nhưng nếu xoá thực thể cha thì sẽ xoá hết tất cả thực thể con -> thực thể con không áp dụng cascade delete.

Ví dụ code bên dưới
Lớp B:
```java

//Các field khác

@ManyToOne
@JoinColumn(name = "a_id")
private A a;

public void setA(A a) {
	this.a = a;
}

public A getA() {
	return a;
}

//constructor / getters / setters
```

Lớp A:
```java
//các field khác

//các loại cascade này để khi nào có xoá một đối tượng B
//thì A không bị xoá
@OneToMany(mappedBy = "a",
			cascade = {CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST, CascadeType.REFRESH})
private List<B> bs;

public void setBs(List<B> bs) {
	this.bs = bs;
}

public List<B> getBs() {
	return bs;
}

//constructor / getters / setters
```


##### @ManyToMany
Để lưu trữ được mối quan hệ nhiều nhiều, sẽ phát sinh thêm 1 table mà trong hibernate được gọi là JoinTable, cung cấp mapping giữa 2 bảng.

Ví dụ giữa User và Product, JoinTable sẽ là ShoppingCart. Cách config như sau:
- Bên lớp User
	```java
	public class User {
		...
		
		@ManyToMany
		@JoinTable(
			name = "ShoppingCart",
			//Tên column phải dùng với cột trong database
			joinColumns = @JoinColumn(name = "userId"),
			inverseJoinColumns = @JoinColumn(name = "productId")
		)
		private List<Product> products;
		
		//getter và setter
		
		...
	}
	```
- Bên lớp Product
	```java
	public class Product {
		...
		
		@ManyToMany
		@JoinTable(
			name = "ShoppingCart",
			//Do mình đang đứng ở lớp Product nên joinColumns sẽ là productId
			//còn inverse là chỉ cột của đối tượng còn lại
			joinColumns = @JoinColumn(name = "productId"),
			inverseJoinColumns = @JoinColumn(name = userId)
		)
		private List<User> users;
		
		//getter và setter
		
		...
	}
	```
	
Annotation `@JoinTable` nói với Spring rằng là hãy nhìn vào cột `userId` và cột `productId` trong bảng ShoppingCart để tìm ra mối quan hệ giữa đối tượng User và Product

Từ khoá `inverse` có thể hiểu là khi mình config trên lớp A, `joinColumns` trỏ về 1 thuộc tính của bảng A, còn `inverseJoinColumns` thì trỏ về thuộc tính của bảng B có mối quan hệ nhiều nhiều với bảng A.

Lưu ý: Thực thể phụ sẽ không có 1 class cụ thể trong project của mình.

Bản chất của việc xoá: Xoá mối *quan hệ của thực thể* (Thực thể phụ) rồi mới xoá thực thể chính

##### Fetch Types
**Các kiểu lấy dữ liệu: *Eager* và *Lazy***

- Kiểu **Eager** sẽ lấy TẤT CẢ dữ liệu
- Kiểu **Lazy** sẽ lấy khi nào có request

**Về Eager**

Lấy toàn bộ các thực thể phụ thuộc trong 1 lần load. Có thể làm giảm performance trong application. Giả sử nếu search thì sao? Cứ mỗi lần load là lấy hết tất cả các thực thể phụ thuộc, như vậy sẽ làm cho trang web của mình chậm thấy rõ. 

**Về Lazy**

Lấy thực thể chính trước. Chỉ lấy thực thể phụ thuộc khi có yêu cầu từ client. Tuy nhiên, điều này yêu cầu phải mở 1 Hibernate session - tức là cần phải có connection tới database để lấy dữ liệu. Khi session đã đóng và muốn lấy dữ liệu thì Hibernate sẽ tự động ném ra `LazyInitializationException`.

Giải quyết vấn đề `LazyInitializationException`:
- Cách 1: Gọi hàm getter để lấy trước khi đóng session. như vậy dữ liệu đã được load về nên khi đóng gọi lại hàm getter sẽ không báo lỗi.
- Cách 2: Sử dụng HQL. Sử dụng từ khoá `JOIN FETCH` để lấy dữ liệu. Ví dụ như câu lệnh HQL bên dưới.
	```sql
	SELECT i FROM Instructor i 
	JOIN FETCH i.courses
	WHERE i.id = :instructorId
	```
	Giải thích: tất cả đều dựa vào thuộc tính của class và tên class, không hề liên quan đến tên table và các column. `:instructorId` là tham số của câu lênh HQL

**Cách sử dụng trong thực tế**

Dùng lazy loading khi ở trang index hay những trang master (những trang có nhiều danh mục)

Dùng eager loading khi ở trang detail.

**Cách kiểu mặc định của fetch types**
- `@OneToOne`: EAGER
- `@OneToMany`: LAZY
- `@ManyToOne`: EAGER
- `@ManyToMany`: LAZY

### Khi đã gộp với web thì...
Bên dưới là đoạn code thủ công của hibernate khi
```java
	// tạo 1 SessionFactory
	SessionFactory factory = new Configuration()
							 .configure("hibernate.cfg.xml")
							 .addAnnotatedClass(Student.class)
							 .buildSessionFactory();
	//Tạo 1 Session (phiên làm việc)
	Session session = factory.getCurrentSession();

	try {
		//Tạo 1 đối tượng Student
		System.out.println("Creating a new student object...");
		Student student = new Student("Za", "An", "huypc2410@gmail.com");

		//bắt đầu transaction
		session.beginTransaction();

		//lưu đối tượng Student vào database
		System.out.println("Saving the student...");
		session.save(student);

		//commit transaction
		session.getTransaction().commit(); // == _db.SaveChanges();

		System.out.println("Done!");
	}
	catch (Exception e) {
		e.printStackTrace();
	}
	finally {
		session.close();
		factory.close();
	}
```

Chuyển sang web, để tiến hành DI trong project, thì ngay lớp repository chỉ cần thêm `@Transactional` vào method là spring sẽ tự động tạo SessionFactory và tự động commit khi hoàn thành
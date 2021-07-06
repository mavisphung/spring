##### Các bước để tạo Controller và view
1. Tạo 1 lớp với tên + suffix Controller (HomeController) như trong C# và thêm @Controller vào đầu lớp
	-	Do @Controller được kế thừa từ @Component nên hỗ trợ tự động scan
2. Tạo methods (return String) trong controller, các method này được gọi là controller method, có thể đặt tên bất kỳ vì thằng Spring khá là flexible
3. Thêm @RequestMapping("/") vào method mặc định (giống như trang Index trong C#) của controller
4. Giá trị trả về là tên của view

##### Cách truyền tham số từ View về Controller và ngược lại

Có 2 cách để làm điều này
1.  Tiêm đối tượng `HttpServletRequest` và `Model` và controller method
2.  Sử dụng annotation @RequestParam(String) với tham số truyền vào là 1 attibute của đối tượng `Model`

**Cách 1:** Tiêm đối tượng `HttpServletRequest` và `Model` và controller method

```java
// xử lí trang form bằng HttpServletRequest và Model
@RequestMapping("/processFormVersionTwo")
public String processVersion2(HttpServletRequest request, Model model) {
	// Đọc dữ liệu từ request
	String username = request.getParameter("username");

	// Tạo câu thông báo
	String result = "Yo! " + username;

	// Thêm vào attribute của model
	model.addAttribute("username", result);

	return "process-name";
}
```

**Cách 2:** Sử dụng annotation @RequestParam(String)

```java
// Xử lí trang form giống như Version2 nhưng không dùng request mà dùng thẳng
// tham số, binding bằng @RequestParam
@RequestMapping("/processFormVersionThree")
public String processVerion3(@RequestParam("username") String username, Model model) {

	// Tạo câu thông báo
	String result = "Yo! " + username;

	// Thêm vào attribute của model
	model.addAttribute("username", result);

	return "process-name";
}
```

##### Data binding
Sử dụng MVC Form tag được hỗ trợ sẵn chuyên dành cho trang jsp của spring
`<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>`.

Ví dụ cho taglib (giống như TagHelper của C#)
```html
<!-- action là link tới controller method xử lí,
	modelAttribute là tên của attribute để binding dữ liệu
 -->
<form:form action="processForm" modelAttribute="student">
	First name: <form:input path="firstName"/>
	<br> <br>
	Last name: <form:input path="lastName"/>
	<br> <br>
	<input type="submit" value="Submit">
</form:form>
```

*Giải thích:* 
- action là link dẫn tới controller method xử lí 
- modelAttribute là tên của attribute để binding dữ liệu
- path là thuộc tính của đối tượng java (bean), phải viết đúng tên field thì mới binding được

Ngoài ra còn có các thẻ form hay xài như là form:select (dùng có droplist), form:radiobutton (chọn 1 trong nhiều), form:checkbox

Để có thể binding đúng thì khi khởi tạo trang form phải gán một đối tượng được new và thêm đối tượng đó vào attribute của đối tượng Model. Lúc này, đối tượng sẽ được xác định bằng key của attribute.

##### Form Validation (quen thuộc bên C#)
- Từ khoá: Bean Validation API, Hibernate Validator

Sử dụng *Hibernate Validator 6.2* thay cho phiên bản 7 vì từ phiên bản 7 trở lên package javax đã được đổi lại thành jakarta nên không còn phù hợp với Spring 5

Link: [Tải Hibernate Validator tại đây](http://hibernate.org/validator/)

Một số annotation thông dụng cho việc validate
- @NotNull: Kiểm tra thuộc tính (field) được đánh dấu có null hay không
- @Min: kiểm tra có lớn hơn giá trị của field >= tham số truyền vào
- @Max: kiểm tra có lớn hơn giá trị của field <= tham số truyền vào
- @Size: kích thước phải bằng kích thước cho trước (nvarchar(64))
- @Pattern: phải match với regex truyền vào
- @Future / @Past: Kiểm tra Date before hay after so với tham số
- @Valid: dùng cho tham số, để check xem đối tượng có thoả điều kiện không

Ví dụ bên dưới
```java
@RequestMapping("/processForm")
public String processData(@Valid @ModelAttribute("customer") Customer customer, BindingResult results) {
	if (results.hasErrors()) {
		return "customer-form";
	}
	return "customer-confirm";
}
```

- `@Valid` để kiểm tra xem bên model đã thoả mãn điều kiện không
- `@ModelAttribute("customer")` để binding attribute có tên customer vào đối tượng Customer
- Tham số `BindingResult` luôn đi sau đối tượng (điều này được quy định bởi Spring). Nếu đặt trước có thể bị gây lỗi

**Lưu ý về khoảng trắng**

Đây là một trong những lỗi cần chú ý khi mà `@NotNull` và `@Size` không bắt được lỗi này. Do đó sẽ có một cách khác để xử lý vấn đề này. Đó là sử dụng đối tượng `StringTrimmerEditor` để apply toàn bộ đối tượng String không được chứa khoảng trống. Khi 1 đối tượng String hoàn toàn là khoảng trắng thì sẽ trim tới khi null - tức là trả về null.

Ví dụ:
```java
@InitBinder
public void initBinder(WebDatabBinder dataBinder) {
	
	StringTrimmerEditor editor = new StringTrimmerEditor(true);
	
	dataBinder.registerCustomerEditor(String.class, editor);
}
```

- `@InitBinder` là một annotation của phương thức, đánh dấu cho compiler biết rằng đây là một phương thức tiền xử lý. Nó sẽ xử lý các web request gửi đến controller trước khi controller thực thi một phương thức nào đó.

- Phương thức ở trên được hiểu là đối tượng StringTrimmerEditor sẽ xử lý toàn bộ các đối tượng kiểu String trước khi controller bắt đầu direct hay redirect

- Đối tượng StringTrimmerEditor là của bọn Spring framework, java core không có cái này

**Custom Message Error**

Tạo thêm 1 folder trong file src với tên là *resouces*, trong folder resources tạo thêm 1 file là messages.properties với nội dung như sau:
```properties
typeMismatch.customer.freePasses=Invalid number format.
```

Cái này buộc phải đúng cú pháp tên lỗi (đọc thêm loại lỗi trong docs của Spring) để đối tượng  `ResourceBundleMessageSource` của thằng Spring có thể tìm thấy giá trị. Cú pháp như sau:
```properties
loại-lỗi.tên-lớp.thuộc-tính=giá-trị-nào-dó
```

*Làm thế nào để biết được cú pháp lỗi dễ hơn?*
- In đối tượng BindingResult ra console để xem code, và dùng code này làm key trong file .properties

###### Custom Form validation
Đại khái là sẽ tự làm ra annotation của mình dể validate. Sẽ quay lại phần này sau vào một ngày không xa :D

##### Các kiểu truyền dữ liệu
**GetMapping**

Là một short-cut annotation thay thế cho `@RequestMapping` từ bản Spring 4.3 trở lên, hoạt động tượng tự như `@RequestMapping` nhưng chỉ cho phép các request có method **GET**

Ví dụ:
```java
@RequestMapping(path = "/process", method = RequestMethod.GET)
public String process(...) {
...
}

//hoặc

@GetMapping("/process")
public String process(...) {
...
}
```

**PostMapping**

Giống như `@GetMapping` nhưng chỉ cho phép các request có method **POST**. Mô hình như sau:

![[POST.png]]


**Điểm khác biệt giữa GET và POST**

*GET*:
- Dễ debug.
- Có thể đánh dấu bookmark hoặc url
- Giới hạn độ lớn của dữ liệu

*POST*:
- Không thể đánh dấu bookmark hay url
- Không có giới hạn độ lớn của dữ liệu
- Có thể gửi data dạng nhị phân

##### Service Facade design pattern (a.k.a Service layer)
Chen giữa tầng controller và tầng repository sẽ có 1 tầng service. Thiết kế này được gọi là Service Facade design pattern. Tầng này chính là tầng giải quyết các business logic chứ không phải controller như Java Servlet hay làm. Có thể tích hợp nhiều DAO hay repositories trong đó.

![[service-facade.png]]

##### @Service annotation

![[components-in-spring.png]]

`@Service` là một lớp con của `@Component`, nên Spring sẽ tự động đăng ký vào DI như repositories nhờ vào component-scanning. Khi đã chuyển sang **Service Facade** design pattern thì sẽ không dùng `@Transactional` ở method của repository nữa mà sẽ chuyển hết toàn bộ `@Transactional` ở method của đối tượng `@Service`.

![[service-implementation.png]]

##### Tại sao phải chia thành service layer để làm cái khỉ gì?
Giả sử thiết kế phần mềm cho một nhà bank, trong thiết kế có một đối tượng BankRepository với 2 method deposit(...), withdraw(...). Nếu để `@Transactional` ở bên BankRepository, cứ mỗi lần gọi method của đối tượng repository thì sẽ tạo 1 transaction mới hoàn toàn, nếu có lỗi ở 1 trong 2 thì không thể rollback cả 2 mà chỉ rollback một trong hai. Sẽ dẫn đến trường hợp chuyển tiền thành công nhưng bên kia lại không nhận được tiền. Vì thế để đồng bộ trong 1 transaction, tầng Service được sinh ra để giải quyết vấn đề này vì lúc này, cả deposit(...) và withdraw(...) đều nằm trong 1 transaction. Thích lỗi gì cũng đều có thể rollback được.


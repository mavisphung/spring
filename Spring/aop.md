### Lập trình hướng khía cạnh là cái quái gì?
Giải thích AOP thông qua một câu chuyện:

Như mọi project bình thường khác, mọi dev thường sẽ làm theo kiến trúc như ảnh bên dưới

![[service-structure.png]]

Kiến trúc cơ bản nhất là chia controller, service, repository thành 3 layer khác nhau, Controller dùng để direct url, service dùng để xử lí nghiệp vụ (business logic), repository dùng để thao tác với database.

Trong một hàm thêm account vào db ở đối tượng repository bên dưới.
```java
public void add(Account entity) {
	Session session = sessionFactory.getCurrentSession();
	session.save(entity);
}
```

Bây giờ bỗng nhiên có requirement mới:
1. Thêm logging vào hàm add
2. Thêm security vào hàm add


```java
public void add(Account entity) {
	//logging code ở đây
	...
	
	//security code ở đây
	...
	
	//xử lí business logic
	Session session = sessionFactory.getCurrentSession();
	session.save(entity);
}
```

Việc đơn giản nhất đó chính là copy code logging và security từ 1 project khác đem qua, và bùm

![[Pasted image 20210625115051.png]]

Mọi layer trong ứng dụng đều có code logging và security, và lúc này sẽ sinh ra technical debt (nợ kỹ thuật). Code sẽ trở nên rất dơ, nếu trong ứng dụng có tận 1000 class thì vấn đề copy code sẽ gây ra phiền phức rất lớn khi refactor.

Lúc này AOP được sinh ra để giải quyết vấn đề này. **Khía cạnh** ở đây chính là vấn đề logging và security. Thay vì chèn code vào trong đối phương thức add như ở trên thì nguyên lý AOP sẽ biến những code logging và security thành các lớp có thể tái sử dụng. Và mục đích AOP sinh ra chính là có thể tái sử dụng ở nhiều layer khác nhau, một khía cạnh có thể được coi là 1 lớp.

![[Pasted image 20210625120541.png]]

**Vậy AOP là cái quái gì?**

Aspect Oriented Programming (AOP) – lập trình hướng khía cạnh: là một kỹ thuật lập trình (kiểu như lập trình hướng đối tượng) nhằm phân tách chương trình thành cách moudule riêng rẽ, phân biệt, không phụ thuộc nhau.

Khi hoạt động, chương trình sẽ kết hợp các module lại để thực hiện các chức năng nhưng khi sửa đổi 1 chức năng thì chỉ cần sửa 1 module.
### Áp dụng AOP như thế nào trong project
Sử dụng **proxy design pattern**. Trước khi áp dụng thì phải tìm hiểu proxy pattern là gì đã.

Ví dụ khi bạn gọi điện cho một người bạn khác, về phía client, bạn gọi, người kia trả lời. Nhưng về phía server, Khi bạn gọi, cuộc gọi sẽ được quản lý bởi tổng đài, lúc này sẽ thông qua một server, server này sẽ ghi lại vào trong log (logging) và mã hoá (security) những âm thanh của phía bạn và gửi đến địa chỉ (số điện thoại) của người nhận cuộc gọi.

**Vậy proxy pattern là gì?**

Proxy pattern là một design pattern mà ở đó tất cả các truy cập trực tiếp đến một đối tượng nào đó sẽ được chuyển hướng vào một đối tượng trung gian (Proxy Class). Mỗi Proxy (người đại diện) đại diện cho một đối tượng khác thực thi các phương thức, phương thức đó có thể được định nghĩa lại cho phù hợp với mục đích sử dụng.

Proxy Pattern có những đặc điểm chung sau đây:
- Cung cấp mức truy cập gián tiếp vào một đối tượng.
- Tham chiếu vào đối tượng đích và chuyển tiếp các yêu cầu đến đối tượng đó.
- Cả Proxy và đối tượng đích đều kế thừa hoặc thực thi chung một lớp giao diện. Mã máy dịch cho lớp giao diện thường “nhẹ” hơn các lớp cụ thể và do đó có thể giảm được thời gian tải dữ liệu giữa server và client


**Lợi ích của AOP**
- Code của một khía cạnh được định nghĩa trong 1 lớp, tốt hơn là copy code ở nhiều nơi khác nhau, dễ dàng sửa và tái sử dụng.
- Code xử lí nghiệp vụ (business logic) sẽ clean hơn, giảm thiểu độ phức tạp của source code
- Dễ dàng config, tách riêng hoàn toàn code của khía cạnh với code của ứng dụng.

**Nhược điểm**
- Quá nhiều aspects dẫn đến làm chậm ứng dụng
- Làm giảm performance cho việc thực thi khía cạnh


**Cách trường hợp sử dụng nhiều nhất**
- Thông thường nhất chính là ghi logging, security và transactions
- Audit logging - ghi lại thanh toán
	- Ai, lúc nào, ở đâu, cái gì
- Xử lí lỗi
	- Ghi exception ra log rồi gửi qua cho DevOps thông qua SMS/Email
- Quản lý API
	- Ví dụ như bao nhiêu lần 1 method được gọi bởi user?
	- Phân tích dữ liệu như giờ cao điểm, trung bình lượng tải, top user....

### Spring hỗ trợ AOP
Có 2 framework nổi tiếng hỗ trợ cho AOP: Spring AOP của Spring và AspectJ của Eclipse

##### Spring AOP
**Điểm tốt**
- Dễ sử dụng hơn AspectJ
- Sử dụng proxy pattern
- Có thể migrate qua AspectJ khi sử dụng `@Aspect`

**Bất lợi**
- Chỉ hỗ trợ level method,
- Chỉ có thể apply vào các beans được khai báo trong spring context.
- Tốn hao hiệu năng lặt vặt

##### AspectJ
**Điểm tốt**
- Hỗ trợ ở bất kỳ join point nào
- Hoạt động với bất kỳ POJO nào, không nhất thiết phải là beans trong spring context
- Performance nhanh hơn Spring AOP
- Hỗ trợ toàn bộ AOP

**Bất lợi**
- Tốn nhiều thời gian để compile hơn
- Syntax phức tạp

### Ứng dụng vào real-time project
**Các loại Advice**

![[Pasted image 20210625194639.png]]

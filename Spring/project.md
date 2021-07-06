### Context Root là gì?
Nói nôm na dễ hiểu đó chính là đường dẫn root của web app.

Ví dụ: context root là my-web-app => http://localhost:8080/my-web-app

Có thể bi trùng nếu copy nguyên xi 1 project đã chạy trước đó. Cho nên để tránh trùng thì chuột phải vào project > Web Project Settings > Sửa lại context root thành tên khác

**Ngoài ra, Context Root và Context Path là một**

**Tại sao nên sử dụng Context Path (Context Root)?**
- Cho phép tham chiếu tới các path khác nhau của app (dynamically)
- Giúp cho việc liên kết các file css, js,... không bị mất
- Nếu thay đổi context path của app thì mọi link vẫn hoạt động được.
- Tốt hơn là việc code cứng.

### Các câu thần chú cần thiết.
```html
<!-- encode cho jsp -->
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>

<!-- spring form -->
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>

<!-- jstl -->
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<!-- spring security -->
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>
```
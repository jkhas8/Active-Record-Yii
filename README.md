Active Record trong Yii Framework
=================================

Mặc dù Yii DAO có thể giải quyết được hấu như mọi việc liên quan đến database, nhưng việc này làm các lập trình viên tốn đến 90% thời gian của mình dành cho các câu lệnh SQL (CURD). Thật là khó để đảm bảo code của họ khi mà phải trộng lần code với câu lênh SQL. Để giải quyết việc này, Yii đã sử dụng Active Record.

Active Record (AR) là nột kỹ thuật ORM (Object-Relational Mapping) phổ biến. Mỗi lớp AR thay mặt cho một bảng trong database. Trong đó, mỗi thuộc tính của bảng là một thuộc tính của lớp AR, ở một vài trường hợp mỗi thuộc tính trong lớp AR tương ứng với một hàm trong bảng database. Các thao tác CURD thông thường đều được hỗ trợ như các phương thức của AR. Nhờ vậy, các lập trình viên có thể thao tác với DB một cách hướng đối tượng hơn. Sau đây là ví dụ thêm một dòng mới vào bảng `tbl_post`:

```php
$post = new Post;
$post->title = 'sample post';
$post->content = 'post body content';
$post->save();
```

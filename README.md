Active Record trong Yii Framework 1.1
=====================================

Mặc dù Yii DAO có thể giải quyết được hấu như mọi việc liên quan đến database, nhưng việc này làm các lập trình viên tốn đến 90% thời gian của mình dành cho các câu lệnh SQL (CURD). Thật là khó để đảm bảo code của họ khi mà phải trộn lần code với câu lênh SQL. Để giải quyết việc này, Yii đã sử dụng Active Record.

Active Record (AR) là nột kỹ thuật ORM (Object-Relational Mapping) phổ biến. Mỗi lớp AR thay mặt cho một bảng trong database. Trong đó, mỗi thuộc tính của bảng là một thuộc tính của lớp AR, ở một vài trường hợp mỗi thuộc tính trong lớp AR tương ứng với một hàm trong bảng database. Các thao tác CURD thông thường đều được hỗ trợ như các phương thức của AR. Nhờ vậy, các lập trình viên có thể thao tác với DB một cách hướng đối tượng hơn. Sau đây là ví dụ thêm một dòng mới vào bảng `tbl_post`:

```php
$post = new Post;
$post->title = 'sample post';
$post->content = 'post body content';
$post->save();
```

Phần tiếp theo mô tả cách để cài đặt AR và sử dụng nó để thực hiện các thao tác CURD. **Chú ý:** Nếu bạn sử dụng MySQL thì nên thay `AUTOINCREMENT` bằng `AUTO_INCREMENT` trong câu lệnh SQL. Ví dụ:

```SQL
CREATE TABLE tbl_post (
  id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
  title VARCHAR(128) NOT NULL,
  content TEXT NOT NULL,
  create_time INTEGER NOT NULL
);
```

**Chú ý:**

> AR không thể giải quyết được tất cả các tác vụ có liên quan đến DB. AR
> được sử dụng tốt nhất trong mô hình bảng cơ sở dữ liệu và thực hiện
> các truy vấn không quá phức tạp. Với những kịch bản phức tạp, Yii
> khuyên bạn nên dùng Yii DAO

# Thiết lập kết nối đến Cơ sở dữ liệu

AR thực hiện các thao tác liên quan đến Cơ sở dữ liệu phụ thuộc vào sự
kết nối. ở chế độ mặc định, Yii đưa ra db application component sử dụng `CDbConnection` để đáp ứng như DB
connection. Ví dụ:

```php
return array(
  'components'=>array(
    'db'=>array(
      'class'=>'system.db.CDbConnection',
      'connectionString'=>'sqlite:path/to/dbfile',
      // turn on schema caching to improve performance
      // 'schemaCachingDuration'=>3600,
    ),
  ),
);
```

**Mẹo:**

> Bởi vì Active Record dựa vào các metadata của các bảng để định rõ
> thông tin của cột, vì thế nên tốn thời gian cho việc đọc và phân tích các metadata. Nếu sơ đồ CSDL ít thay đổi, bạn nên bật schema caching bằng việc thiết lập thuộc tính `CDbConnection::schemaCachingDuration` về giá trị lớn hơn 0.

Active Record chỉ hỗ trợ giới hạn trong DBMS:
- MySQL 4.1 hoặc mới hơn.
- MariaDB.
- PostgreSQL 7.3 hoặc mới hơn.
- SQLite 2, 3.
- Microsoft SQL Server 2000 hoặc mới hơn.
- Oracle.

Nếu muốn sử dụng một application component khác db, hoặc làm việc với
nhiều database sử dụng Active Record, bạn nên thông qua `
CActiveRecord::getDbConnection()`. Lớp `CActiveRecord` sẽ là lớp cơ sở
cho tất cả các lớp Active Record khác.

**Mẹo:**

> Có 2 cách để làm việc với nhiều DB trong Active Record. Nếu schemas
> của các DB khác nhau, bạn có thể tạo các lớp Active Record cơ sở khác
> nhau tương ứng với các implementation khác nhau của `getDbConnection()`. Ngoài ra, có thể thay đổi giá trị trị của `CActiveRecord::db` cũng là một ý tưởng tốt.

# Định nghĩa lớp Active Record

Để truy cập vào một bảng cơ sở dữ liệu, chúng ta cần định nghĩa một lớp
AR được mở rộng (extending) từ lớp `CActiveRecord`. Mỗi lớp
AR đại diện cho một bảng, mỗi một trường hợp của lớp đó đại diện cho một
hàng trong bảng. Ví dụ về lớp AR tượng trưng cho bảng `tb1_post`.

```php
class Post extends CActiveRecord {
  public static function model($className=__CLASS__) {
    return parent::model($className);
  }

  public function tableName() {
    return 'tbl_post';
  }
}
```

**Mẹo:**

> Bời vì lớp AR thường được gọi từ nhiều nơi, vì thế chúng ta có thể
> import toàn bộ thư mục có chứa lớp AR thay vì including từng cái một.
> Ví dụ, nếu tất cả các lớp AR đều nằm trong thư mục `protected/models`,
> chúng ta có thể cấu hình app như sau:
> ```
> return array(
>   'import'=>array(
>     'application.models.*',
>   ),
> );
> ```



# Tài liệu tham khảo
- http://www.yiiframework.com/doc/guide/1.1/en/database.ar

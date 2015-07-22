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
> ```php
> return array(
>   'import'=>array(
>     'application.models.*',
>   ),
> );
> ```

Mặc định. tên của lớp AR giống tên của bảng trong cơ sở dữ liệu. Nếu
chúng khác nhau hãy ghi đè lên phương thức `tableName()`. Phương thức
`model()` được khai báo cho mọi lớp AR (để giải thích một cách vắn tắt).

**Thông tin thêm:**

> Để sử dụng bảng tính năng tiền tố (table prefix feature), phương thức
> `tableName()` của một lớp AR có thể viết như sau:
> ```php
> public function tableName() {
>   return '{{post}}';
> }
> ```
> Thay vì trả lại tên bảng một cách đầy đủ, ta trả lại tên bảng mà không
> có tiền tố và dấu nháy đôi hay ngặc nhọn kèm theo.

Giá trị một cột của một dòng trong bảng có thể được khai báo như là các
thuộc tính tương ứng của AR. Ví dụ với `title` của bảng `Post`:

```php
$post=new Post;
$post->title='a sample post';
```

Mặc dù, chưa khai báo thuộc tính `title` trong lớp `Post`, nhưng ta vẫn
có thể gán giá trị cho nó trong ví dụ trên. Đó là bởi `title` là một cột
của bảng `tb1_post`, và `CActiveRecord` khiến nó có thể được sử dụng như
là một thuộc tính với sự giúp đỡ của phương thức ``PHP __get()``. Một ngoại lệ sẽ được bỏ qua nếu ta cố truy cập vào một cột không tồn tại với cùng một cách.

**Thông tin thêm:**

> Trong bài viết này, tôi sẽ sử dụng trường hợp đơn giản nhất cho tất cả các tên bảng và tên cột. Sử dụng như vậy bỏi sự xử lý của DBMS khác nhau với từng trường hợp đặc biệt khác nhau.

AR phụ thuộc vào những khóa chính được xác định của các bảng. Nếu một bảng không có khóa chính thì nó sẽ yêu cầu lớp AR chỉ rõ cột/các cột nào sẽ là khóa chính thông qua phương thức `primaryKey()` như ví dụ sau:

```php
public function primaryKey() {
  return 'id';
  // For composite primary key, return an array like the following
  // return array('pk1', 'pk2');
}
```

# Tạo bản ghi

Để thêm dòng mới vào bảng cơ sở dữ liệu, ta tạo ra một trường hợp tương ứng với lớp AR, đặt các thuộc tính cho nó tương ứng với các cột trong bảng. Cuối cùng gọi phương thức `save()` để kết thúc.

```php
$post=new Post;
$post->title='sample post';
$post->content='content for the sample post';
$post->create_time=time();
$post->save();
```

Nếu khóa chính của bảng được tăng tự động, sau khi thêm, nó sẽ chứa một khóa chính đã được cập nhật. Trong ví dụ trên, thuộc tính `id` sẽ phản ánh giá trị khóa chính của `Post` mới được thêm vào, mặc dù ta không thay đổi nó.

Nếu một côt được định nghĩa với các giá trị mặc định tĩnh (VD: một chuỗi, một số) trong một lược đồ bảng, thuộc tính tương ứng trong 

# Tài liệu tham khảo
- http://www.yiiframework.com/doc/guide/1.1/en/database.ar

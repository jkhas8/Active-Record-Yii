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

## Định nghĩa lớp Active Record

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

## Tạo bản ghi

Để thêm dòng mới vào bảng cơ sở dữ liệu, ta tạo ra một trường hợp tương ứng với lớp AR, đặt các thuộc tính cho nó tương ứng với các cột trong bảng. Cuối cùng gọi phương thức `save()` để kết thúc.

```php
$post=new Post;
$post->title='sample post';
$post->content='content for the sample post';
$post->create_time=time();
$post->save();
```

Nếu khóa chính của bảng được tăng tự động, sau khi thêm, nó sẽ chứa một khóa chính đã được cập nhật. Trong ví dụ trên, thuộc tính `id` sẽ phản ánh giá trị khóa chính của `Post` mới được thêm vào, mặc dù ta không thay đổi nó.

Nếu một côt được định nghĩa với các giá trị mặc định tĩnh (VD: một chuỗi, một số) trong một lược đồ bảng, thuộc tính tương ứng trong thực thể AR sẽ tự động có giá trị như vậy sau khi được tạo ra. Thay đổi giá trị mặc định này bằng cách khai báo trực tiếp trong lớp AR:

```php
class Post extends CActiveRecord {
  public $title='please enter a title';
  ......
}

$post=new Post;
echo $post->title;  // this would display: please enter a title
```

Một thuộc tính có thể được thừa kế giá trị của loại `CDbExpression`
trước khi bản ghi được lưu lại vào cơ sở dữ liệu(trong trường hợp thêm hoặc cập nhật). Ví dụ, để lưu lại một timestamp được trả về bởi hàm `NOW()` của MySQL, ta có thể làm như sau:

```php
$post=new Post;
$post->create_time=new CDbExpression('NOW()');
// $post->create_time='NOW()'; will not work because
// 'NOW()' will be treated as a string
$post->save();
```

**Mẹo:**

> Vì AR cho phép chúng ta có thể thực hiện các thao tác với cơ sở dữ liệu
> mà không cần phải viết các câu lệnh SQL vướng víu, ta thường muốn biết
> các câu lệnh SQL được thực hiện bởi dưới AR như thế nào. Điều này hoàn
> toàn có thể thực hiện được bằng cách bật logging feature của Yii lên.
> Ví dụ, ta có thể bật `CWebLogRoute` trong phần cấu hình của
> application, và ta sẽ thấy được câu lệnh SQL đã thực hiện được hiển
> thị ở cuối cùng của trang web. Ta có thể đặt
> `CDbConnection::enableParamLogging` có giá trị `true` trong cấu hình
> của application để các giá trị tham số trong các câu lệnh SQL được log
> lại.

## Đọc bản ghi

Để đọc dữ liệu trong bảng cơ sở dữ liệu, ta cần gọi phương thức `find`
như sau:

```php
// find the first row satisfying the specified condition
$post=Post::model()->find($condition,$params);
// find the row with the specified primary key
$post=Post::model()->findByPk($postID,$condition,$params);
// find the row with the specified attribute values
$post=Post::model()->findByAttributes($attributes,$condition,$params);
// find the first row using the specified SQL statement
$post=Post::model()->findBySql($sql,$params);
```

Ở trên, ta đã gọi phương thức `find` với `Post::model()`. Điều cần lưu
ý ở đây là phương thức tĩnh `model()` là bắt buộc với mọi lớp AR. Phương
thức trả về một thực thể AR được sử dụng để truy cập các phương thức
được phân cấp theo lớp (phương pháp tương tự phương pháp lớp tĩnh) trong
một đối tượng được xét.

Nếu phương thức `find` tìm ra một hàng thỏa mãn điều kiện truy vấn, nó
sẽ trả về một thực thể `Post` với các thuộc tính tương ứng với các cột
trong dòng đó. Ta có thể các giá trị đó như làm với các thuộc tính thông
thường, ví dụ `echo $post->title;`

Phương thức `find` sẽ trả về `null` nếu không tìm thấy gì phù hợp với
điều kiện truy vấn trong cơ sở dữ liệu.

Khi gọi `find`, ta sử dụng `$condition` và `$params` để định rõ các điều
kiện truy vấn. `$condition` có thể là một chuỗi tượng trưng cho mệnh đề
`WHERE` trong câu lệnh SQL, và `$params` là một mảng các tham số với các
giá trị đã đã được xác định trước trong `$condition`. Ví dụ:

```php
// find the row with postID=10
$post=Post::model()->find('postID=:postID', array(':postID'=>10));
```

**Chú ý:**

> ở trên ta cần phải thoát khỏi tham chiều đến cột `postID` cho một hệ
> quản trị cơ sở dữ liệu nhất định. Ví dụ, nếu sử dụng PostgreSQL, ta
> phải viết điều kiện như sau `"postID"=:postID`, bởi PostgreSQL mặc
> định xử lý tên cột như case-insensitive.

Ta cũng có thể sử dụng `$condition` để định rõ nhiều điều kiện truy vấn
phức tạp hơn. Thay vì là một chuỗi, ta có thể cho `$condition` thành `CDbCriteria`. Việc này cho phép ta có thể định rõ nhiều điều kiện hơn là chỉ sử dụng mệnh đề `WHERE`. Ví dụ:

```php
$criteria=new CDbCriteria;
$criteria->select='title';  // only select the 'title' column
$criteria->condition='postID=:postID';
$criteria->params=array(':postID'=>10);
$post=Post::model()->find($criteria); // $params is not needed
```

Chú ý, khi sử dụng `CDbCriteria` như điều kiện truy vấn, `$params` sẽ
khồng cần, vì nó đã được định nghĩa trong `CDbCriteria`.

Một cách khác cho `CDbCriteria` là truyền qua mảng cho phương thức
`find`. Khóa và giá trị của mảng tương ứng với tên và giá trị thuộc tính
của criteria. Để dễ hiểu hơn, ta sẽ viết lại ví dụ trên như sau:

```php
$post=Post::model()->find(array(
  'select'=>'title',
  'condition'=>'postID=:postID',
  'params'=>array(':postID'=>10),
));
```

**Thông tin thêm:**

> Khi điều kiện truy vấn có sự kết hợp một số cột với các giá trị cụ
> thể, ta có thể sử dụng ` findByAttributes()`. Ta đưa tham số `$attributes` thành một mảng của các giá trị đã được đánh index bởi tên cột. Ở một vài framework, việc này được làm bằng cách gọi các phương thức như `findByNameAndTitle`. Mặc dù cách tiếp cận vấn đề như vậy trông có vẻ rất tốt, nhưng nó thưởng xuyên là nguyên nhân của sự nhầm lẫn, conflict và các vấn đề case-sensitivity của tên cột.

Khi nhiều dòng của dữ liệu nối với nhau qua điều kiện truy vấn, ta có
thể mang tất cả chúng gộp lại tìm với phương thức `findAll`, mỗi phần
sẽ được tìm tương ứng với phương thức `find` đã mô tả bên trên.

```php
// find all rows satisfying the specified condition
$posts=Post::model()->findAll($condition,$params);
// find all rows with the specified primary keys
$posts=Post::model()->findAllByPk($postIDs,$condition,$params);
// find all rows with the specified attribute values
$posts=Post::model()->findAllByAttributes($attributes,$condition,$params);
// find all rows using the specified SQL statement
$posts=Post::model()->findAllBySql($sql,$params);
```

Nếu không có gì phù hợp với điều kiện truy vấn, `findAll` sẽ trả về một mảng rỗng (Điều này khác với `find`, bởi `find` sẽ tả về `null` nếu không tìm được).

Bên cạnh `find` và `findAll` còn có các phương thức khác cũng được cung cấp một cách thuận tiện. Ví dụ:

```php
// get the number of rows satisfying the specified condition
$n=Post::model()->count($condition,$params);
// get the number of rows using the specified SQL statement
$n=Post::model()->countBySql($sql,$params);
// check if there is at least a row satisfying the specified condition
$exists=Post::model()->exists($condition,$params);
```

## Cập nhật bản ghi

Sau khi thực thể AR được tạo ra với các cột giá trị, ta có thể thay đổi chúng và lưu lại vào bảng dữ liệu.

```php
$post=Post::model()->findByPk(10);
$post->title='new post title';
$post->save(); // save the change to database
```

Ta cũng sử dụng phương thức `save()` như việc tạo để lưu lại các giá trị cập nhật. Nếu một thực thể AR được tạo ra bằng cách sử dụng `new`, thì việc gọi đén `save()` sẽ thêm dòng mới vào bẳng cơ sở dữ liệu. Nếu thực thể AR là kết quả của `find` hoặc `findAll`, việc gọi `save()` lúc này là cập nhật dòng có sẵn trong bảng. Trong thực tế, ta có thể sử dụng `CActiveRecord::isNewRecord` để xác định xem thực thể AR đó có phải mới hay không.

```php
// update the rows matching the specified condition
Post::model()->updateAll($attributes,$condition,$params);
// update the rows matching the specified condition and primary key(s)
Post::model()->updateByPk($pk,$attributes,$condition,$params);
// update counter columns in the rows satisfying the specified conditions
Post::model()->updateCounters($counters,$condition,$params);
```

Ở đoạn code trên, `$attributes` là một mảng của các cột giá trị được đánh index bởi tên cột; `$counters` là một mảng các giá trị gia tăng được đánh index bởi tên cột; `$condition` và `$params` tương tự như đã nói ở phần trước.

## Xóa bản ghi

Ta cũng có thể xóa một dòng dữ liệu nếu thực thể AR  đã tồn tại (có dòng tương ứng trong bảng).

```php
$post=Post::model()->findByPk(10); // assuming there is a post whose ID is 10
$post->delete(); // delete the row from the database table
```

Chú ý, sau khi xóa, thực thể AR vẫn không đổi, nhưng các hàng tương ứng trong bảng cơ sở dữ liệu đã bị xóa.



## Tài liệu tham khảo
- http://www.yiiframework.com/doc/guide/1.1/en/database.ar

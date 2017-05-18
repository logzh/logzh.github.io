# SQL生成器 Query Builder

## Select
```
$query = DB::select();//返回一个Database_Query_Builder_Select对象，因此可以使用链式语句
DB::select('username', 'password')//指定列名
$query = DB::select()->from('users'); //返回一个Database_Query_Builder_Select对象
SELECT * FROM `user`

$query = DB::select()->from(array('user', 'alias_name')); 
SELECT * FROM `user` AS `new_table`

 where(), and_where() and or_where() methods. These methods take three parameters: a column, an operator(IN, BETWEEN, >, =<, !=, etc), and a value(Use an array for operators that require more than one value).

$query = DB::select()->from('users')->where('username', '=', 'john');//The where() method is a wrapper that just calls and_where().

$query = DB::select()->from('users')->where('username', '=', 'john')->or_where('username', '=', 'jane');

$query = DB::select()->from('users')->where('logins', '<=', 1);
 
$query = DB::select()->from('users')->where('logins', '>', 50);
 
$query = DB::select()->from('users')->where('username', 'IN', array('john','mark','matt'));
 
$query = DB::select()->from('users')->where('joindate', 'BETWEEN', array($then, $now));
```

## Select - AS (column aliases)
```
$query = DB::select(array('openId', 'wx'), 'nickname')->from('user');
SELECT `openId` AS `wx`, `nickname` FROM `user`

$query = DB::select(DB::expr('openId as wx'), 'nickname')->from('user')->execute()
SELECT openId as wx, `nickname` FROM `user`
```
## Select - DISTINCT
```
$query = DB::select('forumId')->distinct(TRUE)->from('post');
SELECT DISTINCT `forumId` FROM `post`
Select - LIMIT & OFFSET
$query = DB::select()->from(`user`)->limit(5)->offset(10);
SELECT * FROM `user` LIMIT 5 OFFSET 10
```
## Select - ORDER BY
It takes the column name and an optional direction string as the parameters.
```
$query = DB::select('*')->from('user')->order_by('id', 'DESC');
SELECT * FROM `user` ORDER BY `id` DESC
```
## Joins

join

```
$query = DB::select('authors.name', 'posts.content')->from('authors')->join('posts')->on('authors.id', '=', 'posts.author_id')->where('authors.name', '=', 'smith');
SELECT `authors`.`name`, `posts`.`content` FROM `authors` JOIN `posts` ON (`authors`.`id` = `posts`.`author_id`) WHERE `authors`.`name` = 'smith';
```
left join
```
$query = DB::select('userTrial.*', 'trial.*', 'toy.title')
            ->from('userTrial')->join('trial', 'left')
            ->on('userTrial.trialId', '=', 'trial.id')->join('toy', 'left')
            ->on('trial.toyId', '=', 'toy.id')
            ->where('userTrial.userId', '=', $userId)->where('userTrial.state', '!=', 'info')
            ->where('userTrial.rowStatus', '=', 0)->where('trial.rowStatus', '=', 0)->order_by('trial.id', 'DESC')
SELECT `userTrial`.*, `trial`.*, `toy`.`title` FROM `userTrial` LEFT JOIN `trial` ON (`userTrial`.`trialId` = `trial`.`id`) LEFT JOIN `toy` ON (`trial`.`toyId` = `toy`.`id`) WHERE `userTrial`.`userId` IS NULL AND `userTrial`.`state` != 'info' AND `userTrial`.`rowS//tatus` = 0 AND `trial`.`rowStatus` = 0 ORDER BY `trial`.`id` DESC;
```
## Database Functions
```
DB::select(DB::expr('COUNT(*) AS total'))->from($this->_table)->where('trialId', '=', $trialId)->where('state', '!=', 'info')->where('rowStatus', '=', 0);
SELECT COUNT(*) AS total FROM `userTrial` WHERE `trialId` = 266 AND `rowStatus` = 0;
```
## Aggregate Functions
```
$query = DB::select('username', array(DB::expr('COUNT(`id`)'), 'total_posts')
    ->from('posts')->group_by('username')->having('total_posts', '>=', 10);
SELECT `username`, COUNT(`id`) AS `total_posts` FROM `posts` GROUP BY `username` HAVING `total_posts` >= 10;
```
## Subqueries
```
$sub = DB::select('username', array(DB::expr('COUNT(`id`)'), 'total_posts')
    ->from('posts')->group_by('username')->having('total_posts', '>=', 10);
 
$query = DB::select('profiles.*', 'posts.total_posts')->from('profiles')
    ->join(array($sub, 'posts'), 'INNER')->on('profiles.username', '=', 'posts.username');
SELECT `profiles`.*, `posts`.`total_posts` FROM `profiles` INNER JOIN
( SELECT `username`, COUNT(`id`) AS `total_posts` FROM `posts` GROUP BY `username` HAVING `total_posts` >= 10 ) AS posts
ON `profiles`.`username` = `posts`.`username`

$sub = DB::select('username', array(DB::expr('COUNT(`id`)'), 'total_posts')
    ->from('posts')->group_by('username')->having('total_posts', '>=', 10);
$query = DB::insert('post_totals', array('username', 'posts'))->select($sub);

INSERT INTO `post_totals` (`username`, `posts`) 
SELECT `username`, COUNT(`id`) AS `total_posts` FROM `posts` GROUP BY `username` HAVING `total_posts` >= 10 
Boolean Operators and Nested Clauses
$query = DB::select()->from('users')
    ->where_open()
        ->or_where('id', 'IN', $expired)
        ->and_where_open()
            ->where('last_login', '<=', $last_month)
            ->or_where('last_login', 'IS', NULL)
        ->and_where_close()
    ->where_close()
    ->and_where('removed','IS', NULL);
SELECT * FROM `users` WHERE ( `id` IN (1, 2, 3, 5) OR ( `last_login` <= 1276020805 OR `last_login` IS NULL ) ) AND `removed` IS NULL ;
```
## Database Expressions
```
$query = DB::update('users')->set(array('login_count' => DB::expr('login_count + 1')))->where('id', '=', $id);
$query = DB::select(array(DB::expr('degrees(acos(sin(radians('.$lat.')) * sin(radians(`latitude`)) + cos(radians('.$lat.')) * cos(radians(`latitude`)) * cos(radians(abs('.$lng.' - `longitude`))))) * 69.172'), 'distance'))->from('locations');
```
## Insert、Update、Delete
```
$result = DB::insert('user', array('openid', 'nickname'))
    ->values(array('openid' => '12341115', 'nickname' => 'test_insert'))
    ->execute();
var_dump($result);// id, row
// array (size=2)
//0 => int 69//id
//1 => int 1//row


$result = DB::insert('user', array('openid', 'nickname'))
     ->values(array('openid' => '12341115', 'nickname' => 'test_insert'))
     ->values(array('openid' => '12341116', 'nickname' => 'test_insert'))
    ->values(array('openid' => '12341118', 'nickname' => 'test_insert'))
    ->execute();
var_dump($result);// id, row
// array (size=2)
//0 => int 69//id
//1 => int 3//row

$result = DB::update('user')
   ->set(array('nickname' => 'update_name5', 'rowStatus' => -1))
   ->where('nickname', '=', 'update_name')
    ->execute();
 var_dump($result);//row 更新行数

$result = DB::delete('user')->where('rowStatus', '=', -1)->execute();

var_dump($result);//row 删除行数
```        
## Executing
生成sql语句之后就是执行sql，返回Database_MySQL_Result对象
```
$result = $query->execute();//使用默认数据库

$result = $query->execute('config_name');//指定配置数据库文件

$result = $query->execute('db_name');//指定数据库
```
# 遍历结果

## 1、常规遍历
```
$rows = array();
foreach ($result as $row)
{
	$rows[] = $row;
}
```
## 2、使用as_array()
```
$rows =$result->as_array();//$rows值和1中相同
$rows = $result->as_array('id', 'name');//array('id1' => 'name1', 'id2' => 'name2')
```
# 手动参数化语句 Parameterized Statements
 
使用参数化的语句允许你手动编写SQL语句，同时可以防止SQL注入

```
$query = DB::query(Database::SELECT, 'select * from user where id=:id');
$query->param(':id', 44);//参数名称使用唯一字符串即可，kohana使用strtr替换指定字符串，建议不要使用$，以免引起混淆
$result = $query->execute();

$query = DB::query(Database::SELECT, 'SELECT * FROM user WHERE sex = :sex AND rowStatus = :rowStatus');
$query->parameters(array(
    ':sex' => '1',
    ':rowStatus' => 0,
));
$result = $query->execute();

$query = DB::query(Database::SELECT, 'select * from user where id=:id')->bind(':id', $id);
$result = $query->execute()->as_array();

$query = DB::query(Database::SELECT, 'SELECT * FROM user WHERE sex = :sex AND rowStatus = :rowStatus')->bind(':sex', 2)->bind(':rowStatus', 0);
$result = $query->execute();

$query = DB::query(Database::UPDATE, "update user set sex=:sex, expr = :expr where id = :id");
$query->parameters(array(':sex' => '2', ':expr' => 345, ':id' => 44));

DB::query(Database::DELETE, "DELETE FROM itest_process_stories WHERE itest_process_stories.process_id = :process_id")->bind(':process_id', $process_id);
$result = $query->execute()

DB::query(Database::INSERT, "...")->bind(':process_id', $process_id);
$result = $query->execute()
 ```

# 事务
```
$db = DB::instance();
try{
    $db->begin();
    ...
    $db->commit();
}catch (Exception $e){
   $db->rollback();
}

```

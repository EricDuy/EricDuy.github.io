---
layout:     post
title:      Python操作MySQL
subtitle:   
date:       2018-08-22
author:     Eric
header-img: img/post-bg-mysql.jpg
catalog: true
tags:
    - 爬虫
---

## 环境搭建
> 先简单介绍一下MySQL5.7免安装教程

#### 1. 软件下载  
https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.19-winx64.zip
#### 2. 添加环境变量
在电脑环境变量之系统变量里PATH中添加解压后的MySQL5.7的路径(如：D:\MySQL\MySQL5.7\bin)
#### 3. 新建my.ini文件
编辑my.ini的内容为如下：
```mysql
[mysqld]
basedir=D:\MySQL\MySQL5.7\
datadir=D:\MySQL\MySQL5.7\data\
port=3306
skip-grant-tables

#basedir表示mysql安装路径
#datadir表示mysql数据文件存储路径
#port表示mysql端口
#skip-grant-tables表示忽略密码
```
#### 4. 安装MySQL服务
启动**管理员模**式下的cmd，然后切换到MySQL5.7的bin路径下，执行`mysqld --install`

![](http://ww1.sinaimg.cn/large/005K0wPWly1fui9xlxv4tj30ab0103y9.jpg)

#### 5. 启动MySQL服务
![](http://ww1.sinaimg.cn/large/005K0wPWly1fui9zd29l4j30bl03o3yb.jpg)  
> 这时遇到安装的第一个坑！

原因：MySQL数据库在升级到5.7版本后，和之前的版本有些不一样，没有data文件夹了，而在data文件夹中保存的是MySQL数据库文件

**解决**：输入`mysqld --initialize-insecure --user=mysql`,然后重新启动MySQL服务。这时就可以成功启动服务了。

#### 6. 进入MySQL管理
输入`mysql -u root -p`，密码可以为空，直接按回车，成功进入MySQL管理界面

![](http://ww1.sinaimg.cn/large/005K0wPWly1fuiagt3hgzj30jb078mx3.jpg)  

#### 7.重新设置密码
输入`update mysql.user set authentication_string=password('123456') where user='root' and Host = 'localhost';`  
密码我举例是'123456',实际使用可以自行设置  
然后刷新一下权限`flush privileges;`，将之前添加的**my.ini**文件里的最后一句skip-grant-tables删除  
再重启一下mysql  
![](http://ww1.sinaimg.cn/large/005K0wPWly1fuib9liu53j30bv0420sj.jpg)

> 这时遇到安装的第二个坑！

登录mysql后，进行操作提示错误，如下   

![](http://ww1.sinaimg.cn/large/005K0wPWly1fuibc5l448j30og0160si.jpg)

##### 解决方法：
step1:输入`SET PASSWORD = PASSWORD('123456');`  
step2:输入`ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;`  
step3:输入`FLUSH PRIVILEGES;`

之后就可以安心使用MySQL了！

---

## 语法基础


#### 创建名为firstdatabase的数据库
``CREATE DATABASE firstdatabase ;``

#### 使用名为firstdatabase的数据库  
``USE  firstdatabase;  ``    

#### 查看已有数据库  
`SHOW DATABASES; ` 
#### 数据常见类型 
int,char,varchar,datetime  
注：char(n)存储固定n个字符,varchar(n)可以存储<=n个任意字符

> **代码例子**
```sql
CREATE TABLE students (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    sex CHAR(1) NULL,
    in_time  DATETIME NULL
    ) DEFAULT CHARSET 'UTF8';
```
为了方便索引查询数据表里的内容，会设置一个主键（PRIMARY KEY)，相当于数据表数据的一个编号  
`DEFAULT CHARSET 'UTF8'`保证数据表里出现的中文不会乱码

### 插入语句  

```SQL
INSERT INTO `students` VALUE (1,`Eric`,`男`,now())
INSERT INTO `students2` (`name`,`sex`) VALUES
    ('Tony','男')，
    ('Mary','女')
；
```
### 查询语句
**常用查询语句**  
``SELECT  FROM  
[WHERE]  [GROUP BY]  [ORDER BY]      [LIMIT]  ``
使用时一定要按照这个顺序
```SQL
SELECT id,nam  FROM firstdatabase WHERE sex ='男' ORDER BY 'id' DESC;
```
```WHERE `sex`= '男'```筛选数据的条件为男  
`ORDER BY` 以id为条件排序  
`DESC` 倒序

### 修改
基本操作：``UPDATE   TABLE   SET  WHERE ;``  

```SQL
UPDATA  firstdatabase SET name='Tony' WHERE sex='男' 
```
`SET`  后面跟的是要修改成的数据，`WHERE`后面跟的是筛选的条件.注意一定要添加`WHERE`，否则将会把数据表全部更新修改了
### 删除
基本操作:``DELETE FROM TABLE WHERE ;``
```SQL
DELETE FROM firstdatabase WHERE sex = '男'
```



## PyMySQL的使用

- 游标对象(cursor)：用于执行查询和获取结果
- cursor对象常用语法：
    

参数名 | 功能
---|---
execute()| 执行数据库的查询，更新等命令。将结果从数据库获取到客户端
fetchone() |获取结果集的下一行
fetchmany(n)|获取结果集的下n行
fetchall()|获取结果集中剩下的所有行
rowcount|最近一次执行execute返回数据的行数或者execute所影响的行数
close()|关闭游标对象

![](http://ww1.sinaimg.cn/large/005K0wPWly1fugfyzplgcj30hl09ijt6.jpg)

> 代码示例

```python
import pymsql
    try:
        #打开数据库连接
        conn = pymsql.connect(     
            host = 'localhost',
            user = 'root',
            passwd = 'duyu2102268',
            db = 'nwws',
            port = 3306,
            charset = 'utf8'
        )
        
        #使用 cursor() 方法创建一个游标对象 cursor
        cursor = conn.cursor()
        
        #所要执行的sql语句
        sql = 'SELECT * FROM USER'
        
        #使用execute()方法执行sql语句
        cursor.execute(sql)
        
        #使用fetchall()获取数据表剩下的数据
        rs = cursor.fetchall()
        for row in rs:
            print('userid={0},username={1}'.format(raw))     #row为一个元组
    except Exception as e:
        print('Error: {}' .format(e))
    finally:
        #关闭数据库连接
        cursor.close()
```
其中通过`fetchall()`得到的是一个元组的元组，所以要想得到相应信息需要遍历


#### 事务操作
事务：execute操作的集合  
提交：让事务中的所有操作同时生效-->正常结束事务`conn.commit() `   
回滚：整个操作的集合回到没有执行的状态-->操作出现异常，结束事务`conn.rollback()`

> 关于python操作MySQL的项目实例可以参见[https://github.com/EricDuy/Python-MySQL](https://note.youdao.com/)

--***END***--

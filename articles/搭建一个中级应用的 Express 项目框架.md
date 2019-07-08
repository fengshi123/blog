## 前言

`Node.js` 对前端来说无疑具有里程碑意义，在其越来越流行的今天，掌握 `Node.js` 已经不再是加分项，而是前端攻城师们必须要掌握的技能。本文将完搭建`Express + MySQL`的中级服务端应用，通过` express-generator `搭建  `express `项目，以及 `express `项目如何与数据库交互、`express`项目如何记录日志、以及利用 `domain `捕获 `uncaughtException ` 异常，解决`Node` 服务退出登问题；已经将项目框架完整的代码上传到 `github`，`github`地址为：[github.com/fengshi123/…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Fexpress_project)，欢迎 **Star**....

 一、Express-generator创建Express应用骨架

1、安装 `express `生成器 `express-generator`

```
$ npm install -g express-generator
```

2、安装 `express`

```
$ npm install -g express
```

3、初始化项目，项目名称为 `backend`

```
$ express backend
```

4、安装依赖

```
$ cd backend
$ npm install
```

5、项目的骨架结构

![img](https://user-gold-cdn.xitu.io/2019/7/1/16ba93806e57c0a2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

- /bin：用于应用启动
- /public： 静态资源目录
- /routes：可以认为是controller（控制器）目录
- /views：jade模板目录，可以认为是view(视图)目录
- app.js：程序main文件

6、启动项目

```
$ npm start复制代码
```

浏览器访问截图如下所示：

![img](https://user-gold-cdn.xitu.io/2019/7/1/16ba937e9e301b06?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

## 二、Express 结合 MySQL 数据库

1、首先安装配置好 `MySQL `数据库，具体操作可以参照以下链接：[supportopensource.iteye.com/blog/141552…](https://link.juejin.im?target=https%3A%2F%2Fsupportopensource.iteye.com%2Fblog%2F1415527)；

以及安装一个数据库管理工具 `navicat for mysql` 。

2、编写 `mysql `的 `sql `脚本，我们只要执行` setup.sh `脚本进行自动创建表等 `sql `操作：

![img](https://user-gold-cdn.xitu.io/2019/7/1/16ba9404048b7c65?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

（1）`setup.sh` 的功能为连接 `mysql `数据库，并且切换数据库，执行对应的 `sql `脚本文件，脚本代码如下：

```
#!/bin/bash
mysql -uroot -p123456 --default-character-set=utf8 <<EOF
drop database if exists research;
create database research character set utf8;
use research;
source init.sql;
EOF
cmd /k
```

（2）`init.sql `的功能为 添加要执行的 `sql `脚本文件，脚本代码如下：

```
source t_user.sql;
```

（3）`t_user.sql `功能为创建表` t_user`，并插入对应数据：

```
create table if not exists t_user( 
  uid varchar(16) primary key, 
  name varchar(16),  
  sex varchar(16)
) default charset = utf8;
insert into t_user values('1','teacher','男');
insert into t_user values('2','teacher1','女');
commit;
```

3、结合 `express `和 `mysql`，创建以下目录文件：

`conf/db.js`  连接 `mysql`数据库

`dao `与数据库交互

（1）`conf/db.js`，编写 `mysql `数据库连接配置

```
// MySQL数据库联接配置
module.exports = { 
  mysql: {   
    host: '127.0.0.1',    
    user: 'root',   
    password: '123456',    
    database: 'research',   
    port: 3306  
  }
};
```

（2）`dao/userSqlMapping.js`，编写` CURD sql `语句：

```
// CRUD SQL语句
const user = {  
   insert: 'insert into t_user(uid, name, sex) VALUES(?,?,?)',  
   update: 'update t_user set name=?, sex=? where uid=?',  
   delete: 'delete from t_user where uid=?',  
   queryById: 'select * from t_user where uid=?',  
   queryAll: 'select * from t_user'};module.exports = user;
}
```

（3）`dao/userDao.js`，数据库 `CURD `的具体实现，示例如下：

```
add: function (req, res, next) {    
pool.getConnection(function (err, connection) {     
 if (err) {        
     logger.error(err);        
     return;      
  }     
  // 获取前台页面传过来的参数      
  var param = req.body;     
  // 建立连接，向表中插入值      
  connection.query(sql.insert, [param.uid, param.name, param.age], function (err, result) {  
   if (err) {         
      logger.error(err);        
   } else {          
      result = {           
        code: 0,           
         msg: '增加成功'         
      };        
   }       
  // 以json形式，把操作结果返回给前台页面        
  jsonWrite(res, result);        
  // 释放连接        
  connection.release();     
  });    
});  
},
```

（4）`routes/users.js` 路由匹配对应的数据库操作：

```
// 增加用户
router.post('/addUser', function (req, res, next) {  
   userDao.add(req, res, next);
});
```

## 三、日志记录

 1、`morgan：express `自带的 `http `请求日志中间件

`morgan`是`express`默认的日志中间件，也可以脱离`express`，作为`node.js`的日志组件单独使用，`morgan`的具体 `api `可参考 github：[github.com/expressjs/m…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fexpressjs%2Fmorgan)

（1）项目中使用其来记录请求的日志，代码如下：

```
const logger = require('morgan');
// 输出日志到目录
var accessLogStream = fs.createWriteStream(path.join(__dirname, '/log/request.log'), 
{ flags: 'a', encoding: 'utf8' }); /
/ 记得要先把目录建好，不然会报错
app.use(logger('combined', { stream: accessLogStream }));
```

（2）日志记录截图如下：

![img](https://user-gold-cdn.xitu.io/2019/7/1/16ba941e61593d45?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

2、winston

由于 `morgan `只能记录 `http`请求的日志，所以我们还需要 `winston `来记录其它想记录的日志，例如：访问数据库出错等；`Winston` 是 `Node.js `上最流行的日志库之一。它被设计为一个简单通用的日志库，支持多种传输（一种传输实际上就是一种存储设备，例如日志存储在哪里）。`winston`中的每一个 `logger `实例在不同的日志级别可以存在多个传输配置；当然 它也可以记录请求记录。`winston `的具体`api `可参考 github：[github.com/winstonjs/w…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fwinstonjs%2Fwinston)

（1）项目中使用 `winston `的代码如下：

```
/*  多container及多transport组合 */
const { createLogger, format, transports } = require('winston');
const { combine, timestamp, printf } = format;
const path = require('path');
const myFormat = printf(({ level, message, label, timestamp }) => {  
    return `${timestamp} ${level}: ${message}`;
});
const logger = createLogger({  
     level: 'error',  
     format: combine(    timestamp(),    myFormat  ), 
     transports: [    
         new (transports.Console)(),   
         new (transports.File)({      
            filename: path.join(__dirname, '../log/error.log')   
            }) 
          ]
       });
module.exports = logger;


const logger = require('../common/logger'); 
logger.error(err);
```

（2）日志记录截图如下：

![img](https://user-gold-cdn.xitu.io/2019/7/1/16ba944b55df9da9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 四、处理 uncaughtException 错误导致的node 服务退出

`Node 0.8` 之后的版本新增了`domain `模块，它可以用来捕获回调函数中抛出的异常。

`domain `主要的 `API `有 `domain.run` 和 `error `事件。简单的说，通过` domain.run` 执行的函数中引发的异常都可以通过 `domain `的 `error `事件捕获。我们在 `express `中项目中使用 `domain `的代码如下：

```
// 处理没有捕获的异常，导致 node 退出
app.use(function (req, res, next) {  
  var reqDomain = domain.create();  
  reqDomain.on('error', function (err) {   
     res.status(err.status || 500);    
     res.render('error'); 
  }); 
  reqDomain.run(next);
});
```

## 五、添加 ESlint 检查代码规范

为了让项目的代码风格保持良好且一致，故在项目中添加 `eslint `来检查 `js `代码规范；

（1）安装`eslint`

```
npm install -g eslint
```

（2）`eslint` 初始化

```
eslint --init
```

## 六、总结

本文我们主要介绍通过` express-generator` 搭建  `express `项目，以及 `express `项目如何与数据库交互、`express`项目如何记录项目日志、以及利用 `domain `捕获 `uncaughtException ` 异常，解决`node `服务退出登问题；已经将项目完整的代码实例上传到 github，github地址为：[github.com/fengshi123/…](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Ffengshi123%2Fexpress_project)，欢迎 star...
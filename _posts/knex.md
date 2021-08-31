# Knex

**knex** 是用来对各种数据库(sqlite3,mysql,posgresql)进增删改查。使用起来非常简单。特别是业务比较简单，侧重于数据查询。并且通过knex就可以创建相关的表结构。

对于一个数据库，我们可能需要根据业务变化不断修改表结构，比如添加一个新的字段， 同时 web app 会运行在多种环境当中(测试、生产等)，一个环境改变了数据库结构，另一个不会随着改变，它不像版本管理那样。而knex数据库迁移(Database migrations) 解决了这个问题。knex的解决方式就是创建一个迁移文件。每次对数据库的更改，都会有个新文件。

迁移文件存储在 /migrations 文件夹中，配置完全局的knex命令之后我们可以通过命令
```
knex:migrate make migration_name
```
>  migateion_name 应该具有描述特性，而不是简单是一个表的名字。你需要通过文件名，直接这个迁移文件干了什么。

打开你刚刚的创建的迁移文件你会发现有
```javascript
exports.up = function(knex) {
    // 对数据库的改变
}
exports.down = function(knex) {
    // 撤销up的更改
}
```
比如up里面创建了一个user表，那么在down里面就要删除。在up里面删除了一个表的字段，那么在down里面就要增加这个字段。


## 利用knex完成对数据库表的创建
假如现在需要你创建一个数据库表包含产品信息的产品表product，它包含 产品名称、描述、价格、分类。那我们应该按照下面的步骤来完成对数据库表的创建

1. 利用knex cli命令行工具创建对应的迁移文件
2. exports.up 中编写创建产品表的逻辑
3. exports.down 里面删除这个表。
4、运行这次迁移

### 步骤1: 利用knex cli命令行工具创建对应的迁移文件
```
knex migrate:make create_products
```
查看一下migrations 文件夹你会发现创建一个带时间戳的文件。
### 步骤2： 在exports.up 中创建表
```javascript
exports.up = function(knex) {
    return knex.schema.createTable('products', function(table){
        table.increments('id').unsigned().primary();
        table.dateTime('createdAt').notNull();
        table.dateTime('updatedAt').nullable();
        table.dateTime('deletedAt').nullable(0);
        
        table.string('name').notNull();
        table.text('description').nullable();
        table.decimal('price', 6, 2).notNull();
        table.enum('category', ['apparel', 'electronics', 'furniture'])
    })
}
```
### 步骤3： 在exports.down 中删除产品表
```javascript
exports.down = function(knex) {
    return knex.schema.dropTable('prodcuts');
}
``` 
记住，done是撤销对up操作。
### 步骤4： 运行这次迁移
现在我们就可以通过运行迁移文件来创建我们所需要的表了。
```
knex migrate:latest
```
指定的环境是当前Node与运行的环境，你可以通过 -env来指定是生产环境。
> 运行完之后，你就后悔了，不想运行这次迁移，那么久赶快回滚吧 `knex migrate:latest`

## 对数据库初始化
与迁移一样，knex还允许你创建一个脚本用来初始化插入数据库的数据。如果数据库表之间有依赖，那么这些脚本要按顺序创建。

```
knex seed:make 01_users
knex seed:make 02_tasks
```
运行seed脚本
```
knex seed:run
```

## knex模块应用
knex模块创建时候，需要你传入所需数据库的配置。

**运行knex init 创建配置文件，配置不同环境下的数据库**
```javascript
knex init
// 根目录下会创建 knexfile.js 
```
**在db文件夹下创建knex.js 用来初始化knex**
```javascript
const environment = process.env.NODE_ENV || 'development';
const config =  require('../knexfile.js')[envionment];
module.exports = require('knex')(config);
```
**使用 knex**
```javascript
const knex = require('./db/knex');
knex('users')// 进行增删改查操作。
```
[参考1](http://perkframework.com/v1/guides/database-migrations-knex.html)

[参考2](https://gist.github.com/NigelEarle/70db130cc040cc2868555b29a0278261)

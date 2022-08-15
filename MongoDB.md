#### 1. 新增

```
db.集合名.insert(文档对象)
db.集合名.insertOne(文档对象)
db.集合名.insertMany(文档对象数组)
```

#### 2. 查询

```
db.集合名.find(查询条件[, 过滤])
db.集合名.findOne(查询条件[, 过滤])
> 过滤对象中需要的属性为1，不需要的属性为0，其中_id默认显示
查一条年龄为18的数据，且只返回stu_id、name
db.students.find({age: 18}, {stu_id: 1, name: 1, _id: 0})
> 除了_id，其他属性间的1和0不可混用
错误例子：db.students.find({age: 18}, {stu_id: 1, name: 0, _id: 0})
```

```
查询条件操作符
1. <($lt), <=(lte), >(gt), >=(gte), !==($ne)
年龄大于等于18的学生：
db.students.find({age: {$gte: 18}})
2. 或($in, $or)
年龄为18或20的学生：
db.students.find({age: {$in: [18, 20]}})
db.students.find({$or: [{age: 18}, {age: 20}]})
3. 非($nin)
年龄不等于18的学生：
db.students.find({age: {$nin: [18]}})
4. 正则匹配
名字以z开头的学生：
db.students.find({name: /^z/})
5. where函数
db.students.find({$where: function() {
	return this.name === 'zhangsan' && this.age === 18
}})
```

#### 3. 修改

```
db.集合名.update(查询条件, 要更新的内容[, 配置对象])
db.集合名.updateOne(查询条件, 要更新的内容[, 配置对象])
db.集合名.updateMany(查询条件, 要更新的内容[, 配置对象])
如下写法会替换掉文档对象（除_id）的所有内容
db.students.update({name: 'zhangsan'}, {age: 19})
正确用法为$set
db.students.update({name: 'zhangsan'}, {$set: {age: 19}})
更新多条数据
db.students.update({name: 'zhangsan'}, {$set: {age: 19}}, {multi: true})
```

#### 4. 删除

```
db.集合名.remove(查询条件)
删除年龄大于等于18的学生：
db.students.remove({age: {$gte: 18}})
```

#### 5. mongoose

```javascript
const mongoose = require("mongoose")
// 连接数据库
mongoose.connect('mongodb://localhost:27017/数据库名', {
    useNewUrlParser: true
})
// 监听数据库连接
mongoose.connection.once('open', function(error) {
    if (error) {
        // 失败
    } else {
        // 成功，在这里操作数据库
        // 先创建规则表
        const studentsSchema = new mongoose.Schema({
            stu_id: {
                type: String,
                required: true,
                unique: true, // 限制唯一
            },
            hobby: [String], // 字符串数组
            info: Schema.Types.Mixed, // 接收所有类型
            date: {
                type: Date,
                default: Date.now()
            }
        })
        // 再创建模型
        const studentsModel = mongoose.model('students', studentsSchema)
        // 新增
        studentsModel.create({
            stu_id: '001',
            name: '张三',
            age: 19
        }, function(err, data) {
            // data：写入的数据
        })
        // 查询
        // find返回一个数组，没查到返回[]
        studentsModel.find(
            {age: 19}, 
            function(err, data) {}
        )
        // findOne返回一个对象，没查到返回null
        studentsModel.findOne(
            {age: 19}, {name: 1, _id: 0},
            function(err, data) {}
        )
        // 更新
        studentsModel.updateOne(
            {age: 19}, {name: 'lalala'}
            function(err, data) {}
        )
        studentsModel.updateMany(
            {age: 19}, {name: 'lalala'}
            function(err, data) {}
        )
        // 删除
        studentsModel.deleteOne(
            {age: 19},
            function(err, data) {}
        )
        studentsModel.deleteMany(
            {age: 19},
            function(err, data) {}
        )
    }
})
```


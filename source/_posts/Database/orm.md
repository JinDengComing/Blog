---
layout: page
title: 数据库ORM模型
tags: 数据库
categories: database
---



## ORM
Object-Relationl Mapping，它的作用是在关系型数据库和对象之间作一个映射，这样，我们在具体的操作数据库的时候，就不需要再去和复杂的SQL语句打交道，只要像平时操作对象一样操作它就可以了 。
业务逻辑和数据处理逻辑分离，使用对象和数据层的映射关系。


## 以[nodejs](https://nodejs.org/zh-cn)后端为例

### [sequelize](https://www.sequelize.cn/)

1. 首先建立连接
```ts
import { Sequelize } from 'sequelize';
import oracledb from 'oracledb'

const sequelize = new Sequelize(
  'orcl',//名称
  'ly_mh_jcfw',//用户
  'ly_mh_jcfw@2021',//密码
  {
    host: 'localhost',//地址
    port: 'post',//端口
    dialect: 'oracle',//类型
    dialectModule: oracledb,//数据库插件
    pool: {
      max: 5,
      min: 0,
      acquire: 30000,
      idle: 10000
    },
    timezone: "+08:00", //改为标准时区
    dialectOptions: {
      dateStrings: true,
      typeCast: true
    },
    define: {
      timestamps: false,
    }
  }
);

//暴露模块
export default sequelize;
```

2. 定义model层
首先建立模型，关联对应数据库表映射
```ts
import Sequelize from '../db/connection';
import { DataTypes, Model } from 'sequelize'

//表模型
const LogSchema = {
    //字段定义
    LOGID: {
        type: DataTypes.STRING,//数据类型
        allowNull: false,//允许为空
        primaryKey: true,//是否主键
    }
}

class Log extends Model { }

Log.init(
    LogSchema,
    {
        sequelize: Sequelize,
        tableName: 'LY_MH_YM_LOG',
        modelName: 'pageLog',
        timestamps: false,
    }
)

//初始化表结构，查看连接是否成功
Log.sync({ force: false }).then(() => {
    Logger.info('日志表同步成功');
}).catch(() => {
    Logger.error('日志表同步失败');
})

export default Log
```

3. controller 业务层使用
```ts
import { Request, Response } from 'express';
import Log from '../models/log.models';

/**
 * 分页查询用户日志表
 */
const userLogPage = async (req: Request, res: Response) => {
    const { pageNum, pageSize, userId } = req?.query;
    const begin: number = (Number(pageNum) - 1) * Number(pageSize);
    const { rows, count } = await Log.findAndCountAll({
        attributes: [
            'ID',
            'USERID',
            'USERNAME',
            'DELETEPAGE',
            'UPDATEPAGE',
            'SORTPAGE',
            'updatedTime'
        ],
        offset: begin,
        limit: Number(pageSize) || 10,
        where: userId ? { USERID: userId || '' } : {},
        order: [['updatedTime', 'desc']]
    });
    const resultList = rows.map((item: any) => {
        return ({
            id: item.ID,
            userId: item.USERID,
            userName: item.USERNAME,
            isDelete: item.DELETEPAGE,
            isUpdated: item.UPDATEPAGE,
            isSort: item.SORTPAGE,
            updatedDate: item.updatedTime
        })
    })
    const data = new ResultMeta(
        {
            list: resultList,
            current: Number(pageNum),
            size: Number(pageSize),
            total: count
        },
        ResultCodeEnum.success,
        true,
        '查询成功'
    )
    res.json(data);
}

```
#### 查
```ts
//重命名
//lastName 重命名为 aliasLastName
User.findAll({
    attributes: [['lastName', 'aliasLastName'], 'age']
})
// 查询一个
  const zj = await User.findOne({
    where: {
      userName: 'zj'
    }
  })
  console.log('zj: ', zj.dataValues)

  // 查询列
  const zjInfo = await User.findOne({
    attributes: ['userName', 'nickName']
  })

  console.log('info: ', zjInfo.dataValues)

  // 查询集合
  const zjBlogs = await Blog.findAll({
    where: {
      userId: 1
    },
    order: [
      ['id', 'desc']
    ]
  })
  console.log('zjBlogs: ', zjBlogs.map( blog => blog.dataValues))

  // 分页
  const blogPageList = await Blog.findAll({
    order: [
      ['id', 'desc']
    ],
    limit: 2,
    offset: 2
  })

  console.log('blogPageList: ', blogPageList.map( blog => blog.dataValues))

  // 查询总数
  const blogListAndCount = await Blog.findAndCountAll({
    order: [
      ['id', 'desc']
    ],
    limit: 2,
    offset: 2
  }) 
  console.log('blogListAndCount：',
   blogListAndCount.count, // 总数
   blogListAndCount.rows.map(blog => blog.dataValues) 
   )

   // 联表查询
   const blogListWithUser = await Blog.findAndCountAll({
    order: [
      ['id', 'desc']
    ],
    include: [
      {
        model: User,
        attributes: ['userName', 'nickName'],
        where: {
          userName: 'zj'
        }
      }
    ]
  })  
  console.log('blogListWithUser: ',
  blogListWithUser.count,
  blogListWithUser.rows.map(blog => {
    const blogVal = blog.dataValues
    blogVal.user = blogVal.user.dataValues
    return blogVal
  })
  )

  // 联表查询2
  const userListWithBlog = await User.findAndCountAll({
    attributes: ['userName', 'nickName'],
    include: [
      {
        model: Blog
      }
    ]
  }) 
  console.log('userListWithBlog: ',
  userListWithBlog.count,
  userListWithBlog.rows.map(user => {
    const userVal = user.dataValues
    userVal.blogs = userVal.blogs.map(blog => blog.dataValues)
    return userVal
  })
  )
```
#### 增
```ts
  // 创建用户
  const zj = await User.create({
    userName: 'zj',
    password: '123', 
    nickName: 'zj'
  })

  //批量创建
  User.bulkCreate([
    {firstName: 'lxc'},
    {firstName: 'xc'},
]).then(users => {
    console.log('查询结果：', JSON.stringify(users))
})
```
#### 删
```ts
//删除:
const affectedRows = await UserModel.destroy({
  where: { firstName: 'King' }
});
```
#### 改
```ts
User.update({age: 50}, {
    where: {
        id: 1
    }
}).then(users => {
    console.log('查询结果：', JSON.stringify(users))
})
```


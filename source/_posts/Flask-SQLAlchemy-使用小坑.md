title: Flask-SQLAlchemy-使用小坑
date: 2016-03-08 22:05:05
tags:
---
今天在搭建 Flask-SQLAlchemy 的demo时碰到了一些小的坑，记录下来，防止以后再范，本文将持续更新，记录一些Flask-SQLAlchemy 的使用小提示

对于在windows下无法使用pip 安装 mysql-python模块 和 flask-mysql模块的同学，请到这里下载32位或者64位的安装包进行安装 http://pan.baidu.com/s/1qWz7I5e




1、封装db对象
Flask封装了一个db对象，官方示例如下：
```
from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:////tmp/test.db'
db = SQLAlchemy(app)
#your code
```
但是在实际使用过程中发现，当Modal有多个文件时db对象每次都是一个新的实例，无法使用 db.create_all() 来一起创建数据表和使用db.session操作数据对象，于是我这样处理了下，不知道对不对
在Modals文件夹中，创建一个 dbModal.py 文件，用来保存 db实例，代码如下：
```
# -*- coding: utf-8 -*-
from datetime import datetime
from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy
from restapi import app

db = SQLAlchemy(app)
```
这样其他 Modals 和需要数据库访问的文件只要如下引入 db 实例就可以了
from restapi.models.dbModel import db



2、一对多的Modal定义：
经常会碰到这样的场景，比如有用户表：Plat_User（平台用户），还有一个表用来存储用户的多个备选头像：Plat_User_Face（平台用户头像表），这就形成了One（用户表）to Many（头像表），我们可以如下分别定义两张表即可：
```
from datetime import datetime
from flask import Flask

from flask.ext.sqlalchemy import SQLAlchemy
from restapi import app
from restapi.models.dbModel import db
from restapi.models.faceModel import *
from restapi.bussiness.UtilsBl import DumpToDict
import json
```
#vip用户的model
```
class PlatUser(db.Model, DumpToDict):
    #表名
    __tablename__ = 'Plat_User'
    
    Id = db.Column(db.Integer, primary_key=True)
    UserName = db.Column(db.String(50))
    Password = db.Column(db.String(50))
    UserTips = db.deferred(db.Column(db.Text))
    WriteTime = db.Column(db.DateTime, onupdate=datetime.now, default=datetime.now)
    Faces = db.relationship('PlatUserFace', backref='Plat_User', lazy='select')

    def __init__(self, UserName=None, Password=None):
        self.UserName = UserName
        self.Password = Password
```
接下来定义Face表：
```
# -*- coding: utf-8 -*-
from datetime import datetime
from flask import Flask

from flask.ext.sqlalchemy import SQLAlchemy
from restapi import app
from restapi.models.dbModel import *
from restapi.models.UsersModel import *
from restapi.bussiness.UtilsBl import DumpToDict
import json
```
#vip用户的model
```
class PlatUserFace(db.Model, DumpToDict):
    #表名
    __tablename__ = 'Plat_User_Face'
    
    Id = db.Column(db.Integer, primary_key=True)
    FaceUrl = db.Column(db.String(50))
    FaceName = db.Column(db.String(50))
    WriteTime = db.Column(db.DateTime, onupdate=datetime.now, default=datetime.now)
    UserId = db.Column(db.Integer, db.ForeignKey('Plat_User.Id'), nullable=False)


    def __init__(self, UserId=None, FaceUrl=None, FaceName=None):
        self.UserId = UserId
        self.FaceUrl = FaceUrl
        self.FaceName = FaceName
```
这样我们就将表 'Plat_User' 和表 'Plat_User_Face' 进行了一对多的关系映射
注意这边有一个小坑，在定义 Plat_user 表 Faces 时
Faces = db.relationship('PlatUserFace', backref='Plat_User', lazy='select')
#这边 PlatUserFace 一定要写类名，写表名会报错！！！

3、关于 lazy 的设置
lazy设置表示这个属性等到用户使用到才去加载，在我们打开 SQLAlchemy 的调试加断点后，可以清晰的看到，第一次查询和第二次查询的语句，lazy也有几种不通设置，如下：

（1）'select' （默认值）意味着 SQLAlchemy 会在使用一个标准 select 语句 时一气呵成加载那些数据
（2）'joined' 让 SQLAlchemy 当父级使用 JOIN 语句是，在相同的查询中加 载关系。
（3）'subquery' 类似 'joined' ，但是 SQLAlchemy 会使用子查询。
（4）'dynamic' 在你有可能条目时是特别有用的。 SQLAlchemy 会返回另一个查 询对象，你可以在加载这些条目时进一步提取。如果不仅想要关系下的少量条目 时，这通常是你想要的。

这里主要说明一下 dynamic 这个设置，这个设置会在查询返回的结果中保留此字段为一个查询实例，可以用如下代码说明：
```
#获取用户信息

sqlData = PlatUser.query.options(lazyload(PlatUser.Faces)).filter_by(Id=user.Id).all()

#这里需要.all() 才能手动的去获取此用户Face的列表

FaceList = sqlData.Faces.all()
```
当然一般使用 select 就足够了

4、deferred列定义
有些字段内容比较多而且不经常使用，比如是一个新闻的内容字段，在列表页或者封面页我们完全不必读取这个大的字段到内存里，这时候我们就可以设定这个列为 deferred，如下：
```
UserTips = db.deferred(db.Column(db.Text))
```
通常在查询时会不包括这个字段，当然我们也可以手动指定某一个字段不加载，例如：
```
user.sqlData = PlatUser\
            .query\
            .options(defer('Password'))\
            .filter_by(Id=self.userId).first()
```
如果想要一次性加载 UserTips 这个字段，则：
```
user.sqlData = PlatUser\
            .query\
            .options(undefer('UserTips'))\
            .filter_by(Id=self.userId).first()
```
5、多对多的定义：
有这样的需求，用户表和兴趣表，一个用户可以拥有多个兴趣爱好，同时一个兴趣爱好，也能被多个用户拥有，这里还需要支持反向查找的功能，不仅可以通过用户的userid查找到他的兴趣爱好，还可以通过一个兴趣爱好，查找那些有这些兴趣爱好的人
Model我们这样定义：
```
#vip用户的model
class PlatUser(db.Model, DumpToDict):
    #表名
    __tablename__ = 'Plat_User'
    
    Id = db.Column(db.Integer, primary_key=True)
    UserName = db.Column(db.String(50))
    Password = db.Column(db.String(50))
    UserTips = db.deferred(db.Column(db.Text))
    WriteTime = db.Column(db.DateTime, onupdate=datetime.now, default=datetime.now)
    Faces = db.relationship('PlatUserFace', backref='Plat_User', lazy='select')
    Hobbies = db.relationship('PlatUserHobby', secondary='Plat_User_Hobby_Relation',
        backref=db.backref('Plat_User', lazy='select'), lazy='select')

    def __init__(self, UserName=None, Password=None):
        self.UserName = UserName
        self.Password = Password

#关联表
class UserHobby(db.Model):
    __tablename__ = 'Plat_User_Hobby_Relation'

    Id = db.Column(db.Integer, primary_key=True)
    UserId = db.Column(db.Integer, db.ForeignKey('Plat_User.Id'), nullable=False)
    HobbyId = db.Column(db.Integer, db.ForeignKey('Plat_User_Hobby.Id'), nullable=False)
    db.UniqueConstraint('UserId', 'HobbyId', name='unique_1')

    def __init__(self, UserId, HobbyId):
        self.UserId = UserId
        self.HobbyId = HobbyId
```
对于兴趣表，我们根本无需特别的定义，因为在user表我们已经定义了 backref
```
#用户兴趣爱好model
class PlatUserHobby(db.Model, DumpToDict):
    #表名
    __tablename__ = 'Plat_User_Hobby'
    
    Id = db.Column(db.Integer, primary_key=True)
    HobbyName = db.Column(db.String(50))
    WriteTime = db.Column(db.DateTime, onupdate=datetime.now, default=datetime.now)

    def __init__(self, UserId=None, HobbyName=None):
        self.UserId = UserId
        self.HobbyName = HobbyName
```
在使用时，和一对多一样，只需要访问User类的Hobbies属性即可
对于反向查找，只需要如下代码：
```
userHobby.sqlData = PlatUserHobby.query.filter_by(Id=hobbyId).first()
userList = userHobby.sqlData.Plat_User
```
#注意最后.Plat_User即可获得平台用户拥有这个属性的列表

6、连接字符串中密码包含 '@' 等冲突字符处理
在sqlalchemy中，如果连接字符串包含 @ 等特殊字符，直接放在连接字符串中是无法连接到数据库的，可以通过如下的方式转义之后再连接
```
from urllib import quote_plus as urlquote
'mysql://用户名:{0}@ip地址/数据库名'.format(urlquote('坏密码'))
```
7、如果确定创建表的顺序
有时候我们回复发现定义好的表怎么也创建不了，报错说XX表没有找到。
这可能存在多个表互相关联的情况，这时候我们必须先创建表B，才能创建表A和C，保证他们创建顺序的方法很简单：
优先 import B，然后再import A, C ，这样就可以保证表B先create table，然后再create A, C了

持续更新
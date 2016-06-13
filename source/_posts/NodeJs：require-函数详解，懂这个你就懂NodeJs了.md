title: NodeJs：require 函数详解，懂这个你就懂NodeJs了
date: 2015-11-19 02:35:28
tags: NodeJs
---
背景

官方的稳定更详细：地址：http://nodejs.org/api/modules.html。

网上关于NodeJs的论述很多，此处不多说。个人认为，NodeJs的编程思想和客户端Javascript保持了一种理念，没有什么变化，只是增加了“require()”函数，因此只要学好require函数，剩下的问题就是如何更好的使用API了。
##require函数详解

###路径

相对路径之当前目录：./xxx/xxx.js 或 ./xxx/xxx。
相对路径之上级目录：../xxx/xxx.js 或 ../xxx/xxx。
绝对路径：F:/xxx/xxx.js 或 /xxx/xxx.js 或 /xxx/xxx。
###require函数语法

require(路径.扩展名)：
如果 路径.扩展名 存在
执行加载 并 返回
否则
抛出异常
require(路径)：
如果 路径.js 存在
执行加载 并 返回
如果 路径.node 存在
执行加载 并 返回
如果 路径/package.json 存在
执行加载(package.json 中 main属性对应的路径) 并 返回
如果 路径/index.js 存在
执行加载 并 返回
如果 路径/index.node 存在
执行加载 并 返回
抛出异常
require(模块名字)：
如果 模块名字是系统模块
执行加载 并 返回
如果 require(./node_modules/模块名字) 能加载到模块  //参考require(路径)的介绍
执行加载 并 返回
如果 require(../node_modules/模块名字) 能加载到模块  //参考require(路径)的介绍
执行加载 并 返回
沿着目录向上逐级执行require(上级目录/node_modules/模块名字)，如果能加载到模块  //参考require(路径)的介绍
执行加载 并 返回
抛出异常
###代码示例

文件结果
![](http://images.cnitblog.com/blog/492619/201305/07131745-f8df335fb6d2424cbaf997109a647dcd.jpg)
require_study.js中的代码
```
require('module_1_1.js');
require('module_1_2');
require('../node_modules/module_2_1.js');
require('../node_modules/module_2_2');
require('../package_2_1');
require('package_3_1');
require('./node_modules/package_3_2');
require('module_3_1');
require('/node_study/level1/level2/level3/node_modules/module_3_1');
require('module_3_2');
require('/node_study/level1/level2/level3/package_3_3');
require('./package_3_4');
require('./module_3_3');
require('same_name_module');
require('same_name_package');
require('same_name_module_and_package');
```
输出结果
```
module_1_1.js
module_1_2.js
module_2_1.js
module_2_2.js
package_2_1
package_3_1
package_3_2
module_3_1.js
module_3_2.js
package_3_3
package_3_4
module_3_3.js
same_name_module.js in leaf
same_name_package in leaf
same_name_module_and_package.js in leaf module
```
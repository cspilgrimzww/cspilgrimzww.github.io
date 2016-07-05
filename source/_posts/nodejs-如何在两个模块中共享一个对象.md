title: nodejs 如何在两个模块中共享一个对象
date: 2016-07-02 02:38:58
tags: Nodejs
---

### nodejs 如何在两个模块中共享一个对象
今天遇到一个问题,可题简单抽象为如下需求:
* file1.js中定义了一个变量a
* file2.js需要用到file1.js中的这个变量
* a变量在file1.js与file2.js中均可能被加工
* 不论在file1.js或是file2.js中,当需要获得a的值时,都需要保证获得的a值为一致的最新值,即a被file1.js或file2.js最后一次加工过的值.
##### 尝试

 `/file1.js`:
```js
var foo = 0;
var increase=function (){
	foo += 100,
	console.log("Value of foo in file1.js:  "+foo);
}
setTimeout(increase,1000);//1s后改变foo

module.exports=foo;
```

 `/file2.js`:
```js
var foo = require('./file1');

var shout=function(){
	console.log("Value of foo in file2.js:  "+foo);
}
shout();//0s时输出foo
setTimeout(shout,2000);//2s时输出foo
```
运行:
```
>node file2.js
```
输出:
```
Value of foo in file2.js:  0
file1 has increased the foo to:  100
Value of foo in file2.js:  0
```
file1在修改foo为100后,file2再次读取仍为0
##### 修改代码

 `/file1.js`:
```js
var foo = ["0"];

var shout=function(){
	console.log("Value of foo in file1.js:  "+foo);
}
var increase=function (){
	foo .push("100"),
	console.log("file1 has increased the foo to:  "+foo);
}
shout();
setTimeout(increase,1000);//1s后改变a值
setTimeout(shout,2000);
setTimeout(shout,4000);

module.exports=foo;
```

 `/file2.js`:
```js
var foo = require('./file1');

var shout=function(){
	console.log("Value of foo in file2.js:  "+foo);
}
var increase=function (){
	foo .push("100"),
	console.log("file2 has change the foo to:  "+foo);
}
shout();
setTimeout(shout,2000);
setTimeout(increase,3000);//
setTimeout(shout,4000);
```
运行:
```
>node file2.js
```
输出:
```
Value of foo in file1.js:  0
Value of foo in file2.js:  0
file1 has increased the foo to:  0,100
Value of foo in file1.js:  0,100
Value of foo in file2.js:  0,100
file2 has change the foo to:  0,100,100
Value of foo in file1.js:  0,100,100
Value of foo in file2.js:  0,100,100
```
当exports暴露出的变量类型为number或string时,会拷贝一个副本给引用者,而exports一个list或dict,则会将该list或dict的实例地址暴露路给引用者,要想将number或string也作为共享变量,可以将number或string放入list或map来实现.

##### 示例
 `/file1.js`:
```js
var foo = {a:0};

var shout=function(){
	console.log("Value of foo in file1.js:  "+foo.a);
}
var increase=function (){
	foo.a += 1,
	console.log("file1 has increased the foo to:  "+foo.a);
}
shout();
setTimeout(increase,1000);//1s后改变a值
setTimeout(shout,2000);
setTimeout(shout,4000);

module.exports=foo;
```

 `/file2.js`:
```js
var foo = require('./file1');

var shout=function(){
	console.log("Value of foo in file2.js:  "+foo.a);
}
var increase=function (){
	foo.a += 1,
	console.log("file2 has change the foo to:  "+foo.a);
}
shout();
setTimeout(shout,2000);
setTimeout(increase,3000);//
setTimeout(shout,4000);
```
运行:
```
>node file2.js
```
输出:
```
Value of foo in file1.js:  0
Value of foo in file2.js:  0
file1 has increased the foo to:  1
Value of foo in file1.js:  1
Value of foo in file2.js:  1
file2 has change the foo to:  2
Value of foo in file1.js:  2
Value of foo in file2.js:  2
```
这样,就可以在两个模块中共享同一对象
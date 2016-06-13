title: python 多线程编程 之threading模块
date: 2015-12-27 11:48:42
tags:
---
实现python的多线程编程有很多模块可以使用，比如thread模块，threading模块。但thread模块已经过时，不推荐使用，threading 模块提供了比thread模块更方便的同步机制，推荐使用.

以下代码实现threading.Thread的一个子类，实现了线程对不同函数的通用行。将其保存为MyThread.py以供随时使用
```
﻿﻿#!/usr/bin/env python
#coding=utf-8


import threading
from time import ctime


class MyThread(threading.Thread):
'''
make a threading class for some function
'''
def __init__(self, func, args, name = ''):
threading.Thread.__init__(self)
self.name = name
self.func = func
self.args = args


def getResult(self):
return self.res


def run(self):
print 'starting', self.name, 'at:', ctime()

self.res = apply(self.func, self.args)


print self.name,'finished at:', ctime()
```
test_mythread.py脚本，比较了递归求斐波那契数列，阶乘和累加和函数的运行。脚本先在单线程中运行着三个函数，然后在多线程中做同样的事。以说明多线程的好处。在脚本中使用了上面子类化过的MyThread
```
#!/usr/bin/env python
from MyThread import MyThread
from time import sleep,ctime


def fib(x):
sleep(0.005)
if x < 2: return 1
return (fib(x-2)+fib(x-1))


def fac(n):
sleep(0.1)
if n < 2:
return 1
return n*fac(n-1)


def sum(n):
sleep(0.1)
if n<1:
return 0
return n+sum(n-1)


funcs = [fib, fac, sum]
n=12


def main():
nfuncs = range(len(funcs))


print '*** SINGLE TRHEADS'


for i in nfuncs:
print 'starting ',funcs[i].__name__, 'at:', ctime()
print funcs[i](n)
print funcs[i].__name__,'finished at:', ctime()


print '\n### MULTIPLE THREADS'
threads = []
for i in nfuncs:
t = MyThread(funcs[i], (n,), name = funcs[i].__name__)
threads.append(t)


for i in nfuncs:
threads[i].start()


for i in nfuncs:
threads[i].join()
print threads[i].getResult()
print 'ALL DONE'


if __name__ == '__main__':
main()
```
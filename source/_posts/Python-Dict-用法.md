title: Python Dict 用法
date: 2015-6-4 00:20:22
tags: Python
---
申明 m={};
[python]dictionary方法说明
|Operation           |Result                                                |Notes     |
|--------------------|:----------------------------------------------------:|:--------:|
|len(a)              |the number of items in a 得到字典中元素的个数         |          |
|a[k]                |the item of a with key k 取得键K所对应的值            |(1), (10) |
|a[k] = v            |set a[k] to v 设定键k所对应的值成为v                  |          |
|del a[k]            |remove a[k] from a 从字典中删除键为k的元素            |(1)       |
|a.clear()           |remove all items from a 清空整个字典                  |          |
|a.copy()            |a (shallow) copy of a 得到字典副本                    |          |
|k in a              |True if a has a key k, else False 字典中存在键k则为返回True，没有则返回False|(2)|
|k not in a          |Equivalent to not k in a 字典中不存在键k则为返回true,反之返回False |(2)|
|a.has_key(k)        |Equivalent to k in a, use that form in new code 等价于k in a |    |
|a.items()           |a copy of a’s list of (key, value) pairs 得到一个键，值的list |(3)|
|a.keys()            |a copy of a’s list of keys 得到键的list               |(3)        |
|a.update([b])       |updates (and overwrites) key/value pairs from b从b字典中更新a字典，如果键相同则更新，a中不存在则追加 |(9)|
|a.fromkeys(seq[, value]) |Creates a new dictionary with keys from seq and values set to value | (7)|
|a.values()          |a copy of a’s list of values                          |(3)        |
|a.get(k[, x])       |a[k] if k in a, else x                                |(4)        |
|a.setdefault(k[, x])|a[k] if k in a, else x (also setting it)              |(5)        |
|a.pop(k[, x])       |a[k] if k in a, else x (and remove k)                 |(8)        |
|a.popitem()         |remove and return an arbitrary (key, value) pair      |(6)        |
|a.iteritems()       |return an iterator over (key, value) pairs            |(2), (3)   |
|a.iterkeys()        |return an iterator over the mapping’s keys            |(2), (3)   |
|a.itervalues()      |return an iterator over the mapping’s values          |(2), (3)   |


```
 #字典的添加、删除、修改操作
dict = {"a" : "apple", "b" : "banana", "g" : "grape", "o" : "orange"}
dict["w"] = "watermelon"
del(dict["a"])
dict["g"] = "grapefruit"
print dict.pop("b")
print dict
dict.clear()
print dict

#字典的遍历
dict = {"a" : "apple", "b" : "banana", "g" : "grape", "o" : "orange"}
for k in dict:
    print "dict[%s] =" % k,dict[k]

#字典items()的使用
dict = {"a" : "apple", "b" : "banana", "c" : "grape", "d" : "orange"}
#每个元素是一个key和value组成的元组，以列表的方式输出
print dict.items()

#调用items()实现字典的遍历
dict = {"a" : "apple", "b" : "banana", "g" : "grape", "o" : "orange"}
for (k, v) in dict.items():
    print "dict[%s] =" % k, v

#调用iteritems()实现字典的遍历
dict = {"a" : "apple", "b" : "banana", "c" : "grape", "d" : "orange"}
print dict.iteritems()
for k, v in dict.iteritems():
    print "dict[%s] =" % k, v
for (k, v) in zip(dict.iterkeys(), dict.itervalues()):
    print "dict[%s] =" % k, v
   


#使用列表、字典作为字典的值
dict = {"a" : ("apple",), "bo" : {"b" : "banana", "o" : "orange"}, "g" : ["grape","grapefruit"]}
print dict["a"]
print dict["a"][0]
print dict["bo"]
print dict["bo"]["o"]
print dict["g"]
print dict["g"][1]

 

dict = {"a" : "apple", "b" : "banana", "c" : "grape", "d" : "orange"}
#输出key的列表
print dict.keys()
#输出value的列表
print dict.values()
#每个元素是一个key和value组成的元组，以列表的方式输出
print dict.items()

dict = {"a" : "apple", "b" : "banana", "c" : "grape", "d" : "orange"}
it = dict.iteritems()
print it

#字典中元素的获取方法
dict = {"a" : "apple", "b" : "banana", "c" : "grape", "d" : "orange"}
print dict
print dict.get("c", "apple")         
print dict.get("e", "apple")
#get()的等价语句
D = {"key1" : "value1", "key2" : "value2"}
if "key1" in D:
    print D["key1"]
else:
    print "None"

#字典的更新
dict = {"a" : "apple", "b" : "banana"}
print dict
dict2 = {"c" : "grape", "d" : "orange"}
dict.update(dict2)
print dict
#udpate()的等价语句
D = {"key1" : "value1", "key2" : "value2"}
E = {"key3" : "value3", "key4" : "value4"}
for k in E:
    D[k] = E[k]
print D
#字典E中含有字典D中的key
D = {"key1" : "value1", "key2" : "value2"}
E = {"key2" : "value3", "key4" : "value4"}
for k in E:
    D[k] = E[k]
print D

#设置默认值
dict = {}
dict.setdefault("a")
print dict
dict["a"] = "apple"
dict.setdefault("a","default")
print dict

#调用sorted()排序
dict = {"a" : "apple", "b" : "grape", "c" : "orange", "d" : "banana"}
print dict  
#按照key排序 
print sorted(dict.items(), key=lambda d: d[0])
#按照value排序 
print sorted(dict.items(), key=lambda d: d[1])

#字典的浅拷贝
dict = {"a" : "apple", "b" : "grape"}
dict2 = {"c" : "orange", "d" : "banana"}
dict2 = dict.copy()
print dict2


#字典的深拷贝
import copy
dict = {"a" : "apple", "b" : {"g" : "grape","o" : "orange"}}
dict2 = copy.deepcopy(dict)
dict3 = copy.copy(dict)
dict2["b"]["g"] = "orange"
print dict
dict3["b"]["g"] = "orange"
print dict
```
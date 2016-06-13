title: python脚本中的 /usr/bin/python
date: 2016-01-17 09:06:56
tags:
---
#估计有不少人注意过一些python脚本开头有这么行东东：
```
 #!/usr/bin/python
 ```
它是用来干嘛的？貌似没有它对脚本功能也没啥影响。它是用来指定用什么解释器运行脚本以及解释器所在的位置。

以test.py为例，脚本内容如下：
```
def test():
        print 'hello, world'

if __name__ == "__main__":
        test()
```
运行脚本:
```
python test.py
```
输出：
```
hello, world
```
换一种方法运行：
```
./test.py
```
会提示出错，文件无可执行权限：
```
-bash: ./test.py: Permission denied
```

将文件设为可执行：
```
chmod +x test.py```
继续运行：
```
./test.py
```
提示：
```
./test.py: line 1: syntax error near unexpected token `('
./test.py: line 1: `def test():'
```
那是因为系统默认该脚本是shell脚本，把它当shell语句执行，当然失败了。

在前面加上
```
#!/usr/bin/python
```
申明l这是个python脚本，要用python解释器来运行：
```
./test.py
```
输出：
```
hello, world
```

这个东东常用在cgi脚本中，apache启动cgi脚本时就靠它来知道这是个python脚本，执行它需要的python解释器路径在哪里。
有时候写` #!/usr/bin/python `还是不行，很简单，因为python解释器没有装在/usr/bin/目录，改成其所在目录就行了，或者更通用的方法是：
```
 #!/usr/bin/env python
 ```
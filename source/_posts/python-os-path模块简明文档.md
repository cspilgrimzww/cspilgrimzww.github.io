title: python os.path模块简明文档
date: 2015-11-27 03:01:40
tags: Python
---

`os.path.abspath(path)`
取path的绝对目录，实际上就是os.getcwd()+path
`os.path.basename(path)`
取path最后的文件或文件名。如何path以／结尾，那么就会返回空值
`os.path.commonprefix(list)`
返回list中，所有path共有的最长的路径。
如：
```
>>> os.path.commonprefix(['/home/td','/home/td/ff','/home/td/fff'])
'/home/td'
os.path.dirname(path)
```
返回path的目录名，注意如果path以\结尾，那么返回的就是path.其实就是os.path.split(path)返回的前半部分
`os.path.exists(path)`
如果path存在返回True;如果path不存在，或者没有执行os.stat()的权限，或者已损坏的链接会返回False
`os.path.lexists(path)`
与os.path.exists(path)的不同是如果有损坏的链接会返回True
`os.path.expanduser(path)`
将～等用用户的家目录进行替换
`os.path.expandvars(path)`
接受环境变理的扩展，path中可以使用环境变量
如：
```
>>> os.path.expandvars('$PATH')
'/usr/lib64/qt-3.3/bin:/usr/kerberos/bin:/usr/lib64/ccache:/usr/local/bin:/usr/bin:/bin:/usr/local         
/sbin:/usr/sbin:/sbin:/home/jimin/bin'
>>> os.path.expandvars('$HOME')
'/home/jimin'
os.path.getatime(path)
```
返回最后一次进入此path的时间。如果os.stat_float_times() 返回True, 那么返回的结果是一个浮点值
`os.path.getmtime(path)`
返回这个path最后一次修改的时间。
`os.path.getctime(path)`
返回path的大小
`os.path.isabs(path)`
如果path是绝对路径，返回True
`os.path.isfile(path)`
如果path是常规文件，返回True.
```
os.path.isdir(path)
os.path.islink(path)
os.path.ismount(path)
```
如果path是一个挂载点，返回True
`
os.path.join(path1[, path2[, ...]])
`
如：
`
>>> os.path.join('/home/aa','/home/aa/bb','/home/aa/bb/c')
'/home/aa/bb/c'
`
```
os.path.normcase(path)
os.path.normpath(path)
os.path.realpath(path)
```
返回path的真实路径，去除符号链接
`os.path.relpath(path[, start])`
返回一个“相关路径”，当前目录或者可选的start
Return a relative filepath to path either from the current directory or from an optional start point.
如：
```
>>> os.path.relpath('/home/jimin','/usr/lib/')
'../../home/jimin'
```
`os.path.samefile(path1, path2)`
如果path1与path2是相同的文件或目录，返回真
`os.path.sameopenfile(fp1, fp2)`
如果fp1和fp2指向的是同一个文件，返回True
`os.path.samestat(stat1, stat2)`
如果 stat tuple stat1和stat2指向同一个文件，返回真。stat tuple结构是由fstat()、lstat()、stat()产生的
```
os.path.split(path)
os.path.splitdrive(path)
os.path.splitext(path)
os.path.splitunc(path)
os.path.walk(path, visit, arg)
os.path.supports_unicode_filenames 
```
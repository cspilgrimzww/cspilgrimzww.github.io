title: python删除文件示例分享
date: 2015-9-20 01:40:50
tags: Python
---
删除文件
代码如下:
```
os.remove(   filename )   # filename: "要删除的文件名"
```
产生异常的可能原因:
(1)filename 不存在
(2)对filename文件， 没有操作权限或只读。
删除文件夹下所有文件和子文件夹 :
代码如下:
```
import os  
def delete_file_folder(src):  
    '''delete files and folders''' 
    if os.path.isfile(src):  
        try:  
            os.remove(src)  
        except:  
            pass 
    elif os.path.isdir(src):  
        for item in os.listdir(src):  
            itemsrc=os.path.join(src,item)  
            delete_file_folder(itemsrc)  
        try:  
            os.rmdir(src)  
        except:  
            pass 
  if __name__=='__main__':  
      dirname=r'G:\windows' 
    print delete_file_folder(dirname)
```
或者使用shutil模块的rmtree函数，也可以级联删除
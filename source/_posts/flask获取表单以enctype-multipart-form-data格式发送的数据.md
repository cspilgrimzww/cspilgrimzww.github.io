title: flask获取表单以enctype=multipart/form-data格式发送的数据
date: 2015-4-4 03:06:43
tags: Flask
---
最早的HTTP POST是不支持文件上传的，给编程开发带来很多问题。但是在1995年，ietf出台了rfc1867,也就是《RFC 1867 -Form-based File Upload in HTML》，用以支持文件上传。所以Content-Type的类型扩充了multipart/form-data用以支持向服务器发送二进制数据。因此发送post请求时候，表单<form>属性enctype共有二个值可选，这个属性管理的是表单的MIME编码：

 ①application/x-www-form-urlencoded(默认值)
 ②multipart/form-data
其实form表单在你不写enctype属性时，也默认为其添加了enctype属性值，默认值是enctype="application/x- www-form-urlencoded".

当需要上传文件时，需要在form 标签中包含enctype="multipart/form-data"和method="post"属性，表单就会以二进制形式提交到服务器
flask 通过`request.form.get("keyword")`获取上文提到的两种格式的数据，通过`request.files.get("filename")`来获取单个文件通过`reuquest.files.getlist("files[]")`来获取多个上传文件

在html代码中，多个上传文件可以通过两种方式获取
①设置多个`<input>`标签,需保证name属性值相同，服务器通过reuquest.files.getlist("files[]")获取这一组文件
```
<input type="file" name ="file[]">
<input type="file" name ="file[]">
<input type="file" name ="file[]">
```
②设置一个`<input type="file" multiple="multiple" name="file[]">`，如此可向一个添加文件按钮中添加多个文件，服务器依然通过reuquest.files.getlist("files[]")获取这一组文件


**示例代码**：
HTML：
```
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8"></meta>
	<title>测试表单</title>
</head>
<body>
	<form method="POST" action="http://127.0.0.1:5000/form/new" enctype="multipart/form-data">
		<p>ID: <input type="text" name="id" /></p>
		<p>body: <input type="text" name="body" /></p>
		<p>param1: <input type="text" name="param1" /></p>	
		<input type="file" name="imgs[]">
		<input type="file" name="imgs[]">
		<input type="file" name="imgs[]">
		<br />
		<input type="submit" value="Submit" />	
	</form>
</body>
</html>
```
fpython代码：

```
import os
from config import basedir #defined in config
                           #basedir = os.path.abspath(os.path.dirname(__file__))
                           
@api.route("/form/new")
def new_form():
#save image file
    imgs = request.files.getlist()
    path = basedir+"/static/imgs/"
    if not os.path.exists(path):
        os.mkdir(path)
    for img in imgs:
        file_path = path+img.filename
        img.save(file_path)
    #save other parameter
    id = request.form.get("id")
    body = request.form.get("body")
    param1 = request.form.get("body")
    print "id:"+str(id)
    print "body:"+str(body)
    print :param1:"+str(param1)
    return jsonify({})
```
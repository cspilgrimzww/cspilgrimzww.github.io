title: Python使用QRCode模块生成二维码
date: 2015-8-18 21:35:49
tags: python
---
最近在工作中需要批量的生成二维码，发现python里已经带有可以满足类似需求的包，现将其记录下来
QRCode官网
https://pypi.python.org/pypi/qrcode/5.1

简介
python-qrcode是个用来生成二维码图片的第三方模块，依赖于 PIL 模块和 qrcode 库。

简单用法
```
import qrcode 
img = qrcode.make('hello, qrcode')
img.save('test.png')
```

高级用法

```
import qrcode 
qr = qrcode.QRCode(     
    version=1,     
    error_correction=qrcode.constants.ERROR_CORRECT_L,     
    box_size=10,     
    border=4, 
) 
qr.add_data('hello, qrcode') 
qr.make(fit=True)  
img = qr.make_image()
img.save('123.png')
```
 

参数含义：
version：值为1~40的整数，控制二维码的大小（最小值是1，是个12×12的矩阵）。 如果想让程序自动确定，将值设置为 None 并使用 fit 参数即可。

error_correction：控制二维码的错误纠正功能。可取值下列4个常量。
　　ERROR_CORRECT_L：大约7%或更少的错误能被纠正。
　　ERROR_CORRECT_M（默认）：大约15%或更少的错误能被纠正。
　　ROR_CORRECT_H：大约30%或更少的错误能被纠正。

box_size：控制二维码中每个小格子包含的像素数。

border：控制边框（二维码与图片边界的距离）包含的格子数（默认为4，是相关标准规定的最小值）
# 二维码生成脚本
本地生成二维码图片脚本，并切图片可以带上相关说明信息。
需要安装python、ImageDraw、qrcode
创建make_qrcode.py文件，内容如下：
```python
#!/usr/bin/env python
# coding: utf-8

import os, qrcode, sys
from PIL import Image,ImageDraw,ImageFont

def make_qrcode(data, save_path='./qrcode.png', border=3, image_size=(300, 300), icon_path='', factor=5):
    # 生成二维码主体
    qr = qrcode.QRCode(error_correction=qrcode.constants.ERROR_CORRECT_L, border=border, box_size=10, version=1)
    qr.add_data(data)  # 向二维码写入数据
    qr.make()
    qrcode_image = qr.make_image().resize(image_size, Image.ANTIALIAS).convert('RGBA')

    if icon_path and os.path.exists(icon_path):  # 在二维码中间贴上图标
        icon_size = int(image_size[0] / factor), int(image_size[1] / factor)
        icon = Image.open(icon_path).resize(icon_size, Image.ANTIALIAS).convert('RGBA')
        icon_margin = int((image_size[0] - icon_size[0]) / 2), int((image_size[1] - icon_size[1]) / 2)
        mask = Image.new('RGBA', icon_size, color='white')
        qrcode_image.paste(mask, icon_margin, mask)
        qrcode_image.paste(icon, icon_margin, icon)

    # 保存二维码为文件
    if not os.path.isdir(os.path.dirname(save_path)):
        os.makedirs(os.path.dirname(save_path))
    qrcode_image.save(save_path)
    
def add_codeinfo(data, save_path='./qrcode.png', border=3, image_size=(380, 400), icon_path='', factor=5,text='',subtext=''):
    make_qrcode(data=data, save_path=save_path, border=border, image_size=(300, 300), icon_path=icon_path, factor=factor)
    oriImg = Image.open(save_path)
    oriImg.resize((300, 300))
    #背景图
    bgimg = Image.new('RGB', image_size, (255, 255, 255))
    #将二维码放在底图上
    bgimg.paste(oriImg, (40, 10))#将二维码放在底图上
    draw = ImageDraw.Draw(bgimg)
#    draw.fontmode = "1"
    #设置字体
    if text:
#        text = text.encode('utf8').decode('utf8')
        font = ImageFont.truetype('PingFang.ttc', 16,encoding="utf-8")
#        w, h = len(text)*14,14 # 计算汉字占用的像素大小
        draw.text((40, 320), unicode(text,'UTF-8'), (0, 0, 0), font=font)
    if subtext:
        font = ImageFont.truetype('PingFang.ttc', 14,encoding="utf-8")
        draw.text((40, 360), unicode(subtext,'UTF-8'), (169, 169, 169), font=font)

    #把字添加到图片上
    bgimg = bgimg.convert('RGB')
    bgimg.save(save_path)

if __name__ == '__main__':

    # url = 'itms-services://?action=download-manifest&url=https://storage.jd.com/tripitaka-jenkins/Release/JDFocus_iOS/JDFocus/JDFocus_iOS_Release_V1.7.0_master_20201125154526.plist'
    # qrcodeName = './huangzhen.png'
    # make_qrcode(data=url, save_path=qrcodeName, icon_path='/Users/huangzhen122/Development/JDFocus/focus_ios/JDFocus/JDFocus/Resources/50@2x.png')
    
    url = sys.argv[1]
    qrcodeName = sys.argv[2]
    icon_path = sys.argv[3]
    titleText = sys.argv[4]
    subTitleText = sys.argv[5]
    # icon_path='./focus_ios/JDFocus/JDFocus/Resources/50@2x.png'
    add_codeinfo(data=url, save_path=qrcodeName, icon_path=icon_path,text=titleText,subtext=subTitleText)

```
使用示例：
```shell
python3 ./make_qrcode.py  '需要生成二维码的内容（或Url）' "./保存为的文件名.png" ./二维码中间的icon.png "说明信息一" "说明信息二"  

```
<img src="https://raw.githubusercontent.com/xueliangwd/leon/main/images/leon_blog_qrcode.jpg" alt="" title="">

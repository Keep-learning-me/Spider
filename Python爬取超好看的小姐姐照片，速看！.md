​	大家好，[我](https://blog.csdn.net/weixin_56432636?spm=1011.2124.3001.5343)今天给大家分享一个爬取美女图片的小爬虫，网站是[搜好图](https://www.wahaotu.com/meinv/)，话不多说，先看网站里面的图片：


![在这里插入图片描述](https://img-blog.csdnimg.cn/2cdc21895203437e8e130b605503273a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA55yL5oiR5ZKv,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

​	怎么样，是不是福利满满，那么接下来我们就就先进行这个网站的分析，

###### 	首先，我们按`F12`或者`Ctrl + Shift + I`打开开发者工具，切换到网络，然后点击`Fetch/XHR`选项，刷新网页，查看这些图片的来源，刷新之后我们发现，在这个选项卡里面没有一个活动，
![在这里插入图片描述](https://img-blog.csdnimg.cn/b69ddf40913e4d36a6899be2cfffc4c1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA55yL5oiR5ZKv,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

###### 	那么我们几乎可以肯定，这些个图片详情页的链接就是在网页中自带的。接下来我们随便点开一个链接，再重复上诉操作，看该图片详情页里面的图片是否为链接所直接能获取到的，在查看了之后，我们可以知道，这些图片链接和图片都是我们访问它的`url`可以直接获得的。

###### 	接下来我们再次观察它的主页，可以看到下面可以切换页码，我们切换到第二页和第三页，发现它们的`url`分别是

```
https://www.wahaotu.com/meinv/list_2.html
https://www.wahaotu.com/meinv/list_3.html
```

###### 	接着我们切换回第一页，发现它的`url`变成了

```
https://www.wahaotu.com/meinv/list_1.html
```

###### 	这几乎就是摆在我们眼前的规律呀，根据这个规律，每一页的`url`就可以轻而易举的获得了。

​	接着我们开始写代码吧：

```python
# 首先
import requests
import os
# 在这个地方，我们之前得出了结论是在list后面的数字就是它的页数，那么我们就可以写一个表达式来将这个链接进行格式化
page_num = 1
first_url = 'https://www.wahaotu.com/meinv/list_%d.html'

detail_urls = []
# 定义一个获得所有页面的图片链接的函数
def get_page_data():
    if page_num <= 5:
        get_url = format(index_url % page_num)
        response = requests.get(url=get_url, headers=headers)
        # 在这我们使用的是xpath解析
        tree = etree.HTML(response.text)
        detail_list = tree.xpath('/html/body/div[2]/ul/li')
        for detail_data in detail_list:
            detail_url = detail_data.xpath('./span/a/@href')[0]
            detail_urls.append(detail_url)
        print(f'第{page_num}页解析完成')
        page_num += 1
        get_page_data(page_num)

get_page_data(page_num=page_num)
    
```

​	这样，我们就可以通过循环来依次获得每一个页面的所有图片链接了

​	获得`url`之后，我们继续访问每一个`url`，可以看到：

![在这里插入图片描述](https://img-blog.csdnimg.cn/6d19160bd8ee4bfba3182dde4a757f61.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA55yL5oiR5ZKv,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


​	每一个图片的链接都是在`p`标签中，接着我们继续对这个`p`标签进行`xpath`解析(`xpath`路径可以直接在浏览器中复制得来)，获得每一张图片的实际`url`：

```python
pic_urls = []
for detail_url in detail_urls:
    response = requests.get(url=detail_url, headers=headers).text
    tree = etree.HTML(response)
    pic_list = tree.xpath('/html/body/div[2]/div[2]/div[1]/div[1]/div[2]/p')
    for pic in pic_list:
        if pic.xpath('./img/@src'):
            pic_url = 'https://www.wahaotu.com' + pic.xpath('./img/@src')[0]
            pic_urls.append(pic_url)
```

​	接着，我们对每一个图片的`url`发起请求，获得二进制数据，再将其存储下来（当然首先要创建一个文件夹）：

```python
if not os.path.exists('./图片合集'):
    os.makedirs('./图片合集')
for pic_url in pic_urls:
    response = requests.get(url=pic_url, headers=headers).content
    pic_name = pic_url.split('/')[-1]
    with open(f'./图片合集/{pic_name}', 'wb')as f:
        f.write(response)
    print(pic_name + '下载完成')
```

​	那么到这儿就大功告成了！哈哈哈，如果有不懂的小伙伴或者说想要这些代码的小伙伴可以直接私信我噢，在这附上`xpath`的基本用法:
![在这里插入图片描述](https://img-blog.csdnimg.cn/84e398090dc94c4f827c42e61ef23fd3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA55yL5oiR5ZKv,size_18,color_FFFFFF,t_70,g_se,x_16#pic_center)

记得点赞收藏哈！拜。

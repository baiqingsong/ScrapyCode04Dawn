# scrapy 项目4 爬取动态网页Ajax，获取js

* [项目地址](#项目地址)
* [spider代码](#spider代码)


## 项目地址
[v.qq.com/cover/q/qviv9yyjn83eyfu/u0016gbvtgc.html](https://v.qq.com/x/cover/qviv9yyjn83eyfu/u0016gbvtgc.html)

获取网页中的评论信息

## spider代码
```
# _*_ coding:utf-8 _*_
import json
import re

import scrapy


class QqtvSpider(scrapy.Spider):
    name = "qqtv"
    start_urls = ["https://v.qq.com/x/cover/qviv9yyjn83eyfu/x00168rm3xi.html"]
    comment_url = 'http://coral.qq.com/article/%s/comment?commentid=0&reqnum=10'
    sns_url = 'http://sns.video.qq.com/fcgi-bin/video_comment_id?otype=json&op=3&vid='

    def parse(self, response):
        vid = response.xpath('//div[@class="mod_episode"]/span[@class="current item"]/@id')[0].extract()
        print vid
        sns_url = self.sns_url + vid
        yield scrapy.Request(sns_url, callback=self.parse_id)

    def parse_id(self, response):
        id = re.search('"comment_id":"(.*?)"', response.body, re.S).group(1)
        commentUrl = self.comment_url % id
        yield scrapy.Request(commentUrl, callback=self.parse_comment)

    def parse_comment(self, response):
        # with open("result.txt", "wb") as f:
        #     f.write(response.body)
        # jsDict = json.loads(response.body_as_unicode())
        jsDict = json.loads(response.body)
        jsData = jsDict['data']
        comments = jsData['commentid']
        for comment in comments:
            try:
                print comment['content']
            except:
                print "error"
                print comment
```

spider是拼接的地址，首先是初始地址  
但是要找到评论信息再js中sns_url，但是这个地址少了个id  
需要找到comment_url，这个地址需要对应的一个vid  
需要从初始地址中找到，这个直接在页面中查找即可
---
title: 【phantomjs系列】Phantomjs正确打开方式
date: 2017-03-31 11:30:58
comments: true
tags: 
- Phantomjs
categories: 爬虫技术
password:
copyright: true
---
<blockquote class="blockquote-center">你是如何走出人生的阴霾的？
多走几步</blockquote>

　　前段时间分析了[Selenium+Phantomjs的使用方法以及性能优化问题](http://thief.one/2017/03/01/Phantomjs%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/)，期间也分析了利用[Selenium+phantomjs爬虫爬过的一些坑问题](http://thief.one/2017/03/01/Phantomjs%E7%88%AC%E8%BF%87%E7%9A%84%E9%82%A3%E4%BA%9B%E5%9D%91/)。然而在使用phantomjs的过程中，并没有正真提升phantomjs的性能，爬虫性能也没有很好的提升。经过网友的提醒，发现其实是使用phantomjs的方法出了问题，因此无论怎么优化，都不能从根本上去提升性能。那么本篇就来好好说说，Phantomjs正确的打开方式。
<!--more -->
### 抛弃selenium+phantomjs
　　之前我一直使用selenium去使用phantomjs，原因是因为selenium封装了phantomjs一部分功能，selenium又提供了python的接口模块，在python语言中可以很好地去使用selenium，间接地就可以使用phantomjs。然而，我现在要说的是，是时候抛弃selenium+phantomjs了，原因之一此封装的接口很久没有更新了（没人维护了），原因之二selenium只实现了一部分phantomjs功能，且很不完善。

### phantomjs APi
　　通过查看phantomjs官方介绍，我们可以发现phantomjs的功能异常强大，绝不仅仅是selenium封装的功能那么简陋。phantomjs提供了很多种APi，具体可以查看：[phantomjs api介绍](http://thief.one/2017/03/13/Phantomjs-Api%E4%BB%8B%E7%BB%8D/)，其中最常用的要属Phantomjs WebService与Phantomjs WebPage，前者用于开启http服务，后者用于发起http请求。

### Phantomjs正确使用方式
正确打开方式应该使用phantomjs Webservice作为一种web服务的形式（api）,将其与其他语言分离开来（比如python）。

#### 设计流程：
　　Python通过http请求下发任务，Phantomjs Webservice获取任务后去处理，处理完以后再将结果返回给Python。任务调度、存储等复杂操作交给Python去做，Python可以写成异步并发去请求Phantomjs Webservice，需要注意的是目前一个Phantomjs Webservice只支持10个并发。但我们可以在一台服务器上多开几个phantomjs Webservice启用不同的端口即可，或者可以多台服务器做个集群，用nginx做反向代理。

#### Phantomjs Webservice
新建test.js，写入如下代码：
```bash
//此js用来获取网页源码

var system=require('system');  //get args
var args=system.args;
if (args.length ===2){
    var port=Number(args[1]);
}
else{
    var port=8080;
}
var webserver = require('webserver');
var server = webserver.create()
var service = server.listen(port, function(request, response) {
    try{
        var postRaw=request.postRaw;
        var aaa=new Array();
        aaa=postRaw.split("=");
        var url=aaa[0];
        var md5_url=aaa[1];
        url=decodeURIComponent(url);

        // 创建page
        var webPage = require('webpage');
        var page = webPage.create();
        page.settings.resourceTimeout = 20000;//timeout is 20s

        // 页面错误捕捉
        page.onError = function(msg, trace) {
            console.log("[Warning]This is page.onError");
            var msgStack = ['ERROR: ' + msg];
            if (trace && trace.length) {
                msgStack.push('TRACE:');
                trace.forEach(function(t) {
                  msgStack.push(' -> ' + t.file + ': ' + t.line + (t.function ? ' (in function "' + t.function +'")' : ''));
                });
            }
            // console.error(msgStack.join('\n'));
        };

        // phantomjs错误捕捉
        phantom.onError = function(msg, trace) {
            console.log("[Warning]This is phantom.onError");
            var msgStack = ['PHANTOM ERROR: ' + msg];
            if (trace && trace.length) {
              msgStack.push('TRACE:');
              trace.forEach(function(t) {
                msgStack.push(' -> ' + (t.file || t.sourceURL) + ': ' + t.line + (t.function ? ' (in function ' + t.function +')' : ''));
              });
            }
              console.error(msgStack.join('\n'));
              phantom.exit(1);
        };
        // 打开网页，获取源码
        page.open(url, function (status) {

            console.log('Target_url is ' + url);  //输出待检测的网站url

            if(status=='success'){
                var current_url = page.url;
                var body= page.content;
            }
            else
            {
              var body="";
              var current_url="";
            }
              response.status=200;
            //  response.write(body);  //返回获取到的网页源码
              response.write(current_url); //返回当前的网页url
              page.close();
              response.close();
        });

    }
    catch(e)
    {
      console.log('[Error]'+e.message+'happen'+e.lineNumber+'line');
    }
});
```
作用：处理http请求，获取url，进行截图或者获取源码操作。
使用：
```bash
phantomjs.exe test.js 8080
```
会在本地开启web服务，端口为8080。

#### Python Client
新建http_request.py，写入如下代码：
```bash
#! -*- coding:utf-8 -*-

import requests
import hashlib
import base64
from multiprocessing.dummy import Pool

class http_request:

  def __init__(self,port="8080"):
    self.url="http://localhost:"+port
  
  def getwebbody(self,domain):
    '''
    获取网页源代码
    '''
    base_domain=base64.b64encode(domain)
    md5_domain=hashlib.md5(base_domain).hexdigest()
    payload={domain:md5_domain}

    try:
      response=requests.post(self.url,data=payload,timeout=30).content
      return response
    except requests.exceptions.ConnectionError:
      print "requests connection error"
    except Exception,e:
      print e
    return

if __name__=="__main__":
  port="8080"
  cur=http_request(port)
  domain_list=["http://thief.one"]*10

  def test(domain):
    print "Result_url is ",cur.getwebbody(domain)

  pool = Pool(processes=10)
  for domain in domain_list:  #并发下发任务
    pool.apply_async(test, args=(domain,))   #维持执行的进程总数为10，当一个进程执行完后添加新进程.
  pool.close()
  pool.join()
```
作用：异步并发下发任务。

#### 运行截图
运行python以后，异步下发10个任务，Phantomjs服务器端接收到url并开始处理，并发处理10个任务并输入结果。
![](/upload_image/20170331/1.png)

#### 异常处理

现象：截图为黑屏
原因：网页还没加载完，就开始截图了
解决：在代码中open以后判断status值，判断网页是否加载完毕。

现象：程序出错--windows报错
解决：更换最新版本的phantomjs

现象：内存占用过大，导致报错停止phantomjs进程
原因：phantomjs没有释放内容
解决：代码中open以后，要open.close();

现象：没有截图成功
原因：用了page.close，因为onloadfinished是非阻塞的，因此要将page.close放在open代码层内部。


>转载请说明出处:[Phantomjs正确打开方式 | nMask'Blog](http://thief.one/2017/03/31/Phantomjs正确打开方式/)
本文地址：http://thief.one/2017/03/31/Phantomjs正确打开方式/

### 传送门

>[【phantomjs系列】phantomjs正确打开方式](http://thief.one/2017/03/31/Phantomjs%E6%AD%A3%E7%A1%AE%E6%89%93%E5%BC%80%E6%96%B9%E5%BC%8F/)
[【phantomjs系列】phantomjs api介绍](http://thief.one/2017/03/13/Phantomjs-Api%E4%BB%8B%E7%BB%8D/)
[【phantomjs系列】selenium+phantomjs爬过的那些坑](http://thief.one/2017/03/01/Phantomjs%E7%88%AC%E8%BF%87%E7%9A%84%E9%82%A3%E4%BA%9B%E5%9D%91/)
[【phantomjs系列】selenium+phantomjs性能优化](http://thief.one/2017/03/01/Phantomjs%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/)



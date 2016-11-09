title: web客户端存储方式
date: 2016-11-09 18:30:30
---



### 存储方式
近来，在使用angular.js建立web应用时，使用到$cacheFactory缓存服务对应用中的一些数据进行缓存，以提高网站性能，减少数据请求量，突然想对web客户端存储方式做一个总结。
存储方式：
* cookie
* localStorage
* sessionStorage

我们来一一说明。
<!--more-->
假设有这样一种情况，在某个用例流程中，由A页面跳至B页面，若在A页面中采用JS用变量temp保存了某一变量的值，在B页面的时候，同样需要使用JS来引用temp的变量值，对于JS中的全局变量或者静态变量的生命周期是有限的，当发生页面跳转或者页面关闭的时候，这些变量的值会重新载入，即没有达到保存的效果。这个问题，就可以使用cookie或者localStorage来保存该变量的值。
#### cookie
cookie的使用在前端开发中是很普遍的，基本人尽皆知，简单说明一下，不做太多赘述。
cookie是存于用户硬盘的一个文件，这个文件通常对应于一个域名，当浏览器再次访问这个域名时，便使这个cookie可用。因此，cookie可以跨越一个域名下的多个网页。
另外，稍微了解一下cookie的结构，简单地说：cookie是以键值对的形式保存的，即key=value的格式。各个cookie之间一般是以“;”分隔。
cookie的写入，读取，删除可参考：[http://www.jb51.net/article/14566.htm](http://www.jb51.net/article/14566.htm)。
### localStorage
localStorage是html5提供的新的存储数据的方法。
在localStorage之前，客户端的数据是由cookie保存的。但是cookie不适合大量数据的存储，因为它们由每个对服务器的请求来传递，这使得 cookie 速度很慢而且效率也不高。而localStorage，数据不是由每个服务器请求传递的，而是只有在请求时使用数据。它使在不影响网站性能的情况下存储大量数据成为可能。
localStorage有以下几个特点：
1.localStorage是一个普通对象，任何对象的操作都适用。
2.localStorage对象的属性值只能是字符串。
3.localStorage支持的默认空间大小为5M,现代浏览器支持良好。除了IE７及以下不支持外，其他标准浏览器都完全支持。
4.localStorage本身带有方法有：
　　添加键值对：localStorage.setItem(key,value)
　　获取键值：localStorage.getItem(key)
　　删除键值对：localStorage.removeItem(key)。
　　清除所有键值对：localStorage.clear()。
　　获取localStorage的属性名称（键名称）：localStorage.key(index)。
5、没有时间限制的数据存储。不主动清除，localStorage一直存在。



另外，假设有这样一种情况，在某个用例流程中，我们希望数据在页面强制刷新的时候或者关闭的时候，数据自动清空，不污染下次进入页面的数据。在页面服务不中断的时候保存数据。这个时候就可以用sessionStorage。

### sessionStorage
sessionStorage和localStorage拥有完全一样的特点，唯一的区别是，sessionStorage在页面强制刷新的时候或者关闭的时候，数据自动清空。而localStorage需要主动清除。

既然是$cacheFactory缓存服务引发的对存储方式的思考。当然需要来介绍一下$cacheFactory缓存服务：

### AngularJs $cacheFactory 缓存服务
完全适用于sessionStorage的应用场景。可用于缓存angular模版和其他数据。$cacheFactory存储的数据也可以在不同controller中传递。
$cacheFoctory
用于生成一个用来存储缓存对象的服务，并且提供对对象的访问。
$cacheFactory.Cache
一个用于存储和检索数据的缓存对象。主要使用$http和脚本指令来缓存模板和其他数据。
该服务有以下方法：
put(key,value);
在缓存对象中插入一个键值对(key,value)。

get(key);
在缓存对象中通过指定key获取对应的值。

romove(key);
在缓存对象中通过指定key删除对应的值。

removeAll();
删除缓存对象中所有的键值对。

destroy();
销毁这个缓存对象。

info();
获取缓存对象信息（id，size）。








---
title: Session与Cookie
date: 2017-02-27 15:15
---

Cookie和Session都为了用来保存状态信息，都是保存客户端状态的机制，它们都是为了解决HTTP无状态的问题而所做的努力。
<!-- more -->
Session可以用Cookie来实现，也可以用URL回写的机制来实现。

Cookie和Session有以下明显的不同点：

1）Cookie将状态保存在客户端，Session将状态保存在服务器端；

2）Cookies是服务器在本地机器上存储的小段文本并随每一个请求发送至同一个服务器。网络服务器用HTTP头向客户端发送cookies，在客户终端，浏览器解析这些cookies并将它们保存为一个本地文件，它会自动将同一服务器的任何请求缚上这些cookies。

3）Session是针对每一个用户的，变量的值保存在服务器上，用一个sessionID来区分是不同用户session变量,这个值是通过用户的浏览器在访问的时候返回给服务器，当客户禁用cookie时，这个值也可能设置为由get来返回给服务器；

4）就安全性来说：当你访问一个使用session 的站点，同时在自己机器上建立一个cookie，建议在服务器端的SESSION机制更安全些.因为它不会任意读取客户存储的信息。

Session机制

Session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。

当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否已包含了一个session标识 - 称为 session id，如果已包含一个session id则说明以前已经为此客户端创建过session，服务器就按照session id把这个 session检索出来使用（如果检索不到，可能会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个 session id将被在本次响应中返回给客户端保存。

Session的实现方式

1 ） 使用Cookie来实现

服务器给每个Session分配一个唯一的JSESSIONID，并通过Cookie发送给客户端。

当客户端发起新的请求的时候，将在Cookie头中携带这个JSESSIONID。这样服务器能够找到这个客户端对应的Session。

 

 

 

 2 ）使用URL回显来实现

URL回写是指服务器在发送给浏览器页面的所有链接中都携带JSESSIONID的参数，这样客户端点击任何一个链接都会把JSESSIONID带给服务器。
如果直接在浏览器中输入url来请求资源，Session是匹配不到的。

Tomcat对 Session的实现，是一开始同时使用Cookie和URL回写机制，如果发现客户端支持Cookie，就继续使用Cookie，停止使用URL回写。如果发现Cookie被禁用，就一直使用URL回写。jsp开发处理到Session的时候，对页面中的链接记得使用 response.encodeURL() 。

 

回顾完Session和Cookie，我们来说说为什么手机端与服务器交互没有实现在同一session下？

1）原因很简单，就是因为android手机端在访问web服务器时，没有给http请求头部设置sessionID，而使用web浏览器作为客户端访问服务器时，在客户端每次发起请求的时候，都会将交互中的sessionID：JSESSIONID设置在Cookie头中携带过去，服务器根据这个sessionID获取对应的Session,而不是重新创建一个新Session(除了这个Session失效)。

 

以java.net.HttpURLConnection发起请求为例：

获取Cookie： 

URL url = new URL(requrl);
 HttpURLConnection con= (HttpURLConnection) url.openConnection(); 
// 取得sessionid. 
String cookieval = con.getHeaderField("set-cookie"); 
String sessionid; 
if(cookieval != null) { 
sessionid = cookieval.substring(0, cookieval.indexOf(";")); 
}

//sessionid值格式：JSESSIONID=AD5F5C9EEB16C71EC3725DBF209F6178，是键值对，不是单指值

发送设置cookie： 

URL url = new URL(requrl);
HttpURLConnectioncon= (HttpURLConnection) url.openConnection(); 
if(sessionid != null) { 
con.setRequestProperty("cookie", sessionid); 
}

只要设置了sessionID，这样web服务器在接受请求的时候就会自动搜索对应的session了，从而保证了在同一会话Session。
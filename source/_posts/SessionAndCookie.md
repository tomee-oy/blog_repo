---
title: Session和Cookie：相爱相杀
date: 2017-01-16 15:57:34
tags: [js,session,cookie]
categories: 技术
---
&emsp;&emsp;关于session和cookie，以前在大学就开始接触，但从来没有对这两个东西有一个系统的认识。如今想来，是时候对它们进行一下梳理了。
## 一、认识cookie
&emsp;&emsp;cookie是存在于客户端的状态跟踪机制。由于HTTP是无状态协议，当客户端向服务器发送请求，服务器对请求进行响应，随后连接断开，当下一次同一个客户端再次访问服务器的时候，服务器并不知道这个请求就是上一次的发起者，所有的请求数据都要重新构造一遍，服务端的效率相当低下。
&emsp;&emsp;所以，cookie就发挥了作用。客户端浏览器每一次向服务器发送请求的时候，都要在http请求头中携带关于cookie的信息随之发送到服务器，服务器获得cookie的信息并根据信息判断此次请求的发送者是不是之前发送过请求的“人”，以此判断做出相应的反应。
```bash
function getCookie(Cookies[] cookies,String name){
    if(cookies != null){
        for(Cookie cookie : cookies){
            if(cookie.getName() == name){
                return cookie.getValue();
            }
        }
    }
    return null;
}

Cookies[] cookies=request.getCookies();
String account=getCookie(cookies,"account") ;
String password=getCookie(cookies,"password");
if(account==null){
    response.addCookie(new Cookie("account","ouyang"));
}
if(password == null){
    response.addCookie(new Cookie("password","123456"))
}
response.setHeader("Set-Cookie");
```
&emsp;&emsp;打个比方，我第一次去某城市乘地铁，由于没有当地的交通卡，每次乘地铁都必须要手动买票，相当不方便，于是地铁管理处给我办了张交通卡，以后每次乘地铁刷一下卡就可以直接进站了。
&emsp;&emsp;cookie也相当于服务器给每个访问它的客户端办了一个通行卡。就拿登录来说，我第一次登录的时候服务端检测到我的cookie是空的，那么它就把我当前登录的用户名和密码设置到cookie中并写入返回的http响应头中，客户端接收到cookie内容后将其保存在客户端，下一次请求的时候http请求头中自然就携带了已经验证过的用户名和密码，在服务器可以直接登录了。
&emsp;&emsp;cookie包含名称、值、过期时间、域、路径这几个重要的内容。下面简单介绍一下：

> **名称（name）**：cookie名称，  
**值（value）**：名称对应的cookie值，  
**过期时间（Expires)**：若不设置过期时间，则表示cookie的过期时间为浏览器会话时间，一旦浏览器关闭，cookie就会消失。如果设置了过期时间，cookie将会被储存到本地硬盘中，在设置的这个时间到来之前，无论浏览器是否关闭，cookie都会存在。当设置为负数时，表示cookie在浏览器关闭时立即消失，设置为0时表示删除该cookie。  
**域名（Domain)**：限制可以访问该cookie的域名，设置的时候别忘了要包含最前面的点（“.”）如cookie.setDomain(".baidu.com")，  
**路径（Path)**：限制哪些路径下的文件可以访问到该cookie，  
**是否使用安全协议传输（secure)**：布尔值。如果为true，表示是否只在SSH连接时传递该cookie

&emsp;&emsp;cookie具有不可跨域名性。同时，cookie也是不安全的，别人可以分析存放在本地的cookie并进行cookie欺骗。cookie的大小不能超过4K，因为过大的cookie在浏览器和服务器之间交互相当耗带宽，影响性能。对于这种只能存储基本小容量信息的cookie，远远不能满足更广泛的信息存储需求，于是，session登场了。
## 二、认识session
&emsp;&emsp;session是服务端存储客户信息的机制。它以一种散列表的结构存储信息。尽管session打破了cookie容量的限制，但是，它也需要一种将客户端与服务端对应起来的机制，否则，服务器无法知道哪个session是属于当前登录用户的。这就形成了cookie与session合作共赢的格局。
&emsp;&emsp;当浏览器向服务器发起请求时，服务器首先去检查http请求头的cookie中是否包含一个sessionid，如果有，就根据sessionid找到对应的session，读取其中的信息。如果没有，就为当前请求客户创建一个session，并把唯一标识的sessionid存入cookie返回给服务器。等到下次再请求的时候，请求信息中就自然携带了对应的sessionid，并在服务器找到对应的session信息。
&emsp;&emsp;由于cookie可以在客户端被禁止，所以，此时要想sessionid传到服务器，就需要启用URL地址重写，sessionid以字符串的形式追回到url末尾，服务器会解析出sessionid，效果跟cookie携带是一样的。

&emsp;&emsp;更多关于session和cookie的信息在[这里](http://blog.csdn.net/fangaoxin/article/details/6952954/)
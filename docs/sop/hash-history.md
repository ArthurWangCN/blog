---
tag: 前端
---

# 为什么hash模式不会出现404，而history模式会出现404？

1）Hash模式
在hash路由模式下，URL中的Hash值（#后面的部分）用来表示应用的状态或路由信息。当用户切换路由时，只有Hash部分发生变化，并没有向服务器发出请求，就做到了浏览器对于页面路由的管理。

Hash模式下，URL和路由路径由#号分隔：http://example.com/#/about?query=abc

当#后面的路径发生变化时，会触发浏览器的hashchange事件，通过hashchange事件监听到路由路径的变化，从而导航到不同的路由页面。

Hash模式#后面的路径并不会作为URL出现在网络请求中。例如对于输入的[example.com/#/about] ，实际上请求的URL是[example.com/] ，所以不管输入的Hash路由路径是什么，实际网络请求的都是主域名或IP:Port

2）History模式

History路由模式下，调用浏览器HTML5中historyAPI来管理导航。URL和路径是连接在一起的，路由的路径包含在请求的URL里面，路由路径作为URL的一部分一起发送。

History模式下，URL路由格式为：http://example.com/about&query=abc

当我们向服务器发出请求时，服务器会请求对应的路径的资源

综上，当我们打开入口文件index.html的路径时，切换url此时是本地路由，访问正常，但是当我们处于非入口页面时，刷新浏览器，此时发出请求，由于服务器就找不到资源路径了，变成了404。

而对于Hash模式来说，总是请求的根路径，所以不会出现这种情况。

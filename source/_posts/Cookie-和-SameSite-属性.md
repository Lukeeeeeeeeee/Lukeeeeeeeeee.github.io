---
title: Cookie 和 SameSite 属性
date: 2020-04-02 14:16:29
tags: [浏览器]
---

#### HTTP

一般我们都会说“HTTP 是一个无状态的协议”，不过要注意这里的 HTTP 其实是指 HTTP 1.x，而所谓无状态协议，简单的理解就是即使同一个客户端连续两次发送请求给服务器，服务器也识别不出这是同一个客户端发送的请求，这导致的问题就比如你加入了一个商品到购物车中，但是因为识别不出时同一个客户端，刷新一下页面就没有了...

#### Cookie

为了解决 HTTP 无状态导致的问题，后来出现了 Cookie。不过这样说可能会让你产生一些误解，首先无状态并不是不好，有优点，但也会导致一些问题。而 Cookie 的存在也不是为了解决通讯协议无状态的问题，只是为了解决客户端与服务器会话状态的问题，这个状态是指后端服务的状态而非通讯协议的状态。

#### Cookie 介绍

引用维基百科：

> Cookie (复数形式 Cookies)，类型为[小型文本文件]，只某些网站为了辨别用户身份而储存在用户本地终端上的数据。

作为一段一般不超过 4KB 的小型文本数据，它由一个名称(Name)，一个值(Value)和其他几个用于控制 Cookie 有效期、安全性、使用范围的可选属性组成，这些涉及的属性后面会介绍。

#### Cookie 的查看

我们可以在浏览器的开发者工具中查看到当前页面的 Cookie：

![](https://pic.downk.cc/item/5e85893f504f4bcb04d14eda.png)

尽管我们在浏览器里查看到了 Cookie，这并不意味着 Cookie 文件只是存放在浏览器里的。实际上，Cookie 相关的内容还可以存到本地文件里，就比如说 Mac 下的 Chrome，存放目录就是 `~/Library/Application Support/Google/Chrome/Default`，里面会有一个名为 Cookie 的数据库文件，你可以使用 sqlite 软件打开它。存放在本地的好处就在于即使你关闭了浏览器，Cookie 依然可以生效。

#### Cookie 的设置

那 Cookie 是怎么设置的呢？简单来说就是：

1. 客户端发送 HTTP 请求到服务器
2. 当服务器收到 HTTP 请求时，在响应头里面添加一个 Set-Cookie 字段
3. 浏览器收到响应后保存下来 Cookie
4. 之后对该服务器每一次请求中都通过 Cookie 字段将 Cookie 信息发送给服务器

以 `https://main.m.taobao.com/` 为例

请求返回的 Response Header 可以看到 Set-Cookie 字段

![](https://pic.downk.cc/item/5e85aac3504f4bcb04eb5ba1.png)

再查看 Cookie

![](https://pic.downk.cc/item/5e85ab4d504f4bcb04ebc3af.png)

可以在 Request Header 中看到 Cookie 字段

![](https://pic.downk.cc/item/5e85abe5504f4bcb04ec369c.png)

#### Cookie 的属性

从下面这张图中，可以看到 Cookie 相关的一些属性

![](https://pic.downk.cc/item/5e85ae03504f4bcb04edd13e.png)

需要注意的点：

###### Name/Value

用于 JavaScript 操作 Cookie 的时候注意对 Value 进行编码处理。

###### Expires

Expires 用于设置 Cookie 的过期时间。

`Set-Cookie: id=a3etds; Expires=Thu, 16 Apr 2020 06:21:20 GMT;`

当缺少 Expires 属性时，表示是会话性 Cookie 如上图 Expires 的值为 Session，表示的就是会话性 Cookie，当为会话性 Cookie 的时候，值保存在客户端内存中，并在用户关闭浏览器时失效。需要注意的是，有些浏览器提供了会话恢复功能，这种情况下，即使关闭了浏览器，会话期 Cookie 也会被保留下来，就好像浏览器从来没有关闭一样。

与会话性 Cookie 相对应的是持久性 Cookie，持久性 Cookie 会保存在用户的硬盘中，直到过期或者清除 Cookie。这里值得注意的是，设定的日期和时间只与客户端相关，而不是服务器。

###### Max-Age

Max-Age 用于设置在 Cookie 失效之前需要经过的秒数。

`Set-Cookie: id=a3etds; Max-Age=604800;`

Max-Age 可以是正数、负数甚至是 0.
如果 Max-Age 属性为正数，浏览器会将其持久化，即写到对应的 Cookie 文件中。
如果 Max-Age 属性为负数，则表示该 Cookie 只是一个会话性 Cookie。
如果 Max-Age 属性为 0，则会立即删除这个 Cookie。

假如 Expires 和 Max-Age 都存在，则 Max-Age 优先级更高。

###### Domain

Domain 指定了 Cookie 可以送达的主机名。假如没有指定，那么默认值为当前文档访问地址中的主机部分(但是不包含子域名);

像淘宝首页设置的 Domain 就是 .taobao.com，这样无论是 a.taobao.com 还是 b.taobao.com 都可以使用 Cookie。

需要注意的是，不能跨域设置 Cookie。

###### Path

Path 指定了一个 URL 路径，这个路径必须出现在要请求的资源的路径中才可以发送 Cookie 首部。比如设置 `Path=/a`, `/a/b` 下的资源会带 Cookie 首部，`/c`则不会携带 Cookie 首部。

Domain 和 Path 标志共同定义了 Cookie 的作用域：即 Cookie 应该发送给哪些 URL。

###### Secure 属性

标记为 Secure 的 Cookie 只应通过被 HTTPS 协议加密过的请求发送给服务端。使用 HTTPS 安全协议，可以保护 Cookie 在浏览器和 Web 服务器间的传输过程中不被窃取和篡改。

###### HTTPOnly

设置 HTTPOnly 属性可以防止客户端脚本通过 document.cookie 等方式访问 Cookie，有助于避免 XSS 攻击。

###### SameSite

SameSite 是最近非常值得一提的内容，因为 2 月份发布的 Chrome 80 版本中默认屏蔽了第三方的 Cookie。

###### 作用

SameSite 属性可以让 Cookie 在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击。

##### 属性值

SameSite 可以有下面三种值：

- **Strict** 仅允许一方请求携带 Cookie，即浏览器将只发送相同站点请求的 Cookie，即当前网页 URL 与请求目标 URL 完全一致

- **Lax** 允许部分第三方请求携带 Cookie

- **None** 无论是否跨站都会发送 Cookie

之前默认是 None，Chrome 80 后默认为 Lax。

###### 跨域和跨站

首先要理解的一点就是**跨站**和**跨域**是不同的。[同站(same-site)/跨站(cross-site)] 和 [第一方(first-party)/第三方(third-party)](https://www.jianshu.com/p/7e7d41cd2bac)是等价的。但是与浏览器同源策略(SOP)中的[同源(same-origin)/跨域(cross-origin)]是完全不同的概念。

同源策略的同源是指两个 URL 的协议/域名/端口一致。例如，https://www.taoba.com/pages/...，它的协议是 `https`，域名是 `www.taobao.com`，端口是 `443`。

同源策略作为浏览器的安全基石，其[同源]判断是比较严格的，相对而言，Cookie 中的 [同站]判断就比较宽松：只要两个 URL 的 eTLD+1 相同即可，不需要考虑协议和端口。其中，eTLD 表示有效顶级域名，注册于 Mozilla 维护的公共后缀列表(Public Suffix List)中，例如，.com、.co.uk、.github.io 等。eTLD+1 则表示，有效顶级域名 + 二级域名，例如 `taobao.com`。

举几个例子，`www.taobao.com` 和 `www.baidu.com` 是跨站，`www.a.taobao.com` 和 `www.b.taobao.com` 是同站，`a.github.io` 和 `b.github.io` 是跨站(注意是跨站。有效顶级域名：.github.io，二级域名不同)

###### 改变

接下来看下从 None 改成 Lax 到底影响了哪些地方的 Cookie 的发送?

| 请求类型  | 实例                                  | 以前        | Strict | Lax         | None        |
| --------- | ------------------------------------- | ----------- | ------ | ----------- | ----------- |
| 链接      | `<a href="..."></a>`                  | 发送 cookie | 不发送 | 发送 cookie | 发送 cookie |
| 预加载    | `<link ref="prerender" href="..." />` | 发送 cookie | 不发送 | 发送 cookie | 发送 cookie |
| get 表单  | `<form method="GET" action="...">`    | 发送 cookie | 不发送 | 发送 cookie | 发送 cookie |
| post 表单 | `<form method="POST" action="...">`   | 发送 cookie | 不发送 | 不发送      | 发送 cookie |
| iframe    | `<iframe src="..."></iframe>`         | 发送 cookie | 不发送 | 不发送      | 发送 cookie |
| AJAX      | `$.get("...")`                        | 发送 cookie | 不发送 | 不发送      | 发送 cookie |
| Image     | `<img src="...">`                     | 发送 cookie | 不发送 | 不发送      | 发送 cookie |

从上表可以看出，对大部分 web 应用而言，Post 表单，iframe，AJAX，Image 这四种情况从以前的跨站发送第三方 Cookie，改变成不发送。

Post 表单：应该的，学 CSRF 总会据表单的例子。

iframe：iframe 嵌入的 web 应用有很多是跨站的，都会受到影响。

AJAX：可能会影响到部分前端取值的行为和结果。

Image：图片一般放 CDN，大部分情况不需要 Cookie，故影响有限。但如果引用了需要鉴权的图片，可能会受影响。

除了这些还有 script 的方式，这种方式也不会发送 Cookie，像淘宝的大部分请求都是 jsonp，如果涉及到跨站也有可能会被影响。

###### 解决

解决方案就是设置 SameSite 为 none。

不过也有两点要注意的地方：

- HTTP 接口不支持 SameSite = none

如果你想加 SameSite=none 属性，那么该 Cookie 就必须同时加上 Secure 属性，表示只有在 HTTPS 协议下该 Cookie 才会被发送。

- 需要 UA 监测，部分浏览器不能加 SameSite=none

IOS 12 的 Safari 以及老版本的一些 Chrome 会把 SameSite=none 识别成 SameSite=Strict，所以服务器必须在下发 Set-Cookie 响应头时进行 User-Agent 检测，对这些浏览器不下发 SameSite=none 属性。

###### Cookie 的作用

1. 会话状态管理(如用户登录状态、购物车、游戏分数或其它需要记录的信息)
2. 个性化设置(如用户自定义设置、主题等)
3. 浏览器行为跟踪(如跟踪分析用户行为等)

###### Cookie 的缺点

1. 储存空间很小(只有4KB - 10KB左右)
2. 安全性不够高，可能被截取篡改
3. 有些状态不可能保存在客户端，如为了防止重复提交表单，需要在服务器端保存一个计数器。如果计数器保存在客户端，那么就没有任何作用。

> https://github.com/mqyqingfeng/Blog/issues/157

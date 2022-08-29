#### 1. 用户输入url并回车

1.1 浏览器进程判断输入的是url还是关键字，当是关键字时，会传给当前搜索引擎，当是url时，浏览器会自动补全url，如输入的是baidu.com，会补全为http://www.baidu.com/。

```
URI（Uniform Resource Identifier）统一资源标识符，用于指定一个资源（包含URL、URN）
URL（Uniform Resource Locator）统一资源定位符，使用地址来定位一个资源
URN（Uniform Resource Name）统一资源名称，使用名称来定位一个资源
```

1.2 浏览器进程构建请求信息，然后通过进程间通信将url请求发送给网络进程。

1.3 网络进程接收到url请求后，先检查浏览器缓存中是否有该请求的缓存结果和缓存标识（强制缓存），如果有且缓存有效，则直接将缓存返回给浏览器进程。如果不存在强制缓存，再查找是否存在协商缓存标识，如果有则把协商缓存标识写入请求头中，然后网络进程会向服务器发起http请求。

##### 浏览器缓存

from memory cache：缓存在内存中，速度快，容量小，关闭页面后会消失，一般是js、图片等文件
form disk cache：缓存在硬盘中，速度较慢，容量大，时效性长，不会随着页面关闭而消失，一般是css文件

`ctrl+f5` 强制刷新，跳过强制缓存和协商缓存
`f5` 跳过强制缓存，但会检查协商缓存

**强制缓存**

```
> Expire
  是服务器返回的到期时间，需要比较系统的本地时间来判断是否过期（系统的本地时间不准确、可修改）
> Cache-Control
  优先级高于Expire
  public：客户端、服务端都可以缓存
  private：只有客户端可以缓存
  no-cache：客户端缓存，但是否使用缓存需要通过协商缓存验证
  no-store：不使用缓存
  max-age：缓存有效时间（相对时间），精确到秒
```

**协商缓存**

强制缓存失效后进入协商缓存，浏览器携带缓存标识发起HTTP请求，资源更新时返回200，并缓存新资源和缓存标识，资源未更新时返回304，使用缓存的资源

```
> Last-Modified/If-Modified-Since
  Last-Modified返回资源文件在服务器上最后的修改时间（秒）
  浏览器再次发起请求时会携带If-Modified-Since，值为上一次请求返回的Last-Modified，服务器会将收到的If-Modified-Since与当前资源文件最后的修改时间做对比
  注：精度到秒，如果文件的更新频率在秒以下，则缓存会无法使用；如果文件是动态生成，且文件内容不变，则文件的更新时间一直是生成的时间。
> ETag/If-None-Match
  优先级高于Last-Modified/If-Modified-Since
  ETag返回资源文件的唯一标识
  浏览器再次发起请求时会携带If-None-Match，值为上一次请求返回的ETag，服务器会将收到的If-None-Match与当前资源文件的ETag做对比
  注：生成资源文件的唯一标识会消耗部分性能
```

#### 2. 网络进程发起http请求

##### 2.1 进行DNS域名解析

会先查找DNS缓存，浏览器 => 操作系统（hosts） => 路由器 => ISP的本地DNS服务器，从客户端到本地DNS服务器的查询过程是**递归查询**，不存在DNS缓存，则需要本地DNS服务器向DNS的各层级服务器发起**迭代查询**。

```
为什么URL需要解析（编码）？
如果传输的参数值中包括一些特殊符号如：&或=，则会产生歧义
```

```
为什么要解析成IP？
TCP/IP网络是通过IP地址来确定通信对象
注：当服务器使用了虚拟主机功能时，则无法直接通过IP地址访问，因为虚拟主机是寄存于服务器上的一个或多个没有实体的服务器，所以访问虚拟主机的域名时，需要先解析域名获取实体主机的IP地址，然后实体主机再通过域名转发给对应的虚拟主机。
```

```
DNS（域名系统）用于将域名转换成IP地址
分布式：整个DNS由许多DNS服务器组成，每个DNS服务器各自保存着域名与IP的映射关系。
分层：DNS服务器分为三个层级，根DNS服务器(.)、顶级域DNS服务器(.cn/com/org/...)、权威DNS服务器(.baidu/...)，从上往下查找，最终在权威DNS服务器上找到映射关系。
```

##### 2.2 建立TCP连接

利用IP与服务器建立TCP连接前，要进行TCP三次握手，三次是确认双方是否都有发送和接收能力的最小次数

```
标记位：
> SYN = 1，希望创建连接
> ACK = 1, 确认号有效
> FIN = 1, 希望断开连接
> RST = 1, TCP连接异常，需要断开连接
序列号seq：一个随机产生的序号，解决数据包乱序问题
确认号ack：表示上一个序号的数据已成功接收，期望获取此序号往后的数据
```

```
1. 客户端发送包（SYN=1，seq=x）给服务器，客户端进入SYN_SEND状态
2. 服务端收到客户端的包，并向客户端发送包（SYN=1，ACK=1，seq=y，ack=x+1），服务端进入SYN_RCVD状态
3. 客户端收到包后再发送包（ACK=1，ack=y+1），发送后客户端进入ESTABLISHED状态，服务端收到后也进入ESTABLISHED状态
```

如果使用HTTPS协议，则需要进行SSL/TLS握手，使服务端与客户端建立安全的通信连接

```
1. 浏览器生成client random随机字符串，将浏览器的加密算法列表和client random发送给服务器
2. 服务器生成server random随机字符串，将所选择的加密算法、server random、数字证书等信息发送给浏览器
3. 浏览器从数字证书中验证服务器的合法性并获取公钥，并生成一个新的key，通过公钥对key进行加密后发送给服务器
4. 服务器接收到加密的key后，使用私钥进行解密，至此双方都拿到了同一个key
5. 双方使用之前选择的加密算法、client random、server random、key，生成密钥进行对称加密
```

##### 2.3 发送网络请求

###### HTTP/HTTPS

```
HTTP1.0
> 每次请求都要重新建立TCP连接（慢启动）
> 引入了请求头和响应头来支持传输多种数据类型
> 提供缓存机制Expire、状态码、Cookie
```

```
HTTP1.1
> 默认开启持久连接Connection: keep-alive，同一个域名允许建立多个持久连接
> 支持虚拟主机，请求头增加Host字段，使得不同域名能配置在同一个IP地址的服务器上
> 支持动态生成的内容的传输，1.0提供的Content-Length无法支持，1.1提供了Transfer-Encoding: chunk，将数据分成若干数据块，每次传输的数据化会附带上一个数据块的长度，最后使用一个0长度的数据块作为发送结束的标志
> 新增缓存机制Cache-Control、Last-Modified/If-Modified-Since、Etag/If-None-Match
```

```
HTTP2.0
> 只使用一个TCP持久连接
> 使用二进制传输数据
> 多路复用
  提供二进制分帧层，将请求的信息分帧处理后转成一个个带有ID编号的帧，服务器将这些帧通过ID组装成完整请求信息后处理请求，然后将响应数据也分帧处理后发送给浏览器，浏览器根据ID组装响应数据，然后返回给对应的请求
> 头部压缩
  服务器和浏览器共同维护一张首部表来追踪和储存之前请求的请求头键值对，每次发送请求的HEADERS帧会对比首部表，只发送新的键值
> 服务端推送
  服务器不再被动地接收请求、响应请求，可以主动为客户端发送消息，如当浏览器请求一个HTML文件时，服务器会顺带发送该HTML中引用的资源
```

```
HTTP与HTTPS的区别
1. HTTP明文传输，HTTPS密文传输
2. HTTP默认端口80，HTTPS默认端口443
3. HTTPS需要CA证书
```

###### cookie、localStorage、sessionStorage

```
> cookie可设置失效时间，未设置时默认关闭浏览器后清除
  localStorage只能代码或手动清除
  sessionStorage在关闭tab页或浏览器时清除
> cookie可存放4KB
  localStorage、sessionStorage可存放5MB
> cookie默认会在同域的http请求头中携带
  localStorage、sessionStorage不参与和服务器的通信
```

###### 请求类型

```
GET查，POST改，PUT增，DELETE删
GET和POST从约定上看，GET请求参数放在url上，POST请求参数放在body上
GET和POST从规范上看，GET一般用于查询操作，是幂等的，POST一般用于增删改操作，是不幂等的
GET的参数长度受浏览器URL长度的限制
GET会被浏览器主动缓存
```

###### 处理跨域

```
协议http:// 子域名www 主域名.baidu.com : 端口号
(主机名www 次级域名.baidu 顶级域名.com 根域名.)
同源：协议、域名、端口号相同
同站点：协议、主域名相同
```

```
跨域请求如何携带cookie？
   配置请求头withCredentials: true
   配置响应头Access-Control-Allow-Origin: */http...
   配置响应头Access-Control-Allow-Credentials: true
```

**开发环境跨域**

```javascript
// 通过配置对应脚手架的开发服务器
devServer: {
	proxy: {
        "/api": {
        	target: "http://test.com/", // 跨域的地址，可以是域名和IP
        	changeOrigin: true, // target是域名的时候需要设置
            pathRewrite: { // 路径重写，去掉/api
                "^/api": ""
            }
        }
    }  
}
```

**CORS（跨域资源共享）**

```
> 服务端配置响应头：
  Accept-Control-Allow-Origin: */http...
  Accept-Control-Allow-Credentials: true（跨域后是否允许携带cookie）
   	 
> 简单请求
  请求方法属于：HEAD、GET、POST
  请求头只包含安全字段如：Accept、Accept-Language、Content-Length、Content-Type...
  请求头如果包含Content-Type，其值只限于：
    application/x-www-form-unlencoded（普通form表单提交）
   	multipart/form-data（发送文件）
   	text/plain
   
> 复杂请求
  其他为复杂请求，浏览器会先发送预检请求OPTIONS，询问服务器是否允许访问
  预检请求头：
    Accept-Control-Request-Method（CORS请求的方法）
    Accept-Control-Request-Headers（CORS请求携带的额外头信息字段）
  预检响应头：
    Accept-Control-Allow-Methods（服务器所支持的CORS请求的方法）
    Accept-Control-Allow-Headers（服务器所支持的CORS请求的所有头信息字段）
    Access-Control-Max-Age（告诉浏览器在一定时间段内，对于同样的源、方法、请求头都不需要发送OPTIONS）
```

**Nginx反向代理**

```nginx
server {
    listen	80; 
    server_name	'test.com'; #服务器域名
    location / {
        root html;
        index index.html index.htm;
        proxy_pass http://localhost:8080; #前端地址
    }
}
```

#### 3. 服务器返回请求结果

##### 3.1 解析请求结果

网络进程解析返回的响应头后，如果发现状态码是301或302，说明需要重定向，重定向地址为Location字段，然后再发起新的HTTP请求。检查响应数据的类型**Content-Type**，如**text/html**说明返回的数据是HTML格式，然后将数据发送给浏览器进程。

| 状态码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 请求成功，有响应体                                           |
| 204    | 请求成功，没有响应体                                         |
| 301    | 永久重定向，不影响旧域名，可以将SEO权重慢慢转移到新域名      |
| 302    | 临时重定向，用于网页短时间内进行改版，在不影响用户体验的情况下跳转到临时页面 |
| 304    | 协商缓存命中                                                 |
| 400    | 请求错误，服务器无法处理                                     |
| 401    | 用户未经授权，需要验证                                       |
| 403    | 没权限访问                                                   |
| 404    | 请求的资源不存在                                             |
| 500    | 服务器内部错误                                               |

##### 3.2 TCP四次挥手

客户端第一次发送FIN表示不会再发送数据给服务端，但是还具有接受能力，服务端接收到FIN后只能先回复ACK，因为它可能还有数据要发送给客户端，然后不再发送数据时再发送FIN，客户端再回复ACK确认关闭

```
1. 客户端打算关闭连接，会发送FIN报文给服务端，并进入FIN_WAIT_1状态
2. 服务端收到FIN报文后，回复ACK报文给客户端，并进入CLOSE_WAIT状态，客户端收到ACK报文后，进入FIN_WAIT_2状态
3. 当服务端确认已经没有数据要返回给客户端时，会发送FIN报文给客户端，并进入LAST_ACK状态
4. 客户端收到FIN报文后，回复ACK报文给服务端，并进入TIME_WAIT状态，服务端收到ACK报文后进入CLOSE状态，客户端在TIME_WAIT状态等待2MSL（最大报文段生存时间）后也进入CLOSE状态
```

```
> 为什么客户端先进入TIME_WAIT状态？
  服务端在第三次挥手后如果没有收到客户端的ACK报文，会再次向客户端发起第三次挥手重新发送FIN报文，客户端进入TIME_WAIT状态能正常接收到新的FIN报文，并再次向服务端发送ACK报文。
> 为什么客户端要等待2MSL后才关闭？
  服务端在第三次挥手后如果没有收到客户端的ACK报文，会再次向客户端发起第三次挥手重新发送FIN报文，所以考虑最坏的情况就是：第四次挥手的ACK报文最大生存时间 + 第三次挥手的FIN报文最大生存时间。
```

#### 4. 准备渲染进程

默认每个标签页会对应一个渲染进程，如果两个标签页位于同一个浏览上下文组，且属于同一个站点，则共用同一个渲染进程。

**浏览上下文组**：通过JS脚本关联起来的标签页

**浏览上下文**：每个标签页的内容

```html
<a href="https://www.baidu.com" target="_blank">百度</a>
// 新标签页中可以通过window.opener拿到原来标签页的window
```

```js
const newWindow = window.open("https://www.baidu.com")
// 新标签页中可以通过window.opener拿到原来标签页的window
// 原来的标签页中可以通过newWindow拿到新标签页的window
```

注：当a标签设置`target="_blank"`时，由于新标签页能拿到旧标签页的window，因此可能会受到黑客攻击。

```html
<a href="https://www.baidu.com" target="_blank" ref="noopener onreferrer">百度</a>
// ref="noopener" 新标签页中无法拿到window.opener（旧浏览器不兼容）
// ref="noreferrer" 新标签页中无法拿到window.opener和document.referrer（原来标签页的url地址）
```

#### 5. 渲染阶段

浏览器进程将从网络进程接收到的HTML数据发送给渲染进程后，渲染进程便开始解析页面、加载资源

##### 5.1 构建DOM树

输入的HTML文件通过HTML解析器处理后，生成树结构的DOM

##### 5.2 构建CSS规则树

##### 5.3 构建render树

结合DOM树和CSS规则树生成render树，`dislay:none`的元素不会出现在render树中

##### 5.4 布局

计算各个节点的尺寸、位置等信息

##### 5.5 分层

为特定的节点生成新的图层，并生成图层树

```
> 拥有层叠上下文属性的节点
  定位属性position、透明度opacity、CSS滤镜filter、z轴排序z-index
> 需要被裁剪的节点
  如元素设置了超出隐藏overflow，则可见区域的内容和滚动条都会单独生成新的图层
```

满足特定条件的图层会被浏览器提升为合成层，合成层的位图由GPU进程生成，合成层内容变化不会影响其他图层，但是如果合成层数量过多，会有性能要求（涉及到进程间的通信）

```
> 3D变化如translate3d、translateZ等
> video、canvas、iframe等元素
> position: fixed
> will-change属性
```

##### 5.6 绘制

为图层生成记录其绘制顺序和绘制指令的列表，将其发送给合成线程

##### 5.7 栅格化

合成线程将图层划分为一个个图块，然后将可视区内的图块通过栅格化线程生成位图（合成层会利用GPU进程加速栅格化），然后发送绘制命令给浏览器主进程

##### 5.8 合成和显示

浏览器进程进行页面绘制，显示到屏幕上

```
重排
> 添加或删除可见的DOM元素
> 元素尺寸改变
  margin、padding、border、width、height
> 字体大小改变
> 内容变化，如input输入
> 浏览器窗口大小改变
> 查询元素几何信息
  clientHeight、offsetHeight、scrollHeight、getBoundingClientRect()

重绘
> color
> border-style
> visibility
> background

避免重排、重绘
1. 避免使用display:table布局
2. 避免频繁操作style，可通过class类名操作样式
3. 尽量使用transform代替动画里的位置变化，使用opacity代替visibility
4. 使用display:none模拟元素添加删除
5. 避免重复读取元素的几何信息，可以用变量暂存
6. 开启GPU加速
```




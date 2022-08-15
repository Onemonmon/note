### 1. DNS域名解析

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
DNS是域名系统，用于将域名转换成IP地址
分布式：整个DNS由许多DNS服务器组成，每个DNS服务器保存各自域名与IP的映射关系
分层：DNS服务器分为三个层级，根DNS服务器(.)、顶级域DNS服务器(.cn/com/org/...)、权威DNS服务器(.baidu/...)，从上往下查找，最终在权威DNS服务器上找到映射关系
```

```
DNS缓存：浏览器 => 操作系统 => 本地Hosts文件 => ISP的本地DNS服务器 => 根DNS服务器
```

```
每个ISP会有自己的本地DNS服务器，当客户端发起DNS请求时，会先发往本地DNS服务器，它起着代理作用，将该请求转发到DNS的各层级结构中。 
迭代查询：本地DNS服务器到DNS各层级服务器间的查询过程
递归查询：DNS各层级服务器间的查询过程
```

### 2. TCP三次握手

```
标记位：
> SYN = 1，希望创建连接
> ACK = 1, 确认号有效
> FIN = 1, 希望断开连接
> RST = 1, TCP连接异常，需要断开连接
序列号seq：解决数据包乱序问题
确认号ack：表示上一个数据包已成功接收
```

```
1. 客户端发送包（SYN=1，seq=x）给服务器，客户端进入SYN_SEND状态
2. 服务端收到客户端的包，并向客户端发送包（SYN=1，ACK=1，seq=y，ack=x+1），服务端进入SYN_RCVD状态
3. 客户端收到包后再发送包（ACK=1，ack=y+1），发送后客户端进入ESTABLISHED状态，服务端收到后也进入ESTABLISHED状态
```

```
三次握手是确认双方是否都有发送和接收能力的最小次数
```

### 3. SSL/TLS握手

```
如果使用https，则需要进行SSL/TLS握手，使服务端与客户端建立安全的通信连接
1. 浏览器生成client random随机字符串，将浏览器的加密算法列表和client random发送给服务器
2. 服务器生成server random随机字符串，将所选择的加密算法、server random、数字证书等信息发送给浏览器
3. 浏览器从数字证书中验证服务器的合法性并获取公钥，并生成一个新的key，通过公钥对key进行加密后发送给服务器
4. 服务器接收到加密的key后，使用私钥进行解密，至此双方都拿到了同一个key
5. 双方使用之前选择的加密算法、client random、server random、key，生成密钥进行对称加密
```

### 4. 发送网络请求

#### 4.1 HTTP/HTTPS

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

#### 4.2 浏览器缓存

```
缓存位置：
from memory cache：缓存在内存中，速度快，容量小，关闭页面后会消失
form disk cache：缓存在硬盘中，速度较慢，容量大，时效性长，不会随着页面关闭而消失
```

```
强制缓存
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

```
协商缓存
强制缓存失效后进入协商缓存，浏览器携带缓存标识发起HTTP请求，资源更新时返回200，并缓存新资源和缓存标识，资源未更新时返回304，使用缓存的资源
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

```
ctrl+f5 强制刷新，跳过强制缓存和协商缓存
f5 跳过强制缓存，但会检查协商缓存
```

#### 4.3 cookie、localStorage、sessionStorage

```
> cookie可设置失效时间，未设置时默认关闭浏览器后清除
  localStorage只能代码或手动清除
  sessionStorage在关闭tab页或浏览器时清除
> cookie可存放4KB
  localStorage、sessionStorage可存放5MB
> cookie默认会在同域的http请求头中携带
  localStorage、sessionStorage不参与和服务器的通信
```

#### 4.4 请求类型

```
GET查，POST改，PUT增，DELETE删
GET和POST从约定上看，GET请求参数放在url上，POST请求参数放在body上
GET和POST从规范上看，GET一般用于查询操作，是幂等的，POST一般用于增删改操作，是不幂等的
GET的参数长度受浏览器URL长度的限制
GET会被浏览器主动缓存
```

#### 4.5 处理跨域

```
1. 通过CORS（跨域资源共享）处理
   服务端配置响应头：
   	 Accept-Control-Allow-Origin
   	 Accept-Control-Allow-Credentials（是否允许发送cookie）
   	 
   简单请求：
   > 请求方法只限：HEAD、GET、POST
   > 请求头字段只限：
   		Accept
   		Accept-Language
   		Content-Length
   		Content-Type: 
   			application/x-www-form-unlencoded（普通form表单提交）
   			multipart/form-data（发送文件）
   			text/plain
   
   其他为复杂请求，处理复杂请求的CORS时，会先发送预检请求OPTIONS
   预检请求头：Accept-Control-Request-Method（CORS请求的方法）、Accept-Control-Request-Headers（CORS请求携带的额外头信息字段）
   预检响应头：Accept-Control-Allow-Methods（服务器所支持的CORS请求的方法）、Accept-Control-Allow-Headers（服务器所支持的CORS请求的所有头信息字段）
   
2. 通过Nginx反向代理
```

```
跨域请求如何携带cookie？
   配置请求头withCredentials: true
   配置响应头Access-Control-Allow-Origin: */http...
   配置响应头Access-Control-Allow-Credentials: true
```

### 5. 服务器返回请求结果

### 6. TCP四次挥手

```
1. 客户端打算关闭连接，会发送FIN报文给服务端，并进入FIN_WAIT_1状态
2. 服务端收到FIN报文后，回复ACK报文给客户端，并进入CLOSE_WAIT状态，客户端收到ACK报文后，进入FIN_WAIT_2状态
3. 当服务端确认已经没有数据要返回给客户端时，会发送FIN报文给客户端，并进入LAST_ACK状态
4. 客户端收到FIN报文后，回复ACK报文给服务端，并进入TIME_WAIT状态，服务端收到ACK报文后进入CLOSE状态，客户端在TIME_WAIT状态等待2MSL后也进入CLOSE状态
```

```
客户端第一次发送FIN表示不会再发送数据给服务端，但是还具有接受能力，服务端接收到FIN后只能先回复ACK，因为它可能还有数据要发送给客户端，然后不再发送数据时再发送FIN，客户端再回复ACK确认关闭
```

```
MSL：最大报文段生存时间
进入TIME_WAIT状态是为了防止服务端没有收到客户端的ACK报文，从而再次向客户端发起第三次挥手时，客户端能正常接收到FIN报文并向服务端继续发送ACK报文
等待2MSL是因为考虑服务端没有客户端发送的ACK报文时会重新发送FIN报文，而考虑最坏的情况就是：第四次挥手的ACK报文最大生存时间+第三次挥手的FIN报文最大生存时间
```

### 7. 浏览器解析HTML并渲染

```
1. 构建DOM树
2. 构建CSS规则树
3. 构建render树，dislay:none的元素不会出现在render树中
4. 布局，计算各个节点在页面的大小、位置等
5. 绘制

> 浏览器的GUI渲染线程与JS引擎线程是互斥的
> JS的加载、执行会阻塞DOM树的构建
> CSS不会阻塞DOM树的构建，但是会阻塞页面的渲染
> JS的执行可能会修改CSS，所以需要等待CSS规则树构建完成，因此如果CSS还在请求中，后面的JS代码会被阻塞不能立即执行，进而阻塞了DOM树的构建
```

```
script没有defer、async时，会立即加载并执行JS脚本
defer、async在加载JS时不会阻塞HTML解析，async在加载完之后执行，defer在HTML解析完成后执行
脚本与DOM/其他脚本的依赖关系强 => defer
脚本与DOM/其他脚本的依赖关系弱 => async
```

```
回流：
> 添加或删除可见的DOM元素
> 元素尺寸改变
  margin、padding、border、width、height
> 字体大小改变
> 内容变化，如input输入
> 浏览器窗口大小改变
> 查询元素几何信息
  clientHeight、offsetHeight、scrollHeight、getBoundingClientRect()
重绘：
> color、border-style、visibility、background
```

```
避免回流、重绘
1. 避免使用display:table布局
2. 避免分次改变css属性
3. 尽量使用transform代替动画里的位置变化，使用opacity代替visibility
4. 使用display:none模拟元素添加删除
5. 避免重复读取元素的几何信息，可以用变量暂存
```


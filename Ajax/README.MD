### 第七章 Ajax

本章从服务器和客户端数据传输的角度出发，考虑如何改善用户和网站的交互体验

#### 1. 数据传输

- XMLHttpRequest(XHR)
- Dynamic script tag insertion 动态脚本注入
- iframes
- Comet
- Multipart XHR

##### XMLHttpRequest

具体参考 [脚本化http](https://github.com/xswei/HTTP_Scripting)

##### 动态脚本注入

动态脚本注入的基本思想是使用脚本创建一个新的script标签，并制定src为需要请求的URL。动态脚本注入与XHR最大的优点是能跨域请求数据，不受同源策略的限制。响应正文必须是可执行的javascript代码，因此要使用eval函数执行响应数据。由于响应消息是直接执行的，因此解析速度比XHR快。

XHR | JSONP
--- | ---
可以使用多种请求方法 | 只能使用GET
响应可以是任何格式 | 响应只能是可执行的js代码
可以设置头信息 | 不能设置头信息

##### Multipart XHR

Multipart XHR(MHR)允许客户端用一个HTTP请求就可以从服务端向客户端发送多个资源，通过在服务端将资源打包成一个双方约定好的字符串分割的长字符并发送到客户端，然后再客户端用JavaScript解析，并根据mime-type类型解析出资源。

比如将多个图片转为base64编码。


#### 2. 发送数据

##### XHR

XHR可以使用GET和POST发送数据，但是GET受URL长度的限制(2048)，因此如果发送的数据比较大时，使用POST发送数据。此外GET发送数据时，只发送一个数据包，而POST发送数据时至少发送连个数据包，一个包含头信息，一个包含POST正文。

##### Beacons(图片信标)

这种方式类似JSONP

首先使用JavaScript动态创建一个img标签，并把src属性设置为要发送数据的URL，并把要发送的数据加入到URL的末端。但是这个img标签并不被插入到文档中。这种方式简单，但是无法发送POST数据而且长度有限制。

那么如何知道数据发送成功？可以监听这个对象的load事件，也可以双方约定好图片的大小来表示信息是否正确发送，比如服务器返回一个宽度为1的图片表示成功，宽度为2表示重试等等。

#### 3. 数据格式

在使用数据传输时，常用的文本格式有以下几种

- XML
- JSON
- HTML

##### XML

在Ajax刚开始流行时，XML有很强的优势，比如通用性，格式严格且易于验证等等。但是XML极其冗余，每个数据都会附带大量的标签结构，有效数据比例低。

此外，解析XML也非常麻烦，虽然后来提出了XPath，但是并没有被广泛使用，直到后来出现了JSON。

##### JSON

JSON是一种使用JavaScript对象和数组直接编写的轻量级数据格式。JSON是一段可执行的JavaScript代码，在JavaScript中可以使用JSON.parse()和eval来解析JSON字符串。但是为了安全起见，不推荐使用eval(),因为里面可能包含恶意代码。而JSON.parse()可以检测出JSON字符串中的此法错误。

数组形式的JSON解析速度要比对象形式的快。

##### HTML

Javascript可以很快的将一个较大的数据结构转为HTML，但是在服务器端会更快。因此某些情况下可以在服务器端构建好HTML再发送给客户端，然后使用JavaScript的innerHTML属性把它插入到相应的位置。

在服务器端构建好HTML的的弊端在于，HTML是一种臃肿的数据格式，无疑会加大网络传输压力。而且插入到页面中时会花费大量时间把数据载入页面。


#### 4. Ajax性能指南

##### 缓存数据

如果可以把数据缓存到本地，减少重复请求的次数，会提高性能。

- 在服务器端，设置HTTP头信息以确保响应会被浏览器缓存
- 在客户端，把获取到的信息缓存下来，避免再次请求









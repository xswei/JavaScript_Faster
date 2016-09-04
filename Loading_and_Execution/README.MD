### 第一章 加载和执行

多数浏览器使用单一进程来处理用户界面(UI)和JavaScript，所以同一时刻只能做一件事，JavaScript执行的时间越长，浏览器等待响应的时间就越久。当浏览器遇到script标签时，会停止处理页面，先下载(src)执行JavaScript代码，然后再解析渲染页面。

#### 1. 脚本位置

script标签可以放在head和body中，理论上来说，当script和link放在head中并先加载有助于确保页面渲染和交互的正确性。但是这样做有个严重的问题，head中的脚本阻塞页面渲染，body中的元素会等脚本执行完之后才会显示，会导致明显的延迟。现代浏览器允许并行下载JavaScript文件，但是这个下载过程仍然会阻塞其他资源的下载，比如图片等。

为了避免因脚本阻塞带来的延迟，可采取以下措施：

- 将所有的script标签放在页面底部
- 使用延迟脚本，即defer属性，直到文档载入并解析完成才会执行
- 使用异步脚本，即async属性，浏览器尽快执行脚本，但不阻塞文档解析

#### 2. 组织脚本

减少页面包含的script标签数量也能减少脚本的下载时间。通常如果网站依赖多个JavaScript脚本时，可以将这些脚本合并成一个以减少性能消耗。

文件的合并通过离线打包工具或实时在线服务实现，或使用CDN加速。

#### 3. 动态脚本元素和XMLHttpRequest脚本注入

script元素与页面内其他元素都能通过DOM进行引用，那么当然也可以通过JavaScript创建一个script元素。

```
var script = document.createElement("script");
script.type = "text/javascript";
script.src="file.js"
doument.getElementsByTagName("head")[0].appendChild(script);
```
新创建的元素在被添加到页面时开始下载，返回的代码立即执行，但是这个技术的重点在于：无论何时启用下载，文件的下载和执行都不会阻塞页面其他进程。

另一种无阻塞加载脚本的方式是通过XMLHttpRequest(XHR)对象获取脚本并注入到页面中：先创建一个XHR对象，然后用它下载JavaScript文件，然后通过创建动态script元素将代码注入到页面中。

```
var xhr = new XMLHttpResquest();
xhr.open("get","file.js",true);
xhr.onreadystatechange = function(){
	if(xhr.readyState == 4){
		if(xhr.status>=200 && xhr.status<300 || xhr.status == 304){
			var script = document.createElement("script");
			script.type = "text/javascript";
			script.text = xhr.responseText;
			doument.body.appendChild(script);
		}
	}
}
xhr.send(null);
```
使用XMLHttpRequest的主要局限是受同源策略的影响，只能从相同的源获取脚本内容。

#### 4. 使用工具

- YUI Compressor压缩代码
- LazyLoad类库，延迟加载工具
- LABjs，无阻塞脚本加载工具，有管理依赖的功能

#### 总结

- 将script标签放在页面底部，body闭合之前
- 合并脚本
- 使用无阻塞加载方式：defer，async属性，动态创建script标签，XHR对象注入JavaScript代码

### 快速响应的用户界面


#### 1. 浏览器UI线程

在大多数浏览器中,使用单线程进行JavaScript执行和更新用户界面,这个线程被称为"浏览器UI线程",也就是说当执行JavaScript时,用户界面无法响应用户的输入.UI线程的工作基于队列系统,需要进行的任务会被保存到队列中直到空闲才会执行.

因为浏览器的UI线程在某一时刻执行JavaScript代码或进行界面更新,所以当JavaScript代码运行时间过长时,就会造成页面假死的状态.所以大多数浏览器都对JavaScript代码的执行进行了限制,限制的方式有以下两种：

- 调用栈大小限制
- 运行时间限制

其中运行时间可以从两个方面进行衡量：代码语句数量或代码总执行时间.

当代码超出调用栈限制后会抛出错误,运行超时时会弹出对话框.因此在开发时,要避免长时间运行脚本.那么多长算长？

当代码运行超出5秒或10秒(不同浏览器厂商设置不一样),页面崩溃,这是最糟糕的情况.即使运行时间没有这么长,也会影响用户体验,如果不想让JavaScript脚本的运行时间影响到用户体验,那么脚本的运行最好不要超过100ms(某研究者的结论).

如果脚本在100ms之内不能完成怎么办？下面有几种方案.

#### 2. 使用定时器让出时间片段

使用定时器让出时间片段,意思就是将代码总的运行时间分割成多个时间片段,比如某个任务需要500ms,则可以通过定时器将代码分割成10个50ms的片段.每个50ms之间空出机会让UI界面进行更新,以提高用户体验.具体的思路如下：

假设需要对一个数组(items)进行循环,并对其中的每个元素执行响应操作(process)：

```
for(var i=0,len=items.length;i<len;++i){
	process(item[i]);
}
```

如果使用上述写法,那这个数组循环根本停不下来,也就是说不能对其运行时间进行控制.下面的方式使用了定时器对数组操作进行了分片,每25ms操作一个元素,这样UI界面就有机会在某个元素操作完成后进行更新.

```
var todo = items.concat();	//克隆一个副本
setTimeout(function(){
	process(todo.shift());	//对副本中的每个元素进行操作
	if(todo.length>0){
		setTimeout(arguments.callee,25);	//如果没有遍历完成,则再次添加一个计时器
	}else{
		callback(items);	//遍历完成后调用回调
	}
},25)
```
封装一下,如下：

```
function processArray(items,process,callback){
	var todo = items.concat();
	setTimeout(function(){
		process(todo.shift());
		if(todo.length>0){
			setTimeout(arguments.callee,25);
		}else{
			callback(items);
		}
	},25}
}
```

上述方法每次只能操作一个元素,但是这样效率高吗？我也不知道,直到我看到了一个更高明的做法：记录代码运行时间,在指定时间内,能操作几个算几个.

```
function timdProcessArray(items,process,callback){
	var todo = items.concat();
	setTimeout(function(){
		var start = +new Date();
		do{
			process(item.shift());
		}while(todo.length>0 && (+new Date() - start<50));	//记录时间,运行时间不超过50ms
		if(toto.length>0){
			setTimeout(arguments.callee,25);	//25ms之后再执行50ms
		}else{
			callback(items);
		}
	},25)
}
```

使用计时器分割代码确实是一种提高用户体验的方法,这都是单线程惹的祸,要是在多个线程就好了,于是Web Worker来了.


#### 3. Web Workers

Web Workers是HTML5提供的JavaScript多线程解决方案,用来进行大量计算而不会冻结用户界面.每个worker都在自己独立的线程中执行,不会影响浏览器UI线程和其他的worker线程.

##### worker运行环境

Web Workers没有和浏览器UI进行绑定,也就意味着不能访问浏览器的许多资源,JavaScript和UI共享同一个进程主要是因为他们之间频繁的相互访问.而worker有自己独立的运行环境,其功能只是JavaScript的一个子集.worker运行环境由以下部分组成：

- navigator对象,只包括appName,appVersion,userAgent和platform属性
- location对象,与window.location相同,只不过是只读的
- self对象,全局worker对象
- importScripts()方法,用来加载外部JavaScript文件
- 所有的ECMAScript对象,Date,Array,Object等
- XMLHttpRequest构造器
- setTimeout()和setInterval()方法
- close()方法,用来停止worker运行

定义一个worker：

```
var worker = new Worker("workerCode.js")
```

##### 页面JavaScript和worker通信

Worker与网页之间通过事件接口进行通信,网页代码通过postMessage()方法给Worker传数据,Worker通过onmessage事件处理器接受信息.


网页脚本：

```
worker.onmessage=function(event){	//接收新线程传来的数据
	alert(event.data);	//"hi"
}
worker.postMessage("hello world");	//给新线程传数据

```

worker：

```
self.onmessage = function(event){
	//event.data:"hello world"
}
self.postMessage("hi");

```

网页脚本和Worker之间只能传递特定类型的数据,比如原始值(string,number,boolean,null,undefined),也可以传递Object和Array类型.在内部是通过序列化和反序列化实现的,有效的数据会先被序列化,传递之后再被反序列化.

##### 加载外部文件

在Worker部分代码中也可以引入外部文件,通过importScripts()方法引入.该方法可以接受多个参数,指定多个脚本.并且调用过程是阻塞式的,当所有文件加载并执行完成后才会继续执行,这种阻塞是发生在外部线程的,并不会影响UI界面的更新.

##### 实际用途

Web Workers适用于处理纯数据,或者会与浏览器UI无关的长时间脚本.

- 编码/解码大字符串
- 复杂数学运算(图像视频处理)
- 大数组排序

#### 小结

- 任何JavaScript代码执行时间不要超过100ms,如果超过,使用分片或Web Workers
- JavaScript运行期间,浏览器对用户的交互存在差异,有的会忽略用户操作,有的会将操作加入任务队列
- Web Workers是个好东西

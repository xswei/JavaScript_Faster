### 第二章 数据存取

数据存储位置关系到代码执行过程中数据的检索速度

#### 1. JavaScript中的数据存储

JavaScript中有四种基本的数据存储：

- 字面量，字面量代表自身，不存储在特定位置，比如字符串，数字，布尔值，数组，函数，null，undefined等
- 本地变量，使用var定义的数据存储单元
- 数组存储
- 对象成员

在大多数情况下，字面量和本地变量的读写速度相差不多，都快于数组项和对象成员的访问。

#### 2. 作用域

##### 作用域的工作原理

JavaScript中每个函数都是一个对象(Function的一个实例)。与其他对象一样，函数有两种属性：

- 可以通过编程访问的属性
- 不能通过代码访问，仅供JS引擎存取的内部属性，其中一个是[[Scope]]

##### scope属性

内部属性scope包含了一个函数作用域中的对象的集合，这个集合被称为函数作用域链，它决定了哪些属性能被函数访问。函数作用域中的每个对象称为可变对象，每个可变对象都以键值对的形式存在。当函数被创建后，作用域链会创建一个当前作用域可以访问的数据对象。

```
function Add(num1,num2){
	var sum = num1 + num2;
	return sum;
}
```

![image](https://github.com/xswei/JavaScript_Faster/blob/master/Data_Access/images/add.png)

当函数执行时，会创建一个执行环境的内部对象，执行环境定义了函数运行时的环境。函数每次执行时对应的执行环境都是独一无二的，所以多次调用同一个函数，会创建个执行环境，当函数执行完毕，执行环境就被销毁。

每个执行环境都有自己的作用域链，用以解析标识符，当执行环境被创建时，会创建一个被称为"活动对象"的新对象，活动对象中包含了所有的局部变量，命名参数，参数集合以及this。对象会被插入到作用域链最前端，以确保优先搜索。之后才是函数外的其他可访问对象。


![image](https://github.com/xswei/JavaScript_Faster/blob/master/Data_Access/images/add_run.png)

在函数执行过程中每遇到一个变量，就会搜索一次作用域链，先是活动对象(局部变量),然后跨作用域域到作用域链中的下一个对象，直到最后的全局对象。这样就出现了标识符所在对象的"深浅"问题，局部变量在作用域链最前端，最浅，最快被搜索到，而全局变量所在对象在最后，搜索时间也最长，因此标识符的搜索过程会影响性能。

所以在函数中，尽可能使用局部变量，以提高变量的搜索性能。

##### with和try-catch

with和try-catch可以在函数运行时临时改变作用域链。

with语句会给对象的所有属性创建一个变量，当执行到with时，with所包含的对象会被插入到作用域链的最前端，而原本在最前端的局部变量对象则会被移动到第二个对象中，这样会增加局部变量的搜索性能。

```
function initUI(){
	with(document){
		var bd,links,i,len;
		//do something
	}
}
```

![image](https://github.com/xswei/JavaScript_Faster/blob/master/Data_Access/images/with.png)


try-catch的catch语句与with类似，也会将catch到的对象放在作用域链最顶端，而catch内部的变量会被移动到下一个对象中。此时可以将错误委托给另一个函数来处理。

##### 闭包

闭包是JavaScript最强大的特性之一，它允许函数访问局部作用域之外的数据。

```
function assignEvent(){
	var id = "xdi9592";
	docment.getElementById("save-btn").onclick = function(event){
		saveDocument(id);
	}
}
```
在上述示例中，assignEvent()函数执行时，包含了变量id的活动对象被创建，这个活动对象是执行环境作用域链中的第一个对象，第二个是全局对象。当闭包创建时，闭包的scope被初始化为这些对象，也就是闭包作用域链中的第一个对象为assignEvent作用域链中的第一个对象，第二个为全局对象。

![image](https://github.com/xswei/JavaScript_Faster/blob/master/Data_Access/images/assignEvent_run.png)


那么问题来了，assignEvent执行完后活动对象随着执行环境一起呗销毁，而包含id的这个互动对象仍然被闭包引用。造成这个对象不能被垃圾回收机制回收，这就造成了额外的内存开销。因此闭包与非闭包相比，会占用更多的内存。

当闭包代码执行时，也会创建一个执行环境，它的作用域链与属性scope中引用的两个相同的作用域链对象被一起初始化，然后活动对象为闭包自身所创建。这样在闭包运行完毕后，作用域链所有的对象都会被销毁，避免造成内存泄漏。

![image](https://github.com/xswei/JavaScript_Faster/blob/master/Data_Access/images/bibao_run.png)

#### 3. 对象成员

JavaScript中的对象是基于原型的，对象中有两种成员类型，一种是自有成员，一种是原型成员。可以通过hasOwnProperty()方法判断。

#### 原型链

对象的原型决定了对象的类型。默认情况下，对象都是Object的实例，并继承所有基本方法。但是你可以自定义构造函数来创建另一种类型的原型。

```
function Book(title){
	this.title = title;
}
Book.prototype.sayTitle = function(){
	alert(this.title);
}
var book1 = new Book("Python");
var book2 = new Book("C++");

alert(book1 instanceof Book)	//true
alert(book2 instanceof Book)	//true

book1.sayTitle() 	//Python
alert(book1.toString()) 	//[object object]

```

实例book1的原型是Book.prototype,而Book.prototype的原型是Object，原型之间的关系如下图。

![image](https://github.com/xswei/JavaScript_Faster/blob/master/Data_Access/images/prototype.png)


与作用域链类似，属性如果不是自有属性而是原型链上的，则这个属性越深，搜索性能越差。

#### 总结

- 访问字面量和局部变量的速度最快，数组元素和对象元素速度相对较慢
- 局部变量存在于作用域链的最顶端，因此访问速度最快。变量在作用域链中的位置越深，访问所需时间越长，而全局变量位于作用域链最末端，因此访问速度最慢
- 避免使用with语句，因为它会改变作用域链
- 嵌套对象成员会影响性能
- 属性和方法在原型链中的位置越深，访问速度越慢
- 把常用的对象成员、数组元素、跨域变量保持成局部变量，可以改善JavaScript性能

### 第三章 DOM编程

#### 1. 浏览器中的DOM

文档对象模型(DOM)是一个独立于语言的，用以操作XML和HTML文档的程序接口(API)。在浏览器中的接口是用JavaScript实现的，浏览器通常会把DOM和JavaScript独立实现，这样就可以允许其他技术比如VBScript能共享DOM。

DOM和JavaScript是两个独立的部分，只要通过接口连接，就会产生性能消耗，并且连接次数越多，消耗越大。

#### 2. DOM访问与修改

##### HTML集合

访问DOM元素会产生性能消耗，而修改DOM元素的代价更大--因为会导致浏览器重新计算页面的几何变化并重新渲染。

innerHTML与DOM方法，操作更新一大段HTML时，非标准但支持良好的innerHTML方法在绝大多数浏览器中要快于DOM方法，而对大多数日常操作来说没有明显的区别。

HTML集合是包含了DOM节点引用的类数组对象，可以通过以下方法或属性获取：

- document.getElementsByName()
- document.getElementsByClassName()
- document.getElementsByTagName()
- document.images
- document.links
- document.forms

HTML集合并不是真正的数组，因为没有push和slice等方法，但是有length属性并且可以通过索引获取指定DOM节点。HTML集合与文档保持连接，当文档改变时，HTML集合也会自动更新(假定实时态)，所以每次获取HTML集合信息(比如获取length属性)时都会重新执行查询过程，导致性能低下。与真正的数组相比，HTML集合的遍历速度非常低。

优化方法：
- 涉及到循环时，尽量使用局部变量，避免使用document重复查询元素
- 在循环之前，先将要操作的HTML集合缓存下来(缓存集合，每次操作都要根据索引从HTML集合中获取要操作的DOM)
- 循环之前，将要操作的DOM缓存下来(缓存DOM，性能最好，不用每次根据索引获取指定的DOM元素)

##### 遍历DOM

当获取某个元素周围的元素时候，可以使用childNodes或nextSibing，在IE6和IE7中，使用nextSibing要比childNodes快。

childNodes并不区分元素节点和属性节点，而在大部分现代浏览器中，childNodes只返回元素节点。

选择器API，根据CSS选择器查询特定元素，比如document.querySelector()和querySelectorAll()，这两个方法都是DOM节点的反复，因此支持子查询。返回NodeList(包含匹配节点的类数组对象)，NodeList对象是一个静态列表，不会对应实时的文档结构。

HTML集合 | NodeList
---|---
实时对应文档结构 | 不实时对应文档结构

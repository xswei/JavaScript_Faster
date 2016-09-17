### 编程实践

#### 1. 避免双重求值

JavaScript允许在脚本中动态的执行一个包含代码的字符串，有以下4种标准的方法可以实现：

- eval()
- Function()
- setTimeout()
- setInterval()

```
var num1 = 5,
	num2 = 6;

result = eval("sum = num1+num2");

sum = new Function("arg1","arg2","return arg1+arg2");

setTimeout("sum=num1+num2",100);

setInterval("sum=num1+num2",100);
```

当再JavaScript代码中执行另一端JavaScript代码时会导致双重求值的性能消耗，这段代码首先会以政策的方式求值，然后在执行过程中对包含字符串的代码再进行另一次求值运算。

```
var item = array[0];

var item = eval("array[0]");	//慢
```

上述示例中使用eval时会比直接求值慢，因为每次调用eval都会创建一个新的解释器/编译器实例，同样的过程也会发生在Function,setTimeout,setInterval中。

#### 2. 使用Object/Array直接量

在JavaScript中创建对象和数组的方式有很多种，但是使用直接量是最快的方式。

#### 3. 使用位操作和原生方法


##### 位运算

JavaScript中数字都按照64位格式存储，在位操作中，数字被转为有符号32位格式，直接操作该32位尽管需要转换但是比其他的JavaScript数学运算和布尔操作要快很多。

JavaScript中的位操作：

- 按位与(AND)
- 按位或(OR)
- 按位异或(XOR)
- 按位取反(NOT)

##### 原生方法

无论如何优化，JavaScript的原生方法都是最快的，因为这些原生方法被事先写入浏览器中，并且使用低级的语言实现。

比如当需要进行数学运算时，应当优先使用Math对象的原生方法。

#### 小结

- 避免使用eval,Function,setTimeout,setInterval等，这些会进行双重求值
- 尽量使用直接量
- 避免重复工作
- 考虑二进制位运算
- 优先考虑原生方法
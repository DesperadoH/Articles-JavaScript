# 应用场景等

## First

```

function createClosure () {
  var n = 1;
  return {
    set: function (x) { n = x; },
    get: function () { return n; }
  };
}

var calc = createClosure()
// => calc是一个对象，这个对象有get和set两个方法
// => javascript运行时内部会开辟一块内存，内存里存放n的值，并让这块内存对calc可见
// => 很多人觉得理解困难是因为这个概念太抽象，“可访问内存”是一个直观的角度

calc.set(5)
// => Closure.n -> 5，注意Closure是JS内部的隐式域

calc.get()
// => 返回Closure.n，也就是5


```

```
	(function(){
			function A(a,b,c){
				var ar = [a,b,c];
				return function B(i){
					return ar[i];
				}
			}
		var b = A('Here','I','am');
			console.log(b(2));	

		})();

```


## Second

**打破作用域规则的变量就是闭包。**


先补课：

**JS的变量作用域不是花括号块级的，而是函数级的（ES5）**

> 使用var：
> 在当前函数内声明一个变量

> 使用var为变量名赋值：
> 在当前函数内声明一个变量并赋值

> 不使用var声明，直接为变量名赋值（不开启严格模式）：
> 就会查找在当前函数内的变量，如果找不到，就到父级函数找，如果找到全局还是找不到，就隐式声明一个全局变量并赋值

** JS的数据类型分为两种：值类型（原始类型）、引用类型**

> 原始值：存储在栈（stack）中的简单数据段，也就是说，它们的值直接存储在变量访问的位置。

> 引用值：存储在堆（heap）中的对象，也就是说，存储在变量处的值是一个指针（point），指向存储对象的内存处。

** JS的内存回收**

> 标记-清除算法
> 这个算法把「对象是否不再需要」简化定义为「对象是否可以获得」。
> 定期的，垃圾回收器将从根开始，找所有从根开始引用的对象，垃圾回收器将回收所有不能获得的对象。

例子：

```
var b = function(){
    var n = 12450
    var x = function(){ 
        alert(n)
    }
    x() //12450
    return x
}

var a = b()
a() //12450


```

解释：

```
var a = b();

执行了【函数b】，并把执行结果赋值给了【变量a】

【函数b】的执行结果：返回一个函数
由于函数是引用类型，所以实际上返回了指向【函数x】的地址

把【变量x（变量x是个指针）】的值赋值给【变量a】，【变量a】现在变成了指针，指向了【函数x】

由于【函数x】在【函数b】内，所以按照作用域规则，
在【函数x】内操作【变量n】，其实操作的是：【函数b】内的局部【变量n】



a();

最后执行a，得到了函数x的执行结果


```

【变量a】打破了作用域规则，引用到了【函数x】，成为了闭包。
而且【函数x】不会垃圾回收了。


## Third

**闭包**和**变量提升**是两个比较基本的问题，这两者都是JS作用域的集中体现。

前端工程师在写页面JS的时候很少会关注到编译的情况,更多的是看执行的结果，然后根据相应的报错信息修正代码。在javascript的世界里，scope的作用域概念存在于函数中，当函数运行到某行代码需要查找某个变量时，编译器会在当前函数作用域下查找该变量，如果没有找到则继续往上一层函数作用域查找，这个过程会一直持续到全局作用域，如果没有找到相应变量，则抛出报错信息。
在JS中存在这样的代码，

```

console.log(e());//打印e
function e(){
  console.log('e')
}


```

这个是很令人费解的，在执行第一行代码的时候，编译解析器并没有执行到funtion哪一行，为何还是能打印出字符呢？ 这个问题的答案在于，JS执行每个函数作用域时，会把所有的变量按照local,global,closure分类声明，同时将以var声明的变量赋值undefined，将所有以funtion func(){}形式定义的函数放在FUNCTIONS里面，function在JS中是一级对象，它们可以拥有属性和方法，无论何时都会返回值，当用new形式加载function时，返回this值，除了显式定义return,其它的情况都是返回undefined。

回到正题，当JS在碰到每个函数作用域时，会先声明完函数作用域的本地变量同时对所在函数体内引用到的外界变量创建closure。在closure里面存储的变量有个神奇的地方，它会对它内部所有代码检查是否存在引用上下文变量的情况，如果存在则将变量添加进clousure，由此也可以窥见为何递归在JS中容易出现栈溢出的情况。具体的测试，见下方代码


```
var ddd='ddd';// t的上下文变量
function e(){//t的上下文变量
  console.log('e')
}
var t= (function foo() {
    debugger//查看执行时候，变量存储形式
    var x = 1;
    var test='test';
    return function () {
        return function(){
            console.log(e)
            console.log(ddd)}
        };
}());


```

在debugger区域你可以看到，debugger区域里面的closure已经存储了e和ddd变量(图片支持的不是很好，不贴了，直接把数据结构给出)。

```
local:this
closure:ddd='ddd'
        e:function e()
global

```

好了，做到这一步，基本可以看清楚javascript函数作用域的一些scope变量存储及作用域提升的细节：闭包就是一个函数创建时候的上下文(context)绑定，而变量提升则是JS在执行前做的一些变量类型(global,closure,local,Functions)的声明工作,其实也是比较能够简单易于理解的。
在我看来，对于前端工程师来说，充分利用JS函数作用域的特性，利用local变量缓存全局变量加快代码执行速度，采用闭包优化代码的模块化，利用JS语法的灵活性规划代码，这是比较好的。









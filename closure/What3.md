# ：How do JavaScript closures work?

## If you can't explain it to a six-year-old, you really don't understand it yourself.

** 从前： **

有一位公主......

```
function princess() {
```

她生活在一个充满奇幻冒险的世界里, 她遇到了她的白马王子, 带着他骑着独角兽开始周游这个世界，与巨龙战斗，巧遇会说话的动物，还有其他一些新奇的事物。

```
var adventures = [];

    function princeCharming() { /* ... */ } //白马王子

    var unicorn = { /* ... */ },          //独角兽
        dragons = [ /* ... */ ],         //龙
        squirrel = "Hello!";            //松鼠

    adventures.push(unicorn, dragons, squirrel, ....);


```

但是她不得不回到她的王国里，面对那些年老的大臣。

```
return {
```

她会经常给那些大臣们分享她作为公主最近在外面充满奇幻的冒险经历。

```
story: function() {
            return adventures[adventures.length - 1];
        }
    };
}
```

但是在大臣们的眼里，总是认为她只是个小女孩......

```
var littleGirl = princess();
```

....讲的是一些不切实际，充满想象的故事

```
littleGirl.story();

```

即便所有大臣们知道他们眼前的小女孩是真的公主，但是他们却不会相信有巨龙或独角兽，因为他们自己从来没有见到过。大臣们只会觉得它们只存在于小女孩的想象之中。

但是我们却知道小女孩述说的是事实.......


## Second

看一个例子"


```

var foo = ( function() {
    var secret = 'secret';
    // “闭包”内的函数可以访问 secret 变量，而 secret 变量对于外部却是隐藏的
    return {
        get_secret: function () {
            // 通过定义的接口来访问 secret
            return secret;
        },
        new_secret: function ( new_secret ) {
            // 通过定义的接口来修改 secret
            secret = new_secret;
        }
    };
} () );

foo.get_secret (); // 得到 'secret'
foo.secret; // Type error，访问不能
foo.new_secret ('a new secret'); // 通过函数接口，我们访问并修改了 secret 变量
foo.get_secret (); // 得到 'a new secret'

```

> 之所以可能通过这种方式在 JavaScript 种实现公有，私有，特权变量正是因为闭包，闭包是指在 JavaScript 中，内部函数总是可以访问其所在的外部函数中声明的参数和变量，即使在其外部函数被返回（寿命终结）了之后。

需要注意的一点时，内部函数**访问的是被创建的内部变量本身**，而不是它的拷贝。所以在闭包函数内加入 loop 时要格外注意。另外当然的是，闭包特性也可以用于创建私有函数或方法。


关于为什么在 JavaScript 中闭包的应用都有关键词“return”，引用 JavaScript 秘密花园中的一段话：

> 闭包是 JavaScript 一个非常重要的特性，这意味着当前作用域总是能够访问外部作用域中的变量。 因为 函数 是 JavaScript 中唯一拥有自身作用域的结构，因此闭包的创建依赖于函数。

## Third


#### 什么是闭包？
　　闭包（Closure）这个词的意思是封闭，将外部作用域中的局部变量封闭起来的函数对象称为闭包。被封闭起来的变量与封闭它的函数对象有相同的生命周期。

#### 什么是函数对象？

　　函数对象是作为对象来使用的函数，这里的对象是指编程语言操作的数据。

#### 函数对象与闭包

　　函数对象不一定是闭包。
　　
　　C 语言中，可以获取一个函数的指针，并通过指针间接调用此函数。这就是 C 语言中的对象（函数对象也是对象）。但 C 语言中的函数对象不是闭包——它不能访问外部作用域的局部变量。
　　
　　Javascript 中，每个函数都有一个与之相关联的作用域链。每次调用 JavaScript 函数的时候，都会为之创建一个新的对象用来保存局部变量，并把这个对象添加至作用域链中。当函数返回时，再将这个对象删除，此对象会被当做垃圾回收。但如果这个函数定义了嵌套的函数，并将它存储在某处的属性里，就意味着有了一个外部引用指向这个嵌套的函数。它就不会被当作垃圾回收，它所指向的变量绑定对象同样不会被回收。
　
　　由此可见，JavaScript 中的函数对象是闭包——可以把外部作用域的局部变量“封闭”起来。

#### 什么是对象？

　　面向对象中的“对象”是指问题空间中的元素（猫、狗）及其在解空间中的表示（new Cat(); new Dog()）。对象是过程（函数）与数据的结合。

#### 对象与闭包

　　对象是在数据中以方法的形式内含了过程(函数)，闭包是在过程中以环境的形式内含了数据。所谓“闭包是穷人的对象”、“对象是穷人的闭包”，就是说使用其中的一种方式，就能实现另一种方式能够实现的功能。

#### 应用场景

　　保护函数内的变量安全：如迭代器、生成器。
　　
　　在内存中维持变量：如果缓存数据、柯里化。


## Fourth

JavaScript 中的闭包与其 Scope Chain 特性真是密不可分的.

```
def foo() {
    var a = 1;
        def bar() {
            a = a + 1;
            alert(a);
          }
  return bar;
}


var closure = foo(); // 这个时候返回的是 bar() 这个函数外加其包上的变量 a;
var closure2 = foo(); // 这里同样生成了另外一个闭包(实例)
closure(); // 2
closure2(); // 2 , 绑定了另外一份变量 a
closure(); // 3



```

对于常规的 foo() 方法来说, 在其内部的变量 a 的存在应该在 foo() 方法执行完毕以后就消失了, 但是 foo() 方法返回了一个新的方法 bar(), 而这个方法却访问到了 foo() 方法的变量 a (JavaScript 通过 Scope Chain 访问到父级属性), 而方法 bar() 的存在延长了变量 a 的存在时间, 类似与将变量 a 关闭在了自己的作用域范围内一样, 只要方法 bar() 没有失效, 那么变量 a 则会一直伴随着方法 bar() 存在, 而变量 a 与方法 bar() 的这样存在形式被称为闭包; 

** 闭包的应用: **

在我看来, 虽然时常听到闭包这个概念, 但真正的应用还真不是很多, 也没看到或者想到比较经典的应用. 在 JavaScript 中, 使用得最多的, 恐怕还是将 function 作为 first class 的情况使用, 就好比你可以 var a = new Number(0); 可以将 a 当作函数的参数不断的进行传递, 你也可以 var f = new Function(arg1, arg2...., functionBody) [new Function("x", "y", "return (x + y)/2")] 来定义函数, 然后将 f 作为参数在函数中不断的传递; 但我一般不会让这个 function 产生闭包来进行传递, 而是会传递一个独立的 function 避免写多了自己都忘记是哪个了.

**JavaScript 闭包的实现:**

如开篇所说, JavaScript 中的闭包实现与 JavaScript 的 Scope Chain 是密不可分的.

首先在 JavaScript 的执行中会一直存在一个 Execute Context Stack (想想 JavaScript 解释器在看到一个 alert(x) 的时候, 如果没有上下文他怎么知道这个 x 是什么?), Execute Context Stack 中最下面一个一定是 GlobalContext, 
而在每一个函数的执行开始就会向这个 stack 中压入一个此 Function 的 Execution Context;

而一个 Execution Context 的组成分为三部分: 

1. Variable Object: 存储方法内的变量 vars, 方法传入的参数, 函数内定义的函数等等(函数表达式不保存), Variable Object 在任何时候是不可以被直接访问到的, 当然不同的 JS 引擎提供了访问接口就说不定了;

2. Scope Chain: 这个函数执行的时候用以寻找值的 Scope Chain, 这个 Scope Chain 由 Variable Object + All Parent Scopes 组成, Variable Object 会放在这个 Scope Chain 的最前面, 这也是为什么函数内的变量会被最先找到;

3. thisValue, 函数被调用的时候的 this 对象, 存储的就是函数的调用者(caller)的引用; 

对于 Variable Object 在不同的情况下会有不同的定义, 例如在全局的时候被称为 Global Object, 而在函数中则被称为 Activation Object 激活对象; 

正是由于有了 Execution Context 中的 Scope Chain, JavaScript 才能够使得在方法 bar() 
的内部访问到方法 foo() 中的变量 a, 才能够使方法 bar() 将变量 a 关闭在自己的作用范围内不让他随 foo() 方法的执行完毕而销毁;

## Fiveth

说个应用场景。
利用闭包可以给对象设置私有属性并利用特权(Privileged)方法访问私有属性。

```
var Foo = function(){
      var name = 'fooname';
      var age = 12;
      this.getName = function(){
          return name;
      };
      this.getAge = function(){
          return age;
      };
  };
  var foo = new Foo();

  foo.name;        //  => undefined
  foo.age;         //  => undefined
  foo.getName();   //  => 'fooname'
  foo.getAge();    //  => 12


```












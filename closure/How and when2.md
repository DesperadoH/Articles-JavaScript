

## Q1

最近看了一些 JavaScript 的内存泄露问题，看似没问题的代码原来存在内存泄露，而且部分还不知道怎么回事，比如：


```

function (element,a,b){
	element.onclick = function(){
		//TODO a b here
	}
}

```

这个就造成内存泄露了？

那如果element改为一个非DOM对象也是一样吗？

如果是的话，凡是闭包都会出现内存泄露?通过element.onclick = null可以解决？

## First

这个不叫「内存泄漏」。

这个代码运行之后，只要 element 不再被引用，a、b 也会被回收。题主的意图估计是希望 a、b 的生命周期比 element 短。那是你的设计错误。因为你把 element 的一个 event-handler 设计成依赖于 a、b，那 a、b 当然就要和 element 共生死了。题主给的这个逻辑用不用闭包都会有这个问题。如果硬要释放 a、b，那就是 release before use，会造成 null-dereference error。

### Second

```
作者：rambo
链接：https://www.zhihu.com/question/22806887/answer/39381628
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

var test_obj = {
    closure_fn: function () {
        var that = this;
        var val = setTimeout(function () { 
            console.log('Rambo!'); 
            that.closure_fn();
        }, 1000);
    }
};
test_obj.closure_fn();
test_obj = null;
// 尝试之后，你会发现这他妈才是可怕的。

var test_obj_2 = {
    closure_fn: function () {
        var that = test_obj_2;
        var val = setTimeout(function () { 
            console.log('Rambo!'); 
            that ? that.closure_fn() : clearTimeout(val);
        }, 1000);
    }
};



```

test_obj = null; 不会导致内存被回收，只是改变了test_obj的引用。源对象如果被其他地方引用，仍然不会被释放。




### Third

首先，这是一个闭包。

题主贴的代码里之所以造成内存泄露是因为循环引用。一个闭包在创建时会附有三个属性：VO、thisValue、scope chain，而其中scope chain是指向外层parent scope的引用。

因此这个闭包实际上保存了外层函数中element的引用，而element本身又引用了闭包，循环引用因此而见。

但是我记得hax同时提到，因为循环引用导致的泄露实际上是IE浏览器引擎的bug而导致的，而如果element不是dom对象，不产生循环引用也就不会leak了。





















































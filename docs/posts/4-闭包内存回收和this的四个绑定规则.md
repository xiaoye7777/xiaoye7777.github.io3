[[2022-08-15]]
#js
[[0-Readme-高级jscoderwhy]]

01:12看一点开头睡觉

## 闭包的内存泄漏

这里的一个数字1占用4个字节(byte)
> 关于这个数字是4个字节还是8个字节。一般来说js的number类型有浮点型所以按理来说number类型就是8字节，但是v8引擎出现了，v8搞了个什么sim小数字？用四个字节来表示就可以了（-2的31次方 ~ 2的31次方 - 1）因为有0多以减去1。在这个范围的数字用4个字节就足够表示了。所以V8就只给4个字节。就这么个意思，看看就好

`4*1024=4kb，4byte*1024*1024=4m` 

内存泄漏案例：

![内存泄漏案例](https://github.com/xiaoye7777/imageHosting/blob/main/images/%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E6%A1%88%E4%BE%8B.png?raw=true)

开辟一个1024*1024个元素位置的数组，存进去的数字是4字节大小，那么这个就是4M大小的数组。   
在外面设置一个数组用来接收（使数组们能从GO有引用，防止回收）  
然后循环创建100次，这时候内存里面的情况就是图中这样的。

![GC回收上面的这些垃圾](https://github.com/xiaoye7777/imageHosting/blob/main/images/gc%E5%9B%9E%E6%94%B6%E5%9E%83%E5%9C%BE.png?raw=true)

用了setTimeOut两秒再去掉GO的引用，然后等到GC下一波回收的时候刚好就回收掉了，看图上右边骤降那里就是GC回收的时候

这个讲了好久，老师一直在调

01:57,看到了1h10min处，明天继续。
45分钟看了1h10min的内容加做笔记，还可以。

[[2022-08-15]]17:16开搞开搞

每个标签都是一个进程，每个标签都有自己的GO  


## js闭包引用的自由变量的销毁

先补充一个细节：

![](https://github.com/xiaoye7777/imageHosting/blob/main/images/%E8%A1%A5%E5%85%85%E7%BB%86%E8%8A%82%E9%97%AD%E5%8C%85%EF%BC%8C%E9%94%80%E6%AF%81.png?raw=true)

有一种情况，你把这个foo执行一下返回里面的函数bar给fn这个变量  
然后，同样的又执行一次返回给另一个变量baz。那么这两个就是互不相干的，两套东西，删掉了fn`fn = null`并不影响baz这边，只是删掉fn那边搞出来的AO和函数对象

**正文**：如果在bar里面不打印age呢（这时候的age变量属于是一个虽然在闭包保留下来的AO里面，但是他不会被用到）

这种用不到的变量会被销毁掉，怎么销毁呢？不知道，V8干的，老师也没讲

### 知识点

debugger，在程序里面写一个这个，就会在这里停下来，可以在浏览器里调试



## this指向

有个现象：在浏览器里面全局的this指向window  
但是在node里面指向空对象`{}`

原因是：在node里会把这个js文件当作一个模块，加载，编译，放进一个函数里，然后执行这个函数.call({})

> call是绑定this的方法  
> 比如foo.apply("abc")就是说foo的this指向abc

node源码里把一个空对象传了好几跳给这里的this

那么，this指向谁在哪里看呢？  
所有的函数在调用的时候都会创建一个执行上下文  
里面包含了调用栈，AO等等，**还有this**  
就是在这里看，他是动态绑定，在函数真正要执行的时候才会绑定上去

this指向和函数所处的位置没有关系  
跟函数被调用的方式有关

定义一个函数，我们采用三种不同的方式对它进行调用，它产生了三种不同的结果，说明：  
1.函数在调用时，JavaScript会默认给this绑定一个值
2.this的绑定和定义的位置(编写的位置)没有关系;
3.this的绑定和调用方式以及调用的位置有关系
4.this是在运行时被绑定的

## this的绑定规则

### 绑定一:默认绑定

**独立函数的时候**  
就是直接在全局`foo()`这样

```javascript
var obj = {
	name: "why",
	foo: function() {
		console.log(this)
    }    
var bar = obj.foo
bar()  //window

```

即使是在这种情况下，this仍然是window，因为下面的bar是独立调用，和在哪里定义没有关系

### 绑定二:隐式绑定

**通过某个对象进行调用的**

```javascript
function foo() ｛
  function bar() ｛
	console.log(this)
  }
  return bar
}

var fn = foo()
fn()

var obj = {
  name: "why"
  eating: fn
}
obj.eating()  //👈隐式绑定

```

这时候this就是指向obj，这个叫隐式绑定

### 绑定三:显示绑定

`foo.call() 和 foo.apply()`括号里面可以指定this指向谁

那call和apply有什么区别？  传参的方式不一样，一个直接接着写，一个放数组里。

```javascript
function sum(num1，num2，num3) {
	console.log(num1 + num2 + num3，this)
}
sum.call("call",28,38,40)
sum.apply("apply", [20,30,40])
```

**进阶：**

bind绑定：如果像绑定某个东西，然后用很多次的话每次都要写call（”abc“）这种，重复太多，那就用bind搞一个新的函数

```JavaScript
var newFoo = foo.bind("abc")
newFoo()
newFoo()
newFoo()
...
```

这里newFoo是独立函数调用，按理来说是默认绑定window的，但是newFoo同时显示绑定了”abc“，拿着时候显示绑定的优先级更高，所以this是”abc“

### 绑定四: new绑定

JavaScript中的函数可以当做一个类的构造函数来使用，也就是使用new关键字。

使用new关键字来调用函数，会执行如下的操作:

1. 创建一个全新的对象
2. 这个新对象会被执行prototype链接
3. 这个新对象会绑定到函数调用的this上（this的绑定在这个步骤完成）
4. 如果函数没有返回其他对象，表达式会返回这个新对象

```javascript
function Person(name, age) {
    this.namm = name
    this.age = age
}

var p1 = new Person("abc", 20)
console.log(p1.name, p1.age)

var p2 = new Person("cba", 18)
console.log(p2.name, p2.age)
```

上面这个过程中，new的时候就会在Person函数（构造器）里创建一个空对象，然后把这个对象赋值给Person里的this❓**疑问**，然后执行完了再把这个this返回出来。（说实话不是特别明白）

？this是创建的空对象说是。这样还算能理解

this = 创建出来的对象

ok，差不多消化了。

这个就是new绑定this

### this绑定优先级

new > 显示(bind); 显示 > 隐式; 默认最低

19:54看完了。2h看了2h视频，好慢，感觉我一直都是2倍速啊？中间停下来做笔记也太花时间了吧:thinking:
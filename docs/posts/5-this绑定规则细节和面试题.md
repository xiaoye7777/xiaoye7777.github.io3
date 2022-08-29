[[2022-08-16]]
#js
[[0-Readme-高级jscoderwhy]]
20:02开搞开搞，本来是下午要开始的事情，一道hard算法题给我拖到了晚上

手动实现apply、call和bind函数

题外话：上来就放一段我看不懂的代码，就给看看，说后面再讲

```JavaScript
//来自你不知道的javascript 
function foo（eL） {
	console.log（el,this.id）
}
var obj = {
	id:'awesome'
}
[1,2,3].forEach（foo,obj）
```

### setTimeout的this

```javascript
setTimeout (function() {
  //...
}, 5000)     
//相当于，setTimeout(fn, time)这样传入两个参数
//那么这个函数里面是怎么定义的呢，构造函数内部
function setTimeout(fn,time) {
    //fn();    如果是这样子直接独立调用，那么this就是指向window
    //fn.call("abc")  那要是这样子，就指向string：abc了
}

//答案是：直接调用，所以打印this的时候，打印出来的是window
```

> 但是函数是箭头函数就不一样了，要去上层作用域找this的

### 监听事件的this

```javascript
boxDiv.addEventListener('click',function() {
    console.log(this)
    //fn.call("boxDiv")  老师觉得这个里面应该是这样子的
})
```

### forEach的this

```javascript
var names["abc","cba"，"nba"]
names.forEach(function(item) {
	console.Log(item, this)   //本来这里是指向window的
}，"abc")     //但是他也可以接受第二个参数作为绑定的this

names.map(function(item) {      //同理
	console.log(item, this)
}，"cba")
```

默认是指向window，但是可以接受第二个参数，this指向这个参数

### 规则优先级

👇看这一句话就够了其实  
new绑定>显示绑定(apply/catl/bind)>隐式绑定(obj.foo())>默认绑定(独立函数调用)

```javascript
var obj = {
	name:obj"
	foo:function() {
        console.log(this)
    }
}

obj.foo.call("abc")  //abc,显示 > 隐式

var f = new obj.foo()   //这里是foo
```



#### 显示 VS 隐式

显示高于隐式`obj.foo.call("abc")  //abc`    
`显示里面互相比较呢？bind最高，其他两个不知道，试试就知道了，foo,call("abc").apply("cba")这样子自己试试就行了`

#### new绑定 VS 隐式

new绑定高于隐式  

#### new绑定 VS 显式

new不能和call，apply一起使用  
**new vs bindnew绑定 VS 隐式**  
new > bind

### this规则之外 - 忽略显示绑定

`foo.call(null) 或者 foo.apply(undefined)`  
👆这个结果是this指向了window（call或者apply这里没区别）  
`var bar = foo.bind(null);      bar()`结果也是window

`(obj2.bar = obj1.foo)()`这个前面 的代码被看作一个整体，后面的括号对他进行一个函数式执行（在这个语句之前的函数后面一定要加上分号；）

#### 箭头函数-浅了解一下箭头函数

不绑定this，arguments，不能作为构造函数

```javascript
var foo = (num1,num2,num3) => {}
//常用👇
nums.forEach((item, index, arr) => {})
//简写,会默认把这一行执行体的结果return（就是不用写return了）
/nums.forEach(item => console.log(item))
```

filter/map/reduce一起使用：  
需求是把数组[10，20，65，78]里面的偶数全部相加

```JavaScript
var result = nums.filter(item => item % 2 === 0)
                 .map(item = > item * 100)
                 .reduce((preValue,item) => preValue + item)
console.log(result)  
//filter:拿出来全部偶数，map对每个乘以100，reduce相加
```

在网上看了一下reduce的文章[👉这里👈](https://segmentfault.com/a/1190000010731933),有点高级，只是知道基本用法咋用了，后面再看吧。

如果返回对象呢?`var bar = () => ({name:"xy", age:23})`👈那就这样写，外面加一个括号他才知道这个是一个对象，要直接return。如果没有，那他就不知道这个大括号里面的到底是函数执行体还是对象了（所以会判为那个呢？）
试过了，报错了`Uncaught SyntaxError: Unexpected token ':'`。在vscode里写完就提示错误了，让我在age后面冒号改为分号。也不懂原理，应该看看怎么分辨对象和函数体，这个得看源码才知道吧，不管了先。

#### 箭头函数中的this

不使用四个this规则（就是压根不绑定）   
那么要去哪里找this呢？去外层作用域 

```JavaScript
var foo () => {
    console.log(this)
}
//各种方式调用这个函数
foo()
var obj = {foo: foo}
obj.foo()
foo.call("abc")
//👆以上调用结果打印出来的全部都是window
```

#### 箭头函数应用场景（印象深刻）

场景就是要从服务器请求数据放到外层的data上

```javascript
var obj = {
    data: [],
    getData: function() {
        var _this = this        //没有箭头函数的处理方式
        setTimeout(function() {
            var result = ["123", "456", "789"] //假设拿到的数据
        	_this.data = result
        }， 2000)
    }
}

obj.getData()          

//有箭头函数以后，直接用this
setTimeout(() => {
      var result = ["123", "456", "789"] //假设拿到的数据
      this.data = result
}， 2000)

```

setTimeout的this指向window，所以在那个里面用this.data = result是行不通的，用了箭头函数以后，因为这个函数本身没有this，不存在。就往上层作用域找，这不就完事了吗。完美！

### 几道面试题-关于this的

#### 面试题1

```javascript
var name = "window";
var person = {
  name: "person",
  sayName: function () {
    console.log(this.name);
  }
};
function sayName() {
  var sss = person.sayName;
  sss();               //window,是下面的sayName()调用的所以是独立调用
  person.sayName();    //person，隐式调用
  (person.sayName)();  //person，隐式调用 👆加不加()和上面没区别
  (b = person.sayName)(); //window，但是这里有赋值表达式，算是把右边的独立调用
}
sayName();
```

#### 面试题2  -  对象obj，隐式绑定题目

```JavaScript
var name = 'window'
var person1 = {
  name: 'person1',
  foo1: function () {
    console.log(this.name)
  },
  foo2: () => console.log(this.name),
  foo3: function () {
    return function () {
      console.log(this.name)
    }
  },
  foo4: function () {
    return () => {
      console.log(this.name)
    }
  }
}

var person2 = { name: 'person2' }

//十道题：👇
person1.foo1();               //person1 ,显而易见，小学二年级就会的
person1.foo1.call(person2);   //person2，显示 > 隐式

person1.foo2();               //window，因为是箭头函数，所以跳出person1
person1.foo2.call(person2);   //👉重点！！👈，window，还是箭头函数，箭头函数才不管你的绑定规则，没有this还绑定啥，只能去外面找另外的this（这样子理解对吗，规则只针对当前this？去外层找的不算？）

person1.foo3()();            //window，这个相当于( person1.foo3() ) (),独立函数调用
person1.foo3.call(person2)();  //window，加了call也一样，在括号里面，最后还是独立调用
person1.foo3().call(person2); //person2，call起作用

person1.foo4()();            //person1，箭头函数没有this，找上层的foo4的this是person1
//注意：这里最后是独立函数调用的时候没有this，去找箭头函数的上层，是箭头函数的上层哦
person1.foo4.call(person2)();   //person2，这里箭头函数的上层被显示绑定了person2了
person1.foo4().call(person2);   //person1，给箭头函数call会怎么样？不鸟你，按平常找箭头函数的上层作用域
```

[[2022-08-18]]02:25睡觉睡觉，今天看到了2h处，还差40min，早上起来看完。
10:16开搞开搞，40分钟

继续做了上面的面试题2👆感觉对于this的细节了解了不少，很有用

#### 面试题3  -- 构造函数，new 的this指向

```javascript
var name = 'window'
function Person (name) {
  this.name = name
  this.foo1 = function () {
    console.log(this.name)
  },
  this.foo2 = () => console.log(this.name),
  this.foo3 = function () {
    return function () {
      console.log(this.name)
    }
  },
  this.foo4 = function () {
    return () => {
      console.log(this.name)
    }
  }
}
var person1 = new Person('person1')
var person2 = new Person('person2')

//题：👇
person1.foo1()               // person1，小学二年级就会的隐式绑定
person1.foo1.call(person2)   //person2，正常无坑

person1.foo2()               //person1，箭头函数的上层，就是Person的person1这个实例
person1.foo2.call(person2)   //person1，箭头函数本身call了没作用的，所以和上面一样

person1.foo3()()             //window，独立函数调用咯，不会就看上面那题的第五题
person1.foo3.call(person2)() //window，上层call了person2又怎样，最后还是独立调用咯
person1.foo3().call(person2) //person2，这里call成功了终于

person1.foo4()()             //person1
person1.foo4.call(person2)() //person2 ，箭头函数的上层被绑定了person2
person1.foo4().call(person2) //person1，给箭头函数call没有用的
```



#### 面试题4  构造函数里有obj

```javascript
var name = 'window'
function Person (name) {
  this.name = name
  this.obj = {
    name: 'obj',
    foo1: function () {
      return function () {
        console.log(this.name)
      }
    },
    foo2: function () {
      return () => {
        console.log(this.name)
      }
    }
  }
}
var person1 = new Person('person1')
var person2 = new Person('person2')

person1.obj.foo1()()               //window，独立调用
person1.obj.foo1.call(person2)()   //window，独立调用的，函数里的上层绑了别的也用不着啊
person1.obj.foo1().call(person2)   //person2，直接给要执行的函数call了person2，当然可以

person1.obj.foo2()()               //obj，独立调用箭头函数没有this，找箭头函数上层foo2的this，foo2的this就是指向obj咯
person1.obj.foo2.call(person2)()  //person2，独立调用，箭头函数找上层，上层绑了person2
person1.obj.foo2().call(person2)  //obj，箭头函数call了没用，找上层obj
```

❗**注意：**

> 注意：`var obj = { name: "xiaoye", age:23 }`这里的大括号里不是块级作用域，不是块级作用域。所以不能再说大括号包括的就是块级作用域了🫵，我以前一直这么想来着，没考虑到这个情况 
>
> ```javascript
> var obj = {
>   name:"xiaoye"
>   foo: function() {
>     //这里的上层作用域是，window，可不是obj奥
>   }
> }
> ```
>
> 这里和上面的题目好像有点冲突？为啥面试题4的最后那两个是obj啊？


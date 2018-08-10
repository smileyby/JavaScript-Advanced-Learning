## this 总结

> this: 函数执行的主体，谁把函数执行了，谁就是执行的主体（和函数在哪执行的，以及在哪定义的没有直接的关系）

`JS非严格模式下`

1、 给元素的某一事件绑定方法，当事件触发，方法执行的时候，绑定的这个方法中的this一般是当前操作的这个DOM元素

```javascript
oBox.onclick = function(){
	//=> 当绑定的方法执行，方法中的this:oBox
	//=> 如何让绑定的方法执行
	//=> 1、手动点击oBox，经由浏览器触发点击事件
	//=> 2、oBox.onclick();
	// ...
}

//=> 在IE6~8下，如果我们使用的是DOM2事件绑定方法执行的时候，里面的this不是当前元素而是window

oBox.attachEvent('onclick', function(){
	//=> this:window
});
```

2、自执行函数，方法中的this一般都是window

```javascript
var obj = {
	fn: (function(){
		//=> this: window
		return function(){}
	})()
}

~function(){
	//=> this: window
}()
```

3、 方法执行的时候，看方法名之前是否有`点`，有的话，`点`前面是谁this就是谁，没有的话this是window

```javascript
function fn(){
	console.log(this);
}
var obj = {fn: fn};
fn(); //=> this: window
obj.fn(); //=> this: obj


//==================
[].slice(); //=> 数组通过原型查找机制，把Array.prototype上的slice方法找到，并且让方法执行
[]/slice(); //=> this: []
[].__proto__.slice(); //=> this: [].__proto__
Array.prototype.slice(); //=>this: Array.prototype
```

4、在构造函数执行的时候，函数体中的this都是当前类的实例

```javascript
function Fn(){}
var f = new Fn();
```

5、 使用Funcion.prototype上提供的call/apply/bind实现this的改变


```javascript
var obj = {name: 'a'};
function fn(num1, num2){
	this.total = num1 + num2;
}

//=> call基础语法的应用
// fn(10,20); //=> this:window
obj.fn(10,20); //=> obj中没有fn这个属性，属性值是undefined，它不能作为函数执行，所以会报错：TypeError(undefined is not a function ...)
fn.call(obj, num1, num2); //=> 首先让fn中的this指向传递的第一个参数obj，然后执行fn这个函数，此时fn中的this->obj num1->10 num2-> 20

fn.call(10,20); //=> fn中的this：10 num1:20 num2: undefined

fn.call(); //=> fn中的this：window num1=num2=undefined

fn.call(null);
fn.call(undefined); //=> 第一个参数不管写的诗null还是undefined都代表没有只想的this，所以函数中的this依然诗window
```

```javascript
var obj = {name: 'a'};
function fn(num1, num2){
	this.total = num1 + num2;
}

//=> apply的语法和作用 跟call基本上完全类似只有一个区别

fn.call(obj,10,20);
<=>
fn.apply(obj,[10,20]); //=> apply方法调用的时候，第一个参数诗this执行，第二个参数是一个数组，数组中包含了所有需要给函数传递的实参（语法要求诗写成一个数组，但是和call一样也是一项项给形参赋值的）
```

```javascript
~function(){
	//=> this: obj
}.call(obj);

Array.prototype.slice.call(arguments); //=> slice中的this：arguments
```

所有的this情况遇到call/apply的时候命都以call/apply指向的this为主

```javascript
var obj = {name: 'a'};
function fn(num1, num2){
	this.total = num1 + num2;
}

//=> bind方法
fn.call(obj,10,20); //=> 改变fn中的this，而且把fn立即执行了

fn.bind(obj,10,20); //=> 虽然改变了fn中的this，但是没有把fn执行，它属于预先处理this和实参，不会立即执行，只有达到某个特点条件，才会被出发（IE6~8不兼容）

//=====================================
//=> 使用bind的意义
var obj = {name: 'a'};
function fn(num1, num2){
	this.total = num1 + num2;
}

//=> 需求：一秒后执行fn（定时器驱动），执行fn的时候，让fn的this->obj,并且给fn传递两个实参10，20

setTimeout(fn,1000); //=> 1s后确实执行了fn，但是此时fn中的this是window不是obj，而且没有传递10，20

setTimeout(fn.call(obj,10,20), 1000); //=> 虽然实现了this的改变和参数的传递，但是它设置定时器的时候就把fn执行了，而不是等到1000ms后，1000ms后执行的诗fn执行的返回结果

setTimeout(function(){
	//=> 1000ms后执行的诗匿名函数：我们可以在匿名函数里面把fn执行，并且实现需求
	fn.call(obj,10,20);
}, 1000);

setTimeout(fn.bind(obj,10,20), 1000); //=> 使用bind预处理（不兼容）
```

6、 在ES6中，新增加了剪头函数，箭头函数中没有执行主体，箭头函数中额this会继承它宿主环境中的this

```javascript
var obj = {
	fn: function(){
		//=> this: obj
		setTimeout(function(){
			//=> this: window 不管在哪执行定时器中的this是window
		},1000);

		var _this = this;
		setTimeout(function(){
			//=> _this: obj
		}, 1000);

		setTimeout(()=>{
			//=> this: 箭头函数中的this继承宿主环境中的this
		},1000);
	}
}
obj.fn();
```

**`JS严格模式下的this情况`**

```javascript
"use strict"; //=> 代码第一行加上这句话，就开启了JS中的严格模式
```

```javascript
function fn(){
	"use strict"; //=> 只在当前私有作用域中使用严格模式（也需要处于当前作用域第一行）
}
```

> 非严格模式下不明确执行主体，浏览器认为执行主题是window（this一般是window）但是在严格模式下，执行主题不明确，this是undefined

```javascript
"use strict";
~function(){
	//=> this: undefined
}();

fn(); //=> this :undefined
window.fn(); //=> this: window

fn.call(); //=> this: undefined
fn.call(window); //=> this: window
fn.call(null/undefined); //=> this: null/undefined
```


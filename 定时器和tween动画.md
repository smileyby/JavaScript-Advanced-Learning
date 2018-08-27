## 定时器

> JS中的定时器一共有两种
> 
> 1、window.setTimeout([function], [interval]):设置一个定时器，到达指定时间后执行对应的方法（执行一次定时器就结束了）
>
> 2、 window.setInterval([function], [interval]): 设置了一个定时器，到达指定时间后执行对应当的方法（以后每隔这么长的时间都要重新的执行定时器中的方法，直到定时器清除为止，执行多次）

```javascript
var count = 0;
setTimeout(function(){
	console.log(++count); //=> 1
}, 1000);
```

```javascript
var count = 0;
setInterval(function(){
	console.log(++count); // 1，2，3，4....
}, 1000);
```

> 定时器的返回值： 当我们设置定时器（不管是setTimeout还是setInterval），都会有一个返回值，返回的是一个数字，代表当前是在浏览器中设置的第几个定时器（返回的是定时器的序号）
> 
> 1、setTimeout和setInterval虽然是处理不同需求的定时器，但是都是浏览器的定时器，所以设置的时候，返回的序号是依次排列的
> 
> 2、setInterval设置完成定时器会有一个返回值，不管执行多少次，这个代表序号的返回值不变（设置定时器就有返回值，执行多少次是定时器的处理）

```javascript
var timer1 = setTimeout(function(){
	
}, 1000);
console.log(timer1); //=> 1

var timer2 = setInterval(function(){

}, 1000);
console.log(timer2); //=> 2
```

> 定时器的清除
> 
> clearTimeout([定时器的序号])
> 
> clearInterval([定时器的序号])

定时器需要手动清楚

```javascript
var t1 = setTimeout(function(){
	//=> 当方法执行完成后，定时器没用了，我们清除定时器即可
	//=> clearTimeout(t1); //=> t1存储的就是当前定时器的编号
	//=> clearInterval(t1); //=> 使用它也可以清除掉

	clearTimeout(t1); //=> 在浏览器内部把定时器清除掉（相当于银行业务员在系统中清除我们的排队号）
	t1 = null; //=> 我们手动吧之前存储序号的变量赋值为null（相当于我们把排队号那个纸撕掉）
}, 1000);
```

## JS中的同步编程和异步编程

JS是单线程的（一次只能执行一个任务，当前任务没有完成，下面的任务是不进行处理的）

> 同步编程（sync:synchronize）: 任务是按照顺序一件一件完成的当前任务没有完成，下面的任务不进行处理
> 
> 异步编程（async）：当前任务在等待执行的时候，我们不去执行，继续完成下面的任务，当下面的任务完成后，而是也到达等待的时间，才去完成当前的任务
> + 定时器都是异步编程的
> + 所有的事件绑定也是异步编程的
> + AJAX中有异步编程
> + 有些人把回调函数当做异步编程（理解起来比较牵强）

```javascript
var startTime = new Date();
for(var i = 0; i < 1000000; i++){
	if(i === 999999){
		console.log('no'); //=> 1 'no'
	}
}
console.log(new Date() - startTime);
console.log('ok'); //=> 2 'ok'
```

```javascript
var n = 0;
setTimeout(function(){
	n++;
	console.log(n); //=>2. 1
}, 1000);
console.log(n); //=>1. 0
```

## 同步异步编程的核心原理

> JS中有两个任务队列（存放任务列表的空间就是任务队列）
> 
> 1. 主任务队列：同步执行任务（从上到下一次执行）
> 2. 等待任务队列：异步执行任务

```javascript
setTimeout(function(){
	console.log(1);
}, 50);

setTimeout(function(){
	console.log(2);
}, 10);

setTimeout(function(){
	console.log(3);
}, 30);

for(var i = 0; i < 10000000; i++){
	
}

console.log(4);

// 4-2-3-1
```

```javascript
for(var i = 1; i <= 5; i++){
	setTimeout(function(){
		console.log(i);
	}, i*10);
}

//=> [主任务队列]
// i=0 创建一个定时器
// i=1 创建一个定时器
// i=2 创建一个定时器
// i=3 创建一个定时器
// i=4 创建一个定时器
// i=5 创建一个定时器
// i=6 循环结束，此时主任务队列中的方法都已经执行完成，到等待任务队列中找到时间的方法，拿到主任务队列中执行
// 执行它function(){console.log(i);},i不是自己私有的，找全局下的i（此时的i已经是6了）

//=> [等待任务队列] 等待任务队列中谁先来的先执行谁，等待时间相同的按照前后顺序一次执行
// 0ms 后执行某方法
// 10ms 后执行某方法
// 20ms 后执行某方法
// 30ms 后执行某方法
// 40ms 后执行某方法
// 50ms 后执行某方法
```

```javascript
for(var i = 0; i < 6; i++){
	oDiv.onclick = function(){
		alert(i);
	}
}
```

```javascript
//=> 让主任务队列的任务少一点
var page1 = (function(){
	return {
		init: function(){
	
		}
	}
})();
page1.init();

var page2 = (function(){
	return {
		init: function(){
	
		}
	}
})();
setTimeout(page2.init, 0);
//=> 定时器等待时间设置为0也不是立马执行，浏览器都有一个最小的反应时间（谷歌:5-7ms IE: 10-13ms ...），写0也需要等到几毫秒
```
### 固定步长匀速动画


### 固定时间匀速动画

```javascript
// 1、target: 目标值
// 2、begin: 起始值
// 3、change: 总距离 target-begin
// 4、duration: 总时间
// 5、time: 已经运动的时间

// time/duration 已经走了百分之多少的距离
// time/duration*change 已经运动的距离
// time/duration*change + begin 当前元素的位置

function linear(t, b, c, d){
	return t / d * c + b;
}
```

### setTimeout 模拟 setInterval（递归）

```javascript
function move(){
	//=> 每一次开始之间都把上一次创建的那个定时器清除掉（内存优化）
	clearTimeout(oBox.timer);
	//=> 添加边界结束定时器
	oBox.timer = setTimeout(move, 17);
}

move();

// 边界判断加步长
```

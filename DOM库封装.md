DOM库封装
========

## children方法的封装

> children: 获取当前容器所有元素节点（只获取儿子辈子元素）
> 
> childrenNodes: 获取当前元素下所有节点（包括注释节点在内）
> 
> 在IE6~8下children获取時会把注视节点当做元素节点处理

**[突然忘记DOM相关知识，点这里就可以看到](https://github.com/smileyby/JavaScript-basic/blob/master/JS%E5%9F%BA%E7%A1%8013-DOM%E5%9F%BA%E7%A1%80%E9%83%A8%E5%88%86.md)**

### 封装方法children: 不管在什么浏览器下都只获取第一代子元素，不指定标签名

```javascript
var utils = (function(){
	var toArray = function(classAry){
		console.log(classAry);
		var ary = null;
		try{
			ary = Array.prototype.slice.call(classAry);
		} catch(e) {
			for(var i = 0; i < classAry.length; i += 1){
				ary[ary.length] = classAry[i];
			}
		}
		return ary;
	}

	return {
		toArray: toArray
	}
})();
```

```javascript
//=> 排除法
var box = document.getElementsByClassName('box')[0];
function children(curEle){
	var childList = curEle.children; //=> 获取curEle下的儿子辈子元素
	childList = utils.toArray(childList);
	for(var i = 0; i < childList.length; i += 1){
		var item = childList[i];
		if(item.nodeType !== 1){ //=>
			childList.splice(i,1);
			i -= 1;
		}
	}
	return childList;
}
console.log(children(box));

//=> 假设法
function children(curEle){
	var result = [],
		childList = curEle.childNodes; //=> childNodes 只获取当前元素下儿子辈子元素包括（换行、注释、所有儿子辈子元素节点）
		
	for(var i = 0; i < childList.length; i += 1){
		var item = childList[i];
		item.nodeType === 1 ? result.push(childList[i]) : null;
	}
	return result;
}
console.log(children(box));
```

### 获取当前元素下指定标签名的所有子节点元素

```javascript
function children(curEle, tagName){
	var result = [],
		childList = curEle.childNodes;

	//=> 保证比较時的统一性，全部转化为小写进行比较	
	tagName = tagName.toLowerCase();

	//=> 首次筛选，选择nodeType是1的元素存储在result中
	for(var i = 0; i < childList.length; i += 1){
		if(childList[i].nodeType === 1){
			result.push(childList[i]);
		}
	}

	//=> 第二次筛选，去除标签名不符合条件的元素
	for(var k = 0; k < result.length; k += 1){
		var item = result[k];
		console.log(result[k]);
		if(item.tagName.toLowerCase() !== tagName){
			result.splice(k, 1);
			k -= 1;
		}
	}

	return result;
}
console.log(children(box, 'p'));

//=> 优化
function children(curEle, tagName){
	var result = [],
		childList = curEle.childNodes;
	for(var i = 0; i < childList.length; i += 1){
		var item = childList[i];
		if(item.nodeType === 1){
			if(typeof tagName !== 'undefined'){

				//=> 保证比较時的统一性，全部转化为小写进行比较
				if(item.tagName.toLowerCase() === tagName.toLowerCase()){
					result.push(item);
				}
			} else {

				//=> 没有传标签名，默认返回全部儿子辈子元素
				result.push(item);
			}
		}
	}
	return result;
}
console.log(children(box, 'p'));
```

## getElementsByClassName 封装

> IE6~8下不兼容
> 
> 语法: `[context].getElementsByClassName([class name])`
> 
> 在指定上下文中，通过样式类名获取一组元素集合
> 
> 如果传递多个类名，会获取到同时具备这些样式类的元素（多个样式类之间加多少个空格都无所谓，且不计较类名的前后顺序）`document.getElementsByClassName('box1 box2')`

### 只处理传递单个样式类名的情况

> + 首尾可能有多个空格
> + strClass: 传递进来的样式类名
> + context: 限定获取的范围（上下文）默认为document

```javascript
var box = document.getElementsByClassName('box')[0];

function getElementsByClassName(context, strClass){
	conetxt = context || document;
	strClass = strClass.replace(/ +/g, '');

	var result = [],
		nodeList = context.getElementsByTagName('*'),
		reg = new RegExp('(^| +)'+ strClass +'( +|$)'); //=> 正则中存在动态内容使用构造函数创建

	//=> indexOf 验证strClass時 遇到传递类名为box1，实际类名为box1-1不能验证
	//=> 正则 /\bstrClass\b/ \b 识别中划线 会出现和indexOf一样的问题
	//=> 所以选择正则 /(^| +)strClass( +|$)/	

	for(var i = 0; i < nodeList.length; i += 1){
		var item = nodeList[i];
		reg.test(item.className) ? result.push(item) : null;
	}
	return result;
}
console.log(getElementsByClassName(box, 'box1'));
```

### 处理传递多个类名的情况

> 思路：获取到当前元素下的所有子元素，通过正则匹配每个子元素的类名，完全匹配则存入result数组中并返回
> 
> + 传递进来的strClass 类名之间可能存在多个空格的情况，所以直接正则按照空格拆分
> + 拿到某一个子元素的类名后，与拆分后的strClass数组中的每一项进行正则匹配，如果有一项不符合条件则当前子元素不符合，直接退出正则匹配的循环，继续检查下一个子元素是否符合条件，直接所有子元素都检查完毕


```javascript
//=> 假设法
var box = document.getElementsByClassName('box')[0];
function getElementsByClassName(context, strClass){
	conetxt = context || document;
	strClass = strClass.split(/ +/);

	var result = [],
		nodeList = context.getElementsByTagName('*');

	for(var i = 0; i < nodeList.length; i += 1){
		var item = nodeList[i],
			itemClass = item.className,
			flag = true;
		for(var k = 0; k < strClass.length; k += 1){
			var reg = new RegExp('(^| +)'+ strClass[k] +'( +|$)');
			if(!reg.test(itemClass)){
				flag = false;
				break;
			}
		}
		flag? result.push(nodeList[i]) : null;
	}
	return result;
}
console.log(getElementsByClassName(box, 'box1-1 box2'));
```

> 思路： 先把传递进来的strClass样式类拆分成数组，数组中的每一项都去和当前子元素的类名比较，不符合条件，就从nodeList中剔除掉，strClass中的每一项都和nodeList中的每一项比较完毕后，最终返回的就是符合条件的子元素

```javascript
//=> 排除法
var box = document.getElementsByClassName('box')[0];
function getElementsByClassName(context, strClass){
	conetxt = context || document;
	strClass = strClass.split(/ +/);

	var result = [],
		nodeList = context.getElementsByTagName('*');
		nodeList = utils.toArray(nodeList);

		for (var i = 0; i < strClass.length; i += 1) {
			var reg = RegExp('(^| +)'+strClass[i]+'( +|$)');
			for(var k = 0; k < nodeList.length; k += 1){
				if(!reg.test(nodeList[k].className)){
					nodeList.splice(k, 1);
					k--;
				}
			}
		}
	return nodeList;
}
console.log(getElementsByClassName(box, 'box1 box2'));
```

```javascript
//=> 优化，判断是否为IE6~8
function getElementsByClassName(context, strClass){
	if('getElementsByClassName' in document){
		return context.getElementsByClassName(strClass);
	}
	//=> 下面代码同上
}
```

## 基于惰性思想的高级单例模式的作用和优势

> 什么是惰性思想：能够一次解决的，绝不会重复处理，属于JS的一种优化
> 
> + 生成闭包，存储以后经常用到的变量
> + 控制函数的内外部使用

```javascript
var utils = (function(){
	var flag = xxx;
	function setCss(){};
	function getCss(){};
	function css(){
		getCss();
		setCss();
	}

	return {
		css: css
	}
})();
//=> 其中getCss和setCss方法都没有暴露出来，只供css内部使用
```

### DOM封装-惰性思想函数重写

```javascript
function getCss(){
	if(windw.getComputedStyle){
		getCss = function(curEle, attr){
			return window.getComputedStyle(curEle, null)[attr]
		}
	}else{
		getCss = function(curEle, attr){
			return curEle.currentStyle[attr]
		}
	}
	return getCss(curEle, attr);
}

//=> getCss第一次执行大方法，判断是标准浏览器还是非标准浏览器
//=> 下一次带调用getCss方法的时候，getCss已经被改写，不需要进行重复的判断
```

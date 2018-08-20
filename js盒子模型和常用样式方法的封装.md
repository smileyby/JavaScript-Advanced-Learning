JS盒子模型和常用样式方法的封装
===========================

## 盒子模型

### 传统CSS盒子模型

> width、height: 不是盒子的宽高，而是盒子中内容的宽度和高度
> 
> 盒子的宽高 = width + padding(left/right) + border(left/right)

### CSS3新增加的盒子模型

> `box-sizing:border-box;` width和height不仅仅是内容的宽度，而是代表整个盒子的宽高（已包含padding和border），以后修改padding或者border的值，盒子大小是不变，内容宽高会随着改变

### JS盒子模型

> 提供一些属性和方法用来描述当前盒子的样式

**`clientWidth / clientHeight`**

> 当前盒子的可视区域的宽度和高度
> 
> 可视区域：内容宽高+padding
> 
> clientWidth = width + padding(left/right)
> 
> clientHieght = height + padding(top/bottom)
> 
> 和内容是否溢出以及我们是否设置了overflow:hidden;没有关系

```javascript
//=> 获取当前页面一屏幕的宽度（加或者是为了兼容所有浏览器，低版本浏览器不识别）document.documentElement
document.documentElement.clientWidth || document.body.clientWidth;

//=> 获取当前页面一屏幕的高度
document.documentElement
document.documentElement.clientHeight || document.body.clientHeight;
```

> 小测试
> 
> 让一个盒子在整个页面水平和垂直都居中的位置？如果不知道盒子的宽度和高度如何处理？

```css
/* 使用定位，需要知道盒子宽高 */
.box {
	position: absolute;
	left: 50%;
	top: 50%;
	margin-top: -50px 0 0 -100px;
	width: 200px;
	height: 100px;
}
/* 使用定位，不同知道盒子宽高(不兼容低版本IE浏览器) */
.box {
	position: absolute;
	left: 0;
	top: 0;
	bottom: 0;
	right: 0;
	margin: auto;
	width: 200px;
	height: 100px;
}
/* 使用CSS3的flex伸缩盒模型 */
html {
	height: 100%;
}

body {
	display: flex;
	height: 100%;
	flex-flow: row wrap;
	align-items: center;
	justify-content: center;
}

.box {
	width: 200px;
	height: 100px;
}
```

```html
<div class="box" id="box"></div>
<script>
	var bodyWidth = document.documentElement.clientWidth || document.body.clientWidth;
	var bodyHeight = document.documentElement.clientHieght || document.body.clientHieght;

	var boxW = box.offsetWidth;
	var boxH = box.offsetHeight;
	box.style.left = (winW - boxW) / 2 + 'px';
	box.style.top = (winH - boxH) / 2 + 'px';
</script>
```

**`clientTop 和 clientLeft`**

> 只有top和left，没有right和bottom
> clientTop: 盒子上边框高度
> clientLeft: 盒子做边框宽度
> 获取的结果其实就是border-width的值
> 
> 通过JS盒子模型属性获取的值结果是不带单位的，而只能是整数（它会自动四舍五入）

**`offsetWidth 和 offsetHeight`**
> 在clientWidth和clientHeight的基础加上盒子的边框即可
> 
> 和内容是否溢出没关系
> 
> 项目中，如果获取一个盒子的宽度和高度，我们一般用offsetWidth（而不是clientWidth），border也算是当前盒子的一部分

**`scrollWidth 和 scrollHeight`**

> [没有内容溢出] 获取的结果和clientWidth/clientHieght一样
> 
> [有内容溢出] 真实内容的宽度和高度包含溢出内容，在加上左padding或者上padding的值
> 
> 有内容溢出的情况下，我们通过scrollWidth或者scrollHeight获取的结果是一个约等于的值
> 1) 有内容溢出，每个浏览器由于对行高和文字的渲染不一样，返回的结果也是不一样的
> 2) 是否设置overflow:hidden;对最后获取的结果也有影响

```javascript
//=> 获取页面的真实高度(包含溢出的内容)
document.documentElement.scrollHeight || document.body.scrollHieght
```

`curEle.style.xxx`

> 获取当前元素所有写在行内样式上的样式值
> 
> 特殊：只有把样式写在行内样式上，才可以同股票这种办法获取到（写在其他地方的样式是获取不到的）
> 
> 这种办法在真实项目中使用的特别少：因为我们不可能把所有元素的样式都写在行内（这种方式不常用）

**`window.getComputedStyle / currentStyle`**

> 只要当前元素在页面中显示出来，我们就可以获取其样式值（不管写在行内还是样式表中）；也就是获取所有经过浏览器计算过的样式（当前元素只要能在页面中展示，那么它的所有样式都是经过浏览器计算的，包含一些你没有设置，浏览器按照默认样式渲染的样式）
> 
> window.getComputedStyle: 适用于标准浏览器，但在IE6~8下不兼容，IE6~8下我们使用curEle.currentStyle来获取需要的样式值

```javascript
//=> 通过getComputedStyle获取的结果是一个对象，对象中包含了所有被浏览器计算过的样式属性和属性值
// window.getComputedStyle([当前需要操作的元素], [当前元素的伪类，一般写null])

// window.getComputedStyle(box, null).paddingLeft; 这种方式可以在获取的对象中找到某一具体样式的属性值（通过['paddingLeft']也可以）

//=> 在IE6~8浏览器的window全局对象中，没有提供getComputedStyle这个属性方法，我们只能使用currentStyle属性来获取元素的样式

// [当前元素].currentStyle 获取的结果也是一个包含当前元素所有样式属性和值的对象

// box.currentStyle.paddingLeft
```

```javascript
/*
 * getCss: 获取当前元素某一个样式属性值
 * @parameter
 *     curEle: 当前需要操作的元素
 *     attr: 当前需要获取的样式属性名
 * @return
 *     获取的样式属性值    
*/

//=> 给元素设置透明度，需要写两套（兼容IE6~8）opacity: n;filter:alpha(opacity:n*100);
//=> 在JS中如果沃恩想要获取元素的透明度，我们一般都会这样执行getCss(box, 'opacity');
//=> 在IE6~8下不识别opacity

function getCss(curEle, attr){
	var value = null,
		reg = null;
	if('getComputedStyle' in window){
		value = window.getComputedStyle(curEle, null)[attr];
	}else{
		//=> IE6~8下处理透明度
		//=> 正则捕获 /\d+(\.\d+)?/ 或 /^alpha\(opacity=(.+)\)$/
		if(attr === 'opacity') {
			value = curEle.currentStyle[filter];
			var reg = /^alpha\(opacity=(.+)\)$/;
			reg.test(value) ? value = reg.exec(value)[1] / 100 : value = 1;
		}else{
			value = curEle.currentStyle[attr];
		}
	}
	
	//=> 首先把获取到的结果转换为数字，看是否是NaN，不是NaN说明可以去除单位，我们把转换的结果返回即可，如果是NaN，说明当前获取的结果不是数字+单位，此时直接返回获取到的值即可
	
	//=> 去除单位（复合值不去单位例如 padding:20px 30px 40px 50px;）
	reg = /^-?\d+(\.\d+)?(px|pt|rem|em)?$/i;
	reg.test(value) ? value = parseFloat(value) : null;

	return value;
}
```

```javascript
//=> 设置元素的样式
// curEle.style.xxx = xxx; 设置当前元素的行内样式（JS中设置的样式一般都要设置在行内样式上：行内样式上优先级最大）
// curEle.className = xxx; 设置元素的样式类名

function setCss(curEle, attr, value){
	//=> 如果传递的value值没有单位，我们根据需求自动补充单位
	//=> 需要补充单位的常用样式属性：width/height/margin/padding/top/left/right/bottom
	//=> 2、并不是所有的样式属性传递进来的值都要补充单位
	//=> 2、传递的值已经存在单位，我们不需要再补充，没有单位我们再补充
	var reg = /^(width|height|((margin|padding)?(top|left|right|bottom)?)|borderWidth)$/i;
	if(reg.test(value)){
		if(!isNaN(value)){
			value =+ 'px';
		}
	}
	curEle['style'][attr] = value;
}
```

```javascript
//=> 反思路验证：传递的是纯数字，大部分都需要家单位，但是某些特殊的样式属性是不需要加单位的
//=> 不需要加单位：zIndex、zoom、lineHeight
//=> 如果传递的是opacity，我们需要兼容处理

function setCss(curEle, attr, value){
	
	if(attr === 'opacity'){
		curEle.style.opacity = value;
		curEle.style.filter = 'alpha(opacity='+ value * 100 +')';
		return;
	}
	isNaN(value) && !/^(zIindex|zoom|lineHeight)$/i.test(value) ? value += 'px' : null;
	
	curEle['style'][attr] = value;
}

//=> 批量设置样式
function setGroupCss(curEle, options){
	if(Object.prototype.toString.call(options) === '[object Object]'){
		return;
	}
	
	for(var key in options){
		if(options.hasOwnProperty(attr)){
			setCss(curEle, attr, options[attr]);
		}
	}
}
```

```javascript
//=> 封装css方法，实现设置，批量设置获取元素
var utils = function(){
	//=> 获取元素的css属性
	var getCss = function(curEle, attr){
		var valur = null,
			reg = null;
		if (window.getComputedStyle){
			value = window.getComputedStyle(curEle, null);
		} else {
			if(attr = 'opacity'){
				value = curEle.currentStyle['filter'];
				reg = /^alpha\(opacity=(.+)\)$/i;
				value = reg.test(value) ? reg.exec(value)[1] / 100 : 1;
			} else {
				value = curEle.currentStyle[attr];
			} 
		}
		var reg = /^-?\d+(\.\d+)?(px|pt|rem|em)?$/i;
		reg.test(value) ? value = parseFloat(value) : null;
		return value;
	};
	
	//=> 设置元素的css属性
	var setCss = function(curEle, attr, value){
		if(attr === 'opacity'){
			curEle['style']['opacity'] = value;
			curEle['style']['filter'] = 'alpha(opacity='+ value * 100 +')';
			return;
		}
		isNaN(value) && !/^(zIndex|zoom|lineHeight|fontWeight)$/i.test(attr) ? value += 'px' : null;
		curEle['style'][attr] = value;
	};
	
	//=> 批量设置元素css属性
	var setGroupCss = function(curEle, options){
		if(Object.prototype.toString.call(options) === '[object Oject]'){
			reutrn;
		}
	
		for(var attr in options){
			if(options.hasOwnProperty(attr)){
				setCss(curEle, attr, options[attr]);
			}
		}
	};
	
	//=> 集成获取样式，设置样式
	var css = function(){
		var len = arguments.length;
		if(len >= 3){
			//=> setting styles
			setCss.apply(this, arguments)
			return;
		};
	
		if(len === 2 && Object.prototype.toString.call(arguments[1]) === '[object Object]'){
			setGroupCss.apply(this, arguments);
			return;
		};
	
		return getCss.apply(this, arguments);
	};

	//=> css优化版本
	var css = function(){
		var len = arguments.length,
			type = Object.prototype.toString.call(arguments[1]),
			fn = getCss;
		len >= 3 ? fn = setCss : len === 2 && type === '[object Object]' ? fn = setGroupCss : null;
	
		return fn.apply(this, arguments);
	};
	
	return {
		css: css
	}
	
}();
```

**`offsetParent: 父级参照物`**

> 父级参照物不等价于父级元素：父级参照物和它的父级元素没有直接关系
> 父级参照物：同一个平面中最外层的容器是所有盒子的父级参照物
> 
> 默认情况下，一个页面中所有元素的父级参照物都是body（而body的父级参照物是null）
> 
> 当我们给元素设置定位之后会改变元素的参照物（因为设置定位会让元素脱离文档流 ）

**`offsetLeft offsetTop`**

> 当前元素的外边框 距离 父级参照物的内边框 的偏移量（左偏移、上偏移）
> 
> 标准IE8浏览器有特殊性：它的偏移量是从当前元素的外边框到父级参照物的外边框

```javascript
//=> 封装获取任意元素到body的距离
//=> 思路：首先获取自己的偏移量和自己的父级参照物，如果自己的父级参照物不是body，我们加上父级参照物的边框和偏移量...直到某一次找到的父级参照物是body为止


var utils = function(){
	//=> 获取当前元素距离body的偏移量，包括左偏移和上便宜
	var offset = function(curEle){
		var l = curEle.offsetLeft,
			t = curEle.offsetTop,
			p = curEle.offsetParent;
		while(p.tagName !== 'BODY'){
			//=> 兼容IE8
			if(!/MSIE 8/i.test(navigator.userAgent)){
				l += p.clientLeft;
				t += p.clientTop;
			}
	
			l += p.offsetLeft;
			t += p.offsetTop;
			p = curEle.offsetParent;
		}
	
		return {left:l, top: t}
	};
	
	return {
		offset: offset
	}
}();
```

**`scrollLeft scrollTop`**

> scrollLeft: 横向滚动条卷去的宽度
> 
> scrollTop: 竖向滚动条卷去的高度
> 
> 最小值: 0
> 最大值: scrollHeight - clientHeight 页面真实高度减去一屏幕高度

**前面JS盒子模型属性都是`只读属性`：只能通过属性获取值，不可以修改属性值（修改不生效），但`scrollLeft scrollTop`是可读写属性，可以设置和修改**

```javascript
document.documentElement.scrollTop || document.body.scrollTop;

document.documentElement.scrollLeft || document.body.scrollLeft;
```

```javascript
//=> 操作所有关于浏览器盒子模型属性，处理兼容
var utils = function(){
	var winBox = function(attr, value){
		if(typeof value !== 'undefined'){
			document.documentElement[attr] = value;
			document.body[attr] = value;
			return;
		}
		return document.documentElement[attr] || document.body[attr];
	};
}();
```

回到顶部小案例：[https://codepen.io/smileyby/pen/aaoQxm](https://codepen.io/smileyby/pen/aaoQxm)



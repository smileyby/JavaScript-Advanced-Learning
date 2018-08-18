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


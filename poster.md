# svg实战 - 绘制海报
因为在电商公司，经常做一些h5的活动，需要实现分享海报功能。海报上会有一些个人定制信息，比如得分、评语等，和背景图合成在一起，做成一张图用于分享或下载。因为每个人字数不一样，让内容绘制在合适的位置，就需要一些“手段”。

<figure>
  <img src="https://inagora.github.io/svg-guide/res/poster.jpg" style="max-width:320px">
  <figcaption>效果图</figcaption>
</figure>
如上图中，用户名、已兑换金额是用户当前活动数据，每个人不一样，字符串长度也就不一样。但需要让它们居中显示。我们经常会用到三种方案，各有利弊：
1. 直接使用canvas绘制。需要使用measureText函数获得文本显示的宽度，然后计算它应该绘制的位置；
2. 使用dom布局，配合canvas绘制，让浏览器的布局能力“帮忙”获得字符的合适位置；
3. 使用svg布局，直接生成图片。

## 方案一：纯canvas绘制
直接用canvas来合成这张海报，是我们最熟悉的方案。大致步骤如下：
1. 扣出不可变的背景图，见下面。
2. 使用measureText获得要绘制内容的宽度，然后计算应该绘制的位置，通过fillText绘制到画布上
3. 绘制完所有内容后，使用toDataURL或者toBlob接口，获得图片内容，上传到服务器或者传给接口。

<figure>
  <img src="https://inagora.github.io/svg-guide/res/poster-bg.jpg" style="max-width:320px">
  <figcaption>背景图</figcaption>
</figure>

核心代码如下
``` javascript
async function drawPoster(info){
	// ... 一些准备代码

	// 1. 获得“已兑换”字符串的长度。注意，需要先设置字体。另外，“元现金”的宽度我们认为和它一样
	ctx.font = '42px "Kaiti SC"';
	let textWidth = ctx.measureText('已兑换').width;
	let spaceWidth = 40;
	// 2. 如法获得金额的宽度
	ctx.font = '140px "Kaiti SC"';
	let moneyWidth = ctx.measureText(info.money).width;
	// 3. 文案的总宽度。两段普通文案，两个普通文案和金额之间的空白，加金额的宽度
	let totalWidth = textWidth*2+spaceWidth*2+moneyWidth;
	// 4. 绘制前后两段普通文案
	ctx.textAlign = 'start';
	ctx.textBaseline = 'alphabetic';
	ctx.font = '42px "Kaiti SC"';
	ctx.fillText('已兑换', (posterWidth-totalWidth)/2, 390);
	ctx.fillText('元现金', posterWidth-(posterWidth-totalWidth)/2-textWidth, 390);
	// 5. 绘制金额
	ctx.font = '140px "Kaiti SC"';
	ctx.fillStyle='#a70322';
	ctx.fillText(info.money, (posterWidth-moneyWidth)/2, 400);

	// ... 其它处理代码
}
```
[demo地址](https://inagora.github.io/svg-guide/res/poster-canvas.html) [查看完整源码](https://github.com/inagora/svg-guide/blob/gh-pages/res/poster-canvas.html)

上面代码片断绘制了“已兑换1.50元现金”的文案。为了让这段文字居中，就需要用measureText获得每一段文本的长度，然后根据总宽度，小心计算它们应该的渲染位置。例子中的文字比较少，如果多了；或者样式复杂的时候，使用canvas直接绘制，会让代码变得很臃肿。而且，不方便调试，因为不能直接在开发者工具里修改即所见。

下面用一种“hack”方式，把布局工具交给浏览器。
## 方案二：使用浏览器布局
浏览器里可以方便的用css控制内容的布局，对于居中我们有的是办法。那我们索性把这个任务交给它，然后我们获得每个文字的位置，直接绘制在画布中就行了。比如上例中，我们先用html和css把内容放好。
``` html
<style>
.poster{
	position: absolute;
	width: 721px;
	height: 920px;
	left: -10000px;
	border: 1px solid #000;
	font-family: "Kaiti SC";
	text-align: center;
}
.uname{
	font-size: 36px;
	padding-top:150px;
}
/* 其它样式内容见源码 */
</style>
<div class="poster">
	<div class="uname"></div>
	<div class="money"></div>
</div>
```
然后使用js填充内容。
``` javascript
let moneyHtml = '已兑换'.split('').map(word=>`<span>${word}</span>`).join('');
moneyHtml += info.money.split('').map(word=>`<strong>${word}</strong>`).join('');
moneyHtml += '元现金'.split('').map(word=>`<span>${word}</span>`).join('');
document.querySelector('.money').innerHTML = moneyHtml;
```
注意，上面把文本都用`span`和`strong`标签分开包裹起来，目的是方便接下来用js单独获得每个字符的位置，直接渲染。

然后就是把dom中的内容，“复制”到canvas上。
``` javascript
//绘制金额
ctx.font = '42px "Kaiti SC"';
for(let word of document.querySelectorAll('.money span')){
	let rect = word.getBoundingClientRect();
	ctx.fillText(word.innerHTML, rect.left-left, rect.top-top);
}
ctx.font = '140px "Kaiti SC"';
ctx.fillStyle='#a70322';
for(let word of document.querySelectorAll('.money strong')){
	let rect = word.getBoundingClientRect();
	ctx.fillText(word.innerHTML, rect.left-left, rect.top-top);
}
```
它获取每个字的dom元素，然后挨个儿绘制到canvas上。
> 这种方案特别适合大量文字的情况，尤其是可以方便解决换行问题。因为canvas里的文本不会自动换行，要自己算在哪里换行，太麻烦了。

[demo地址](https://inagora.github.io/svg-guide/res/poster-dom.html) [查看完整源码](https://github.com/inagora/svg-guide/blob/gh-pages/res/poster-dom.html)

## 方案三：svg绘制
既然svg就是图片，让使用它的结构化和css来方便布局，然后直接把它当图drawImage行不行？

不行，因为canvas的drawImage接口中，只支持CSSImageValue，HTMLImageElement，SVGImageElement，HTMLVideoElement，HTMLCanvasElement，ImageBitmap或者OffscreenCanvas。这里面虽然有SVGImageElement，但它不是svg本身，只是svg里使用<image>元素。

但是，<img>支持base64，svg的内容正好可以转化为base64，一下子就可以曲线救国了。

先用svg布局海报内容，因为svg即支持dom，也支持css，甚至js，用起来实在太方便。
``` html
<svg id="poster" viewBox="0 0 721 920">
	<style type="text/css">
		.uname{
			dominant-baseline: middle;
			text-anchor: middle;
			fill: #282521;
			font: 36px 'Kaiti SC';
		}
		.money{
			text-anchor: middle;
			fill: #282521;
			font: 42px 'Kaiti SC';
		}
		.money-count{
			font-size: 140px;
			fill: rgb(167, 3, 34);
		}
	</style>
	<!-- <image x="0" y="0" width="721" height="920" href="https://inagora.github.io/svg-guide/res/poster-bg.jpg" /> -->
	<text x="360" y="165" class="uname">poker</text>
	<text x="360" y="400" class="money">
		<tspan dominant-baseline="ideographic">已兑换</tspan>
		<tspan dx="40" class="money-count">1.50</tspan>
		<tspan dx="40" dominant-baseline="ideographic">元现金</tspan>
	</text>
</svg>
```
其中注释掉的是海报背景图，在调试时可以打开，就可以直接在浏览器看成品效果了。注意，上面代码里把名字和金额直接写进去了，方便理解，正式环境中需要js写进去。

把svg内容转化成base64代码如下：
``` javascript
function loadImg(url){
	return new Promise((resolve) => {
		let img = new Image();
		img.setAttribute('crossOrigin', 'Anonymous');
		img.onload = function () {
			resolve(this);
		};
		if(url instanceof window.SVGSVGElement){
			var xml = new XMLSerializer().serializeToString(document.querySelector('#poster'));
			img.src = 'data:image/svg+xml;base64,'+window.btoa(unescape(encodeURIComponent(xml)));
		} else
			img.src = url;
	});
}
```
它创建了一个Image元素，然后把svg的内容转成base64代码，用图片展示出来。

然后直接把这个图绘制在canvas上就行了。
``` javascript
let svgImg = await loadImg(document.querySelector('#poster'));
ctx.drawImage(svgImg, 0, 0);
```

> 这种方案的好处是即使用svg方便的布局和调试，然后渲染的代码又很简单。

当然，相比于第二种方案，也有一些缺点：

1. 自动换行需要额外处理； 
2. svg内联的图片、字体文件等，这些外联的文件，都不会加载，需要转成base64直接写到svg里

## 总结
上面三种方案，各有利弊，svg简单、代码也行，很“优雅”；dom方案可以完成使用浏览器的布局能力，完全不需要计算，对于特别多文案或内容很复杂的时候，这个方案不错；纯canvas方案的兼容性最好，也不需要额外的dom搀和，“最干净”。
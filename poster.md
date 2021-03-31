# svg实战 - 绘制海报
因为在电商公司，经常做一些h5的活动，需要实现分享海报功能。海报上会有一些个人定制信息，比如得分、价格、评语等，和背景图合成在一起，做成一张图。因为每个人字数不一样，让内容绘制在合适的位置，就需要一些“手段”。

<img src="https://inagora.github.io/svg-guide/res/poster.jpg" style="max-width:320px">

如上图中，用户名、已兑换积分、可抵现这三句话，是用户当前活动数据，每个人不一样。字符串长度就不一样，要让它们居中显示。

本文介绍三种方式完成海报的绘制，各有利弊：
1. 直接使用canvas绘制。使用measureText函数计算文本应该显示的宽度，然后计算它应该绘制的位置；
2. 使用dom布局，配合canvas绘制，让浏览的布局能力自动完成计算；
3. 使用svg布局，直接生成图片。

## canvas绘制海报
直接用canvas来合成这张海报，是我们最熟悉也最成熟的方案。大致步骤如下：
1. 扣出不可变的背景图，如下：
<img src="https://inagora.github.io/svg-guide/res/poster-bg.jpg" style="max-width:320px">
2. 使用measureText获得要绘制内容的宽度，然后计算应该绘制的位置，通过fillText绘制到画布上
3. 绘制完所有内容后，使用toDataURL或者toBlob接口，获得图片内容，上传到服务器或者传给接口。

核心代码如下
``` javascript
async function drawPoster(info){
	let canvas = document.getElementById("canvas");
	canvas.width = 1080;
	canvas.height = 1040;
	let ctx = canvas.getContext("2d");

	let bg = await loadImg('/svg-guide/res/poster-bg.jpg');
	ctx.drawImage(bg, 0, 0);
	
	//绘制名字
	ctx.font = '12px 宋体';
	let nameWidth = ctx.measureText(info.name);
	let x = (1080 - nameWith)/2
	ctx.fillText(info.name, x, 150);

	//同样绘制积分和抵现金额。其中积分中各处字体大小不一样，等分开测量了

	//最后输出内容就行了
	return canvas.toDataURL("image/jpeg", 0.9);
}
```
因为代码太多，就不一一贴出了，源码请查看[html](html)。

从代码可以看出，绘制时，不同颜色、不同字体、不同字号的地方，都需要单独计算，每一处位置都要算的明明白白的才行，着实有些费劲。

既然计算的部分占用了大量代码，我们就想办法把“计算”工作交出去。

## 使用浏览器布局
浏览器里可以方便的用css控制内容的居中，我们利用浏览器帮我们布局，然后我们获得布局后文字的位置，直接绘制在画布中就行了。比如上例中，我们先用html和css把内容放好。
``` html
<style>
.poster{
	position: abs
}
</style>
<div class="poster">
</div>
```
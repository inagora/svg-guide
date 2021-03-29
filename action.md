# svg实战
## 图片压缩
因为svg的在html中可编程性质，图片压缩在svg上的想象空间很大。
* 高清矢量图。比如logo，icon等几何图形或者由几何图形组合的复杂结构，用svg代码文本来做，会比jpg或者png小很多。比如微软搜索的logo，https://cn.bing.com/
* 动画。能用css实现的动画，尽量用css来做，需要的代码会少很多；但经常有一些css做不到的，如路径动画。如果用gif或者video，资源就大很多了。这时候就适合用svg来做；
* 使用svg做“裁剪路径”，与jpeg配合实现“镂空”效果。大部分情况下，Jpg的资源大小会比png小很多。

## 需求
活动需求，开发一个如下图的功能

<img src="https://inagora.github.io/svg-guide/res/page.jpg" style="max-width:320px">

因为可以获得不同的字，弹窗就是一个可复用的组件。把通用的UI抽出来，就是

<img src="https://inagora.github.io/svg-guide/res/bg.png" style="max-width:320px">

这个图片的格式是PNG，大小84K。使用PNG是因为它的四个角需要镂空。

## 优化第一波：mask-image
上面的png图比较大，换成jpg就小很多了。

<img src="https://inagora.github.io/svg-guide/res/bg.jpg" style="max-width:320px">

jpg格式，大小29K，只有Png的34%。不过它不支持透明，四个角没法做到镂空的效果。

这时候就可以使用css属性[mask-image](https://developer.mozilla.org/zh-CN/docs/Web/CSS/mask-image)了，这个属性用于设置元素上遮罩层的图像。使用遮罩，把不需要的地方裁切掉。遮罩图如下：

<img src="https://inagora.github.io/svg-guide/res/mask.png" style="max-width:320px">

它的格式是png，四个角的透明度为0。

代码如下：
``` css
.dia{
	background: url(./bg.jpg);
	background-size: 100%;
	-webkit-mask-image: url(./mask.png);
	-webkit-mask-size: 100%;
	mask-image: url(./mask.png);
	mask-size: 100%;
}
```
这种解决方案，需要的资源总量为 29+3=32k，是原始方案的 `38%`。

### 优化第二波：svg介入
svg既然是“图片”，那就可以替代做一些图片的工作。比如上面的mask图，是一个特别简单的几何结构，可以直接用svg替代。

`mask.svg`文件内容如下：
``` html
<svg viewBox="0 0 566 700">
	<path id="baseShape" fill="red" d="M25 0h516a25,25 0 0,0 25,25v650a25,25 0 0,0 -25,25h-516a25,25 0 0,0 -25,-25v-650a25,25 0 0,0 25,-25Z"></path>
</svg>
```

然后调整上面css文件的内容：
``` css
.dia{
	background: url(./bg.jpg);
	background-size: 100%;
	-webkit-mask-image: url(./mask.svg);
	-webkit-mask-size: 100%;
	mask-image: url(./mask.svg);
	mask-size: 100%;
}
```
一通操作后，就不需要mask.png这个图片了，增加了一个100多字节的svg文件，资源大小减少了。

### 优化第三波：svg的直接绘制
上面即然已经用了svg，那干嘛不直接用svg画这个弹窗背景，裁剪啥的都是可以不使用的。

<img src="https://inagora.github.io/svg-guide/res/bg.png" style="max-width:240px">

看这个图，它的边缘，可以直接用clipPath来裁剪；边缘靠里的一圈线，正好和边缘的形状一致，路径参数还能复用。最后只需要图片中间方框和云，使用图片来做就行。（当然中间一块儿也是可以用svg做的，只是有些复杂）。

``` html
<svg viewBox="0 0 566 700" xmlns="http://www.w3.org/2000/svg">
	<defs>
		<path id="baseShape" d="M25 0h516a25,25 0 0,0 25,25v650a25,25 0 0,0 -25,25h-516a25,25 0 0,0 -25,-25v-650a25,25 0 0,0 25,-25Z"></path>
	</defs>
	<use id="bg" href="#baseShape" fill="#db3e4a" />
	<use id="line" href="#baseShape" stroke="#e09b6c" fill="transparent" stroke-width="1" x="11" y="11" style="transform:scale(0.96,0.97);" />
	<image id="card" x="95" y="78" href="./card.jpg" height="382" width="420"></image>
</svg>
```
上面图用path定义了背景图和线圈，因为它们形状一致，就复用了一下。然后在上面盖上了最小尺寸的图。
这个方式下，最后的资源总量为 21k。
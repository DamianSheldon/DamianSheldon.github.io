---
layout: post
title: "Web 面试题汇总(二)"
date: 2018-09-05 15:52:44 +0800
comments: true
categories: [Archives, Web Development]
keywords: css, web, interview
description: Web 面试题汇总
---

###1.介绍一下标准的CSS的盒子模型？低版本IE的盒子模型有什么不同的？
A:每个元素被表示为一个矩形的方框，框的内容、内边距、边界和外边距像洋葱的膜那样，一层包着一层构建起来。

低版本IE的盒子模型的宽度包括框的内容、内边距和边界。

###2.CSS选择符有哪些？哪些属性可以继承？
A:

* *简单选择器*（Simple selectors）：通过元素类型、class 或 id 匹配一个或多个元素。
* *属性选择器*（Attribute selectors）：通过 属性 / 属性值 匹配一个或多个元素。
* *伪类*（Pseudo-classes）：匹配处于确定状态的一个或多个元素，比如被鼠标指针悬停的元素，或当前被选中或未选中的复选框，或元素是DOM树中一父节点的第一个子节点。
* *伪元素*（Pseudo-elements）:匹配处于相关的确定位置的一个或多个元素，例如每个段落的第一个字，或者某个元素之前生成的内容。 
* *组合器*（Combinators）：这里不仅仅是选择器本身，还有以有效的方式组合两个或更多的选择器用于非常特定的选择的方法。例如，你可以只选择divs的直系子节点的段落，或者直接跟在headings后面的段落。
* *多重选择器*（Multiple selectors）：这些也不是单独的选择器；这个思路是将以逗号分隔开的多个选择器放在一个CSS规则下面， 以将一组声明应用于由这些选择器选择的所有元素。

哪些属性可以继承？

* 所有元素可继承：visibility和cursor。
* 内联元素可继承：letter-spacing、word-spacing、white-space、line-height、color、font、font-family、font-size、font-style、font-variant、font-weight、text-decoration、text-transform、direction。
* 终端块状元素可继承：text-indent和text-align。
* 列表元素可继承：list-style、list-style-type、list-style-position、list-style-image。
* 表格元素可继承：border-collapse。

###3.CSS优先级算法如何计算？
A:CSS根据规则的重要性、专用性和源代码次序来计算优先级。

重要性和源代码次序相对简单，专用性是用四种不同的值（或组件）来衡量的，它们可以被认为是千位，百位，十位和个位:

1.	千位：如果声明是在style 属性中该列加1分（这样的声明没有选择器，所以它们的专用性总是1000。）否则为0。
2.	百位：在整个选择器中每包含一个ID选择器就在该列中加1分。
3.	十位：在整个选择器中每包含一个类选择器、属性选择器、或者伪类就在该列中加1分。
4.	个位：在整个选择器中每包含一个元素选择器或伪元素就在该列中加1分。
<!--more-->
###4.如何居中div？
A: 

* 水平居中
* 水平垂直居中

####水平居中
给div设置一个宽度，然后添加margin:0 auto属性。

{% codeblock html %}

	div{
	 	width:200px;
	 	margin:0 auto;
	}

{% endcodeblock %}

####水平垂直居中

* 容器的宽高确定

{% codeblock css %}
	
	// 确定容器的宽高 宽500 高 300 的层
	// 设置层的外边距
	
	div {
	 	position: relative;		/* 相对定位或绝对定位均可 */
	 	width:500px;
	 	height:300px;
	 	top: 50%;
	 	left: 50%;
	 	margin: -150px 0 0 -250px;     		/* 外边距为自身宽高的一半 */
	 	background-color: pink;	 		/* 方便看效果 */
	 }
{% endcodeblock %}

* 容器的宽高未知

{% codeblock css %}

	 // 未知容器的宽高，利用 `transform` 属性
	
	div {
	 	position: absolute;		/* 相对定位或绝对定位均可 */
	 	width:500px;
	 	height:300px;
	 	top: 50%;
	 	left: 50%;
	 	transform: translate(-50%, -50%);
	 	background-color: pink;	 		/* 方便看效果 */
 	}
 	
{% endcodeblock %}

* 利用 flex 布局

{% codeblock css %}

	 // 利用 flex 布局
	 // 实际使用时应考虑兼容性
	
	.container {
	 	display: flex;
	 	align-items: center; 			/* 垂直居中 */
	 	justify-content: center;		/* 水平居中 */
	
	 }
	.container div {
	 	width: 100px;
	 	height: 100px;
	 	background-color: pink;			/* 方便看效果 */
	 }  

{% endcodeblock %}

###Further Reading
[Centering in CSS: A Complete Guide](https://css-tricks.com/centering-css-complete-guide/)  


###5.display有哪些值？说明他们的作用。
A:display 所有可能的值太多了，这里说下常见的:

| display | 作用 |
| :------- | :--- |
| none | 元素不显示，并从文档流中移除。 |
| inline | 行内元素类型。默认宽度为内容宽度，不可设置宽高，同行显示。 |
| inline-block | 默认宽度为内容宽度，可以设置宽高，同行显示。 |
| block | 块类型。默认宽度为父元素宽度，可设置宽高，换行显示。 |
| list-item | 象块类型元素一样显示，并添加样式列表标记。 |
| table | 元素会作为块级表格来显示。 |

###6.position的值? relative和absolute定位原点是？
A: position  = static | relative | absolute | sticky | fixed

relative: The element is positioned according to the normal flow of the document, and then offset relative to itself based on the values of top, right, bottom, and left.

absolute: The element is removed from the normal document flow, and no space is created for the element in the page layout. It is positioned relative to its closest positioned ancestor, if any; otherwise, it is placed relative to the initial containing block. Its final position is determined by the values of top, right, bottom, and left.

###7.请解释一下CSS3的Flexbox（弹性盒布局模型），以及适用场景？
A:Flexbox is a one-dimensional layout method for laying out items in rows or columns. Items flex to fill additional space and shrink to fit into smaller spaces. 

适用场景:

* Vertically centering a block of content inside its parent.
* Making all the children of a container take up an equal amount of the available width/height, regardless of how much width/height is available.
* Making all columns in a multiple column layout adopt the same height even if they contain a different amount of content.

###8.用纯CSS创建一个三角形的原理是什么？
A:采用的是均分原理。

{% codeblock %}

	.triangle{  
	     width:0;  
	     height:0;  
	     margin:0 auto;  
	     border:6px solid transparent;  
	     border-top: 6px solid red;  
	 }
	   
	// 第二种写法：
	
	.triangle{  
	     width:0;  
	     height:0;  
	     margin:0 auto;  
	     border-width:6px;
	     border-color:red transparent transparent transparent;
	     border-style:solid dashed dashed dashed;//为了兼容IE6，把没有的边框都设置为虚线
	 }
{% endcodeblock %}

###9.一个满屏 品 字布局 如何设计?
A:方式一：上面的 div 宽100%，下面的两个 div 分别宽50%，然后用 float 或者 inline 使其不换行即可；方式二：上面的 div 宽100%，内含一个 div 水平居中，下面的div，内含两个 div,使用 absolute 定位。

{% codeblock %}

	// 方式一
	<!DOCTYPE html>
	<html>
	<head>
	    <meta charset="utf-8">
	    <title>满屏品字布局</title>
	    <style type="text/css">
	        *{
	            margin: 0;
	            padding: 0;
	        }
	
	        html,body{
	            height: 100%;/*此设置非常关键，因为默认的body，HTML高度为0，所以后面设置的div的高度无法用百分比显示*/
	        }       
	
	        .header{
	            height:50%; /*此步结合html,body高度为100%，解决元素相对窗口的定位问题*/
	            width: 50%;     
	            background: #ccc;           
	            margin:0 auto;
	        }
	
	        .main{
	            width: 100%;
	            height: 50%;
	            background: #ddd;
	        }
	
	        .main .left,.main .right{
	            float: left;/*采用float方式，对元素进行左右定位*/
	            width:50%;/*此步解决元素相对窗口的定位问题*/
	            height:100%;/*此步解决元素相对窗口的定位问题*/
	            background: yellow;
	        }
	
	        .main .right{
	            background: green;
	        }
	    </style>
	</head>
	<body>
	<div class="header"></div>
	<div class="main">
	    <div class="left"></div>
	    <div class="right"></div>
	</div>
	</body>
	</html>
{% endcodeblock %}

{% codeblock %}

	<!DOCTYPE html>
	<html>
	<head>
	<meta charset="utf-8">
	    <title>满屏品字布局</title>
	    <style type="text/css">
	        body{  
	          height: 1200px;  
	      }  
	      .main {  
	          position: fixed;  /*此步解决元素相对窗口的定位问题*/
	          left: 0;  
	          top: 0;  
	          height: 100%;  
	          width: 100%;  
	      }  
	      .wrapper-up {  
	          height: 50%;  
	      }  
	
	      .wrapper-down {  
	          height: 50%;  
	          position: relative;  
	      }  
	      .div-square-up {  
	          width: 50%;  
	          margin: 0 auto;  
	          border: 2px solid red;  
	          height: 96%;  
	          box-sizing: border-box;  
	      }  
	
	      .div-square-left {  
	          position: absolute;  /*此步解决元素左右定位问题*/
	          left: 0;  
	          width: 48%;  
	          height: 100%;  
	          box-sizing: border-box;  
	          border: 2px solid red;  
	      }  
	
	      .div-square-right {  
	          position: absolute;   /*此步解决元素左右定位问题*/
	          right: 0;  
	          width: 48%;  
	          height: 100%;  
	          border: 2px solid red;  
	          box-sizing: border-box;  
	      }  
	    </style>
	</head>
	<body>
	    <div class="main">  
	         <div class="wrapper-up">  
	           <div class="div-square-up"></div>  
	         </div>  
	         <div class="wrapper-down">  
	           <div class="div-square-left"></div>  
	           <div class="div-square-right"></div>  
	         </div>  
	    </div> 
	</body>
	</html>
{% endcodeblock %}

###10.css多列等高如何实现？
A:利用padding-bottom|margin-bottom正负值相抵；
设置父容器设置超出隐藏（overflow:hidden），这样子父容器的高度就还是它里面的列没有设定padding-bottom时的高度，当它里面的任 一列高度增加了，则父容器的高度被撑到里面最高那列的高度，其他比这列矮的列会用它们的padding-bottom补偿这部分高度差。


###11.经常遇到的浏览器的兼容性有哪些？原因，解决方法是什么，常用hack的技巧？
A:
经常遇到的浏览器的兼容性有哪些？原因，解决方法是什么?

* png24位的图片在iE6浏览器上出现背景，解决方案是做成PNG8.
* 浏览器默认的margin和padding不同。解决方案是加一个全局的`*{margin:0;padding:0;}`来统一。
* IE6双边距bug:块属性标签float后，又有横行的margin情况下，在ie6显示margin比设置的大。浮动ie产生的双倍距离 `#box{ float:left; width:10px; margin:0 0 0 100px;}`，这种情况之下IE会产生20px的距离，解决方案是在float的标签样式控制中加入 ——`_display:inline;`将其转化为行内属性。利用 `_` 这个符号只有ie6会识别的渐进识别的方式，从总体中逐渐排除局部。
* IE下,可以使用获取常规属性的方法来获取自定义属性,也可以使用getAttribute()获取自定义属性;Firefox下,只能使用getAttribute()获取自定义属性。解决方法:统一通过getAttribute()获取自定义属性。
* IE下,even对象有x,y属性,但是没有pageX,pageY属性;Firefox下,event对象有pageX,pageY属性,但是没有x,y属性。解决方法：（条件注释）缺点是在IE浏览器下可能会增加额外的HTTP请求数。
* 超链接访问过后hover样式就不出现了 被点击访问过的超链接样式不在具有hover和active了解决方法是改变CSS属性的排列顺序:L-V-H-A :  `a:link {} a:visited {} a:hover {} a:active {}`

常用hack的技巧？

由于不同厂商的流览器或某浏览器的不同版本（如IE6-IE11,Firefox/Safari/Opera/Chrome等），对CSS的支持、解析不一样，导致在不同浏览器的环境中呈现出不一致的页面展现效果。这时，我们为了获得统一的页面效果，就需要针对不同的浏览器或不同版本写特定的CSS样式，我们把这个针对不同的浏览器/不同版本写相应的CSS code的过程，叫做CSS hack!

CSS Hack大致有3种表现形式：CSS属性前缀法、选择器前缀法以及IE条件注释法（即HTML头部引用if IE）。

* 属性前缀法(即类内部Hack)：例如 IE6能识别下划线和星号，IE7能识别星号，但不能识别下划线，IE6~IE10都认识"\9"，但firefox前述三个都不能认识。
* 选择器前缀法(即选择器Hack)：例如 IE6能识别`*html .class{}`，IE7能识别`*+html .class{}`或者`*:first-child+html .class{}`。
* IE条件注释法(即HTML条件注释Hack)(注：IE10+已经不再支持条件注释)： 这类Hack不仅对CSS生效，对写在判断语句里面的所有代码都会生效。

{% codeblock css%}

	.bb{
	  	background-color:red;/*所有识别*/
	 	background-color:#00deff\9; /*IE6、7、8识别*/
	 	+background-color:#a200ff;/*IE6、7识别*/
	 	_background-color:#1e0bd1;/*IE6识别*/
	}

{% endcodeblock %}

###12.li与li之间有看不见的空白间隔是什么原因引起的？有什么解决办法？
A:浏览器的默认行为是把inline元素间的空白字符（空格换行tab）渲染成一个空格，也就是我们上面的代码`<li>`换行后会产生换行字符，而它会变成一个空格，当然空格就占用一个字符的宽度。

解决办法:可以将 ul 的字符间隔消除，将 li 内的字符间隔设为默认。

{% codeblock css %}

	.wrap ul{letter-spacing: -5px;}
	.wrap ul li{letter-spacing: normal;}
{% endcodeblock %}

###13.为什么要初始化CSS样式。
A:因为浏览器的兼容问题，不同浏览器对有些标签的默认值是不同的，如果没对CSS初始化往往会出现浏览器之间的页面显示差异。

###14.absolute的containing block(容器块)计算方式跟正常流有什么不同？
A:

The process for identifying the containing block depends entirely on the value of the element's position property:

1.	If the position property is static or relative, the containing block is formed by the edge of the content box of the nearest ancestor element that is a block container(such as an inline-block, block, or list-item element) or which establishes a formatting context (such as a table container, flex container, grid container, or the block container itself).
2.	If the position property is absolute, the containing block is formed by the edge of the padding box of the nearest ancestor element that has a position value other than static (fixed, absolute, relative, or sticky).
3.	If the position property is fixed, the containing block is established by the viewport (in the case of continuous media) or the page area (in the case of paged media).
4.	If the position property is absolute or fixed, the containing block may also be formed by the edge of the padding box of the nearest ancestor element that has the following:

	1.	A transform or perspective value other than none
	2.	A will-change value of transform or perspective
	3.	A filter  value other than none or a will-change value of filter (only works on Firefox).

Reference:[Layout and the containing block](https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block)  

###15.CSS里的visibility属性有个collapse属性值是干嘛用的？在不同浏览器下有什么区别？
A: 对于普通元素 visibility:collapse; 会将元素完全隐藏，不占据页面布局空间，与display:none;表现相同。如果目标元素为table，visibility:collapse;将table隐藏，但是会占据页面布局空间。 

仅在Firefox下起作用,IE会显示元素,Chrome会将元素隐藏,但是占据空间.

###16.position跟display、margin collapse、overflow、float这些特性相互叠加后会怎么样？
A:

* 如果元素的display为none，那么元素不被渲染，position，float不起作用；
* 如果元素拥有`position:absolute;`或者`position:fixed;`属性那么元素将为绝对定位，float不起作用；
* 如果元素float属性不是none，元素会脱离文档流，根据float属性值来显示，有浮动；
* 绝对定位、inline-block属性的元素，margin不会和垂直方向上的其他元素margin折叠；

###17.对BFC规范(块级格式化上下文：block formatting context)的理解？
A:块格式化上下文（Block Formatting Context，BFC） 是Web页面的可视化CSS渲染的一部分，是布局过程中生成块级盒子的区域，也是浮动元素与其他元素的交互限定区域。

块格式化上下文包含创建它的元素内部的所有内容。块格式化上下文对浮动定位与清除浮动都很重要。浮动定位和清除浮动时只会应用于同一个BFC内的元素。浮动不会影响其它BFC中元素的布局，而清除浮动只能清除同一BFC中在它前面的元素的浮动。外边距折叠（Margin collapsing）也只会发生在属于同一BFC的块级元素之间。

Reference:[块格式化上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)  
###18.css定义的权重
A:

1.	Thousands: Score one in this column if the declaration is inside a style attribute (such declarations don't have selectors, so their specificity is always simply 1000.) Otherwise 0.
2.	Hundreds: Score one in this column for each ID selector contained inside the overall selector.
3.	Tens: Score one in this column for each class selector, attribute selector, or pseudo-class contained inside the overall selector.
4.	Ones: Score one in this column for each element selector or pseudo-element contained inside the overall selector.


###19.请解释一下为什么需要清除浮动？清除浮动的方式?
A:清除浮动是为了清除使用浮动元素产生的影响。浮动的元素，高度会塌陷，而高度的塌陷使我们页面后面的布局不能正常显示。

清除浮动的方式:

* The clearfix hack
* Using overflow
* display: flow-root

Reference:[Floats](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Floats)  

###20.什么是外边距合并？
A:外边距合并指的是，当两个垂直外边距相遇时，它们将形成一个外边距。合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者。

###21.zoom:1的清除浮动原理?
A:当设置了zoom的值之后，所设置的元素就会就会扩大或者缩小，高度宽度就会重新计算了，这里一旦改变zoom值时其实也会发生重新渲染，运用这个原理，也就解决了ie下子元素浮动时候父元素不随着自动扩大的问题。

###22.移动端的布局用过媒体查询吗？

###23.使用 CSS 预处理器吗？喜欢那个？

###24.CSS优化、提高性能的方法有哪些？
A:

作者：赵望野
链接：https://www.zhihu.com/question/19886806/answer/50285495
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

* **加载性能**: 这个方面相关的 best practice 太多了，网上随便找一找就是一堆资料，比如不要用 import 啊，压缩啊等等，主要是从减少文件体积、减少阻塞加载、提高并发方面入手的，任何 hint 都逃不出这几个大方向。
* **选择器性能**: 可以参考 GitHub 的这个分享 https://speakerdeck.com/jonrohan/githubs-css-performance，但 selector 的对整体性能的影响可以忽略不计了，selector 的考察更多是规范化和可维护性、健壮性方面，很少有人在实际工作当中会把选择器性能作为重点关注对象的，但也像 GitHub 这个分享里面说的一样——知道总比不知道好。
* **渲染性能**: 渲染性能是 CSS 优化最重要的关注对象。页面渲染 junky 过多？看看是不是大量使用了 text-shadow？是不是开了字体抗锯齿？CSS 动画怎么实现的？合理利用 GPU 加速了吗？什么你用了 Flexible Box Model？有没有测试换个 layout 策略对 render performance 的影响？这个方面搜索一下 CSS render performance 或者 CSS animation performance 也会有一堆一堆的资料可供参考。
* **可维护性、健壮性**: 命名合理吗？结构层次设计是否足够健壮？对样式进行抽象复用了吗？优雅的 CSS 不仅仅会影响后期的维护成本，也会对加载性能等方面产生影响。这方面可以多找一些 OOCSS（不是说就要用 OOCSS，而是说多了解一下）等等不同 CSS Strategy 的信息，取长补短。


###25.浏览器是怎样解析CSS选择器的？
A:按照从上到下，从右到左的顺序解析。

###26.在网页中的应该使用奇数还是偶数的字体？为什么呢？
A:一般来说，由于各种终端设备的分辨率是偶数的，设计上普遍采用偶数像素体系，奇数像素体系受到排挤，主要的冲突表现在：奇数像素宽度的东西，无法在偶数像素元素内居中显示，总是要差一个像素。

###27.margin和padding分别适合什么场景使用？
A:margin是用来隔开元素与元素的间距；padding是用来隔开元素与内容的间隔。

###28.抽离样式模块怎么写，说出思路，有无实践经验？

###29.元素竖向的百分比设定是相对于容器的高度吗？
A:如果是height的话，是相对于容器高度，如果是padding-height,margin-height则是相对于容器的宽度。

###30.全屏滚动的原理是什么？用到了CSS的那些属性？
A:可以利用css3和div的绝对定位来实现。用到了 position, transform.

Reference:[全屏滚动效果](http://www.alloyteam.com/2015/04/quan-ping-gun-dong-xiao-guo-h5fullscreenpage-js/)  

###31.什么是响应式设计？响应式设计的基本原理是什么？如何兼容低版本的IE？
A:Responsive web design (RWD) is an approach to web design that makes web pages render well on a variety of devices and window or screen sizes.

###32.视差滚动效果，如何给每页做不同的动画？（回到顶部，向下滑动要再次出现，和只出现一次分别怎么做？）
A:

* 利用background-attachment属性实现。 background-attachment: fixed || scroll || local 默认情况下，此属性取值scroll，页面滚动时，内容和背景一起运动，如果取值fixed，背景相对浏览器固定。
* 通过js控制 在页面滚动过程中，我们获取页面的scrollTop的值，根据不同参数值去设置各自scene的top值，这样滚动页面的时候，不同的速度就出来了

Reference:[视差滚动的爱情故事](http://www.alloyteam.com/2014/01/parallax-scrolling-love-story/)  

###33.`::before` 和 `:after`中双冒号和单冒号 有什么区别？解释一下这2个伪元素的作用。
A: 单冒号(:)用于CSS3伪类，双冒号(::)用于CSS3伪元素。（伪元素由双冒号和伪元素名称组成）双冒号是在当前规范中引入的，用于区分伪类和伪元素。不过浏览器需要同时支持旧的已经存在的伪元素写法，比如:first-line、:first-letter、:before、:after等，而新的在CSS3中引入的伪元素则不允许再支持旧的单冒号的写法。

想让插入的内容出现在其它内容前，使用::before，否者，使用::after；

###34.如何修改chrome记住密码后自动填充表单的黄色背景 ？
A:

{% codeblock css %}

	input:-webkit-autofill, textarea:-webkit-autofill, select:-webkit-autofill {
	    background-color: rgb(250, 255, 189); /* #FAFFBD; */
	    background-image: none;
	    color: rgb(0, 0, 0);
	}

{% endcodeblock %}

###35.你对line-height是如何理解的？
A:“行高”顾名思意指一行文字的高度。具体来说是指两行文字间基线之间的距离。

Reference:[css行高line-height的一些深入理解及应用](https://www.zhangxinxu.com/wordpress/2009/11/css%E8%A1%8C%E9%AB%98line-height%E7%9A%84%E4%B8%80%E4%BA%9B%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E5%8F%8A%E5%BA%94%E7%94%A8/)  

###36.设置元素浮动后，该元素的display值是多少？
A: 自动变成了 display:block.

###37.怎么让Chrome支持小于12px 的文字？
A:目前Chrome浏览器已经取消了最小字体限制。

###38.让页面里的字体变清晰，变细用CSS怎么做？
A: `-webkit-font-smoothing: antialiased;`

###39.font-style属性可以让它赋值为“oblique” oblique是什么意思？
A:倾斜的字体样式

###40.position:fixed;在android下无效怎么处理？

###41.如果需要手动写动画，你认为最小时间间隔是多久，为什么？
A:多数显示器默认频率是60Hz，即1秒刷新60次，所以理论上最小间隔为1/60＊1000ms ＝ 16.7ms

###42.display:inline-block 什么时候会显示间隙
A:真正意义上的inline-block水平呈现的元素间，换行显示或空格分隔的情况下会有间距。

Reference:[去除inline-block元素间间距的N种方法](https://www.zhangxinxu.com/wordpress/2012/04/inline-block-space-remove-%E5%8E%BB%E9%99%A4%E9%97%B4%E8%B7%9D/)  
###43.overflow: scroll时不能平滑滚动的问题怎么处理？
A: `-webkit-overflow-scrolling: touch;` 

###44.有一个高度自适应的div，里面有两个div，一个高度100px，希望另一个填满剩下的高度。
A: [How to make a div take the remaining height](https://www.whitebyte.info/programming/css/how-to-make-a-div-take-the-remaining-height)  

###45.png、jpg、gif 这些图片格式解释一下，分别什么时候用。有没有了解过webp？
A:

png:
 优点:无损压缩，有透明选项，压缩效率高于bmp 缺点:一般体积比同尺寸的90%压缩率的jpg要大很多(通常是5倍以上),但人眼很难识别其中的区别 用途:最常见的无损压缩图片格式,如果是经常对某图片进行编辑保存,要求图片数据100%完整,或需要透明效果(给PS当素材或用作图标等)则推荐使用

jpg/jpeg:
 优点:体积比png小，打开速度比png快 缺点:属于有损压缩，每次保存都会产生数据损失(jpeg artifacts)，故不适合用于多次编辑保存。压缩率过高图像会失真 用途:网络上最最常见的格式，也是静态图片(不只是照片)就常用的保存格式，就算是同一小组开发的jp2(相当于jpg第2代)也无法取代它。一般如果肉眼无法识别与png的差别和没有特殊要求就用jpg。

gif:

优点:支持动画以及透明，文件小
缺点:色域不广，只有256色
用途:动态图

webp:WebP是谷歌开发的一种新图片格式，同时支持有损和无损压缩的、使用直接色的、点阵图。
从名字就可以看出来它是为Web而生的，什么叫为Web而生呢？就是说相同质量的图片，WebP具有更小的文件体积。现在网站上充满了大量的图片，如果能够降低每一个图片的文件大小，那么将大大减少浏览器和服务器之间的数据传输量，进而降低访问延迟，提升访问体验。

###46.什么是Cookie 隔离？（或者说：请求资源的时候不要让它带cookie怎么做）
A:如果静态文件都放在主域名下，那静态文件请求的时候都带有的cookie的数据提交给server的，非常浪费流量，所以不如隔离开。

因为cookie有域的限制，因此不能跨域提交请求，利用这点，可以将静态文件放在非主要域名下，这样便隔开了。请求头中就不会带有cookie数据，这样可以降低请求头的大小，降低请求时间，从而达到降低整体请求延时的目的。

同时这种方式不会将cookie传入Web Server，也减少了Web Server对cookie的处理分析环节，提高了webserver的http请求的解析速度。

###47.style标签写在body后与body前有什么区别？
A:从有html标准以来到目前为止（2017年5月），标准一直是规定style元素不应出现在body元素中。
不过网页浏览器一直有容错设计。如果style元素出现在body元素，最终效果和style元素出现在head里是一样的。但是可能引起界面闪烁、重绘或重新布局。

###48.什么是CSS 预处理器 / 后处理器？
A:预处理器例如：LESS、Sass、Stylus，用来预编译Sass或less，增强css代码的复用性，提高工作效率。

后处理器例如：PostCSS，通常被视为在完成的样式表中根据CSS规范处理CSS，让其更有效；目前最常做的是给CSS属性添加浏览器私有前缀，实现跨浏览器兼容性的问题。

###49.rem布局的优缺点?
A:

优点：大小不受层次元素字体影响  
缺点：字体不会跟随缩放；需要使用像素值兼容部分浏览器，不便于维护。

##Reference:

* [前端开发面试题](https://github.com/markyun/My-blog/tree/master/Front-end-Developer-Questions/Questions-and-Answers)  


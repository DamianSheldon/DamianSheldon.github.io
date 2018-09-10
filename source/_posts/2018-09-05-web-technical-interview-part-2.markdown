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

###9.一个满屏 品 字布局 如何设计?

###10.css多列等高如何实现？

###11.经常遇到的浏览器的兼容性有哪些？原因，解决方法是什么，常用hack的技巧 ？

###12.li与li之间有看不见的空白间隔是什么原因引起的？有什么解决办法？

###13.为什么要初始化CSS样式。

###14.absolute的containing block(容器块)计算方式跟正常流有什么不同？

###15.CSS里的visibility属性有个collapse属性值是干嘛用的？在不同浏览器下以后什么区别？

###16.position跟display、margin collapse、overflow、float这些特性相互叠加后会怎么样？

###17.对BFC规范(块级格式化上下文：block formatting context)的理解？

###18.css定义的权重

###19.请解释一下为什么需要清除浮动？清除浮动的方式

###20.什么是外边距合并？

###21.zoom:1的清除浮动原理?

###22.移动端的布局用过媒体查询吗？

###23.使用 CSS 预处理器吗？喜欢那个？

###24.CSS优化、提高性能的方法有哪些？

###25.浏览器是怎样解析CSS选择器的？

###26.在网页中的应该使用奇数还是偶数的字体？为什么呢？

###27.margin和padding分别适合什么场景使用？

###28.抽离样式模块怎么写，说出思路，有无实践经验？

###29.元素竖向的百分比设定是相对于容器的高度吗？

###30.全屏滚动的原理是什么？用到了CSS的那些属性？

###31.什么是响应式设计？响应式设计的基本原理是什么？如何兼容低版本的IE？

###32.视差滚动效果，如何给每页做不同的动画？（回到顶部，向下滑动要再次出现，和只出现一次分别怎么做？）

###33.`::before` 和 `:after`中双冒号和单冒号 有什么区别？解释一下这2个伪元素的作用。

###34.如何修改chrome记住密码后自动填充表单的黄色背景 ？

###35.你对line-height是如何理解的？

###36.设置元素浮动后，该元素的display值是多少？

###37.怎么让Chrome支持小于12px 的文字？

###38.让页面里的字体变清晰，变细用CSS怎么做？

###39.font-style属性可以让它赋值为“oblique” oblique是什么意思？

###40.position:fixed;在android下无效怎么处理？

###41.如果需要手动写动画，你认为最小时间间隔是多久，为什么？

###42.display:inline-block 什么时候会显示间隙

###43.overflow: scroll时不能平滑滚动的问题怎么处理？

###44.有一个高度自适应的div，里面有两个div，一个高度100px，希望另一个填满剩下的高度。

###45.png、jpg、gif 这些图片格式解释一下，分别什么时候用。有没有了解过webp？

###46.什么是Cookie 隔离？（或者说：请求资源的时候不要让它带cookie怎么做）

###47.style标签写在body后与body前有什么区别？

###48.什么是CSS 预处理器 / 后处理器？

###49.rem布局的优缺点?

##Reference:

* [前端开发面试题](https://github.com/markyun/My-blog/tree/master/Front-end-Developer-Questions/Questions-and-Answers)  


---
title: zepto.js 简单的可无限增加内容的 h5 滑屏效果
categories:
  - javascript
tags:
  - h5
  - zepto.js
toc: true
donates: false
thumbnail: https://upload.godread.cn/itblog/zeptojsjiandan_01.jpg
abbrlink: da582b16
date: 2018-01-14 07:22:46
---
> 一个简单的 h5 页面，利用了 zepto.js 的一些 api，可以初步窥探 h5 效果的制作方式。其中尤其要注意的是滑动时的白页面问题，这与文档的结构有关，因此做了自动数量判断，用以解决对应序列页面的显隐。

<!-- more -->

页面在滑动时，会滑动到相应次序的页面，如下图所示

![h5页面滑动图解](https://upload.godread.cn/itblog/zeptojsjiandan_01.jpg)



#### **引入 css/js**

``` html
*按文档结构安排路径*
<link rel="stylesheet" href="./css/reset.css"> // 基础配置 css
<link rel="stylesheet" href="./css/index.css"> // 静态页面 css
<link rel="stylesheet" href="./css/animation.css"> // 动画 css

<script src="../node_modules/zepto/dist/zepto.min.js"></script>
<script src="../node_modules/zepto/src/touch.js"></script>
<script src="./js/index.js"></script> // 滑动逻辑
```

<br>

<br>


#### **文档的结构是这样的**：
``` html
<div id="container">
    <div class="page-group pg-group1 page-current" data-row="1">
        <div class="page page-1-1 page-current">
            <img class="img_1 pt-page-moveFromTop" src="./images/page1_1.png" alt="">
            <img class="img_2 pt-page-moveFromLeft" src="./images/page1_2.png" alt="">
            <img class="common_img pt-page-moveIconUp" src="./images/page_up.png" alt="">
        </div>
    </div>

    <div class="page-group pg-group2 hide" data-row="2">
        <div class="page page-2-1 hide">
            <img class="img_1 hide pt-page-moveFromBottom" src="./images/page2-1_1.png" alt="">
            <img class="img_2 hide pt-page-moveCircle" src="./images/page2-1_2.png" alt="">
            <img class="img_3 hide pt-page-moveFromLeft" src="./images/page2-1_3.png" alt="">
            <img class="img_4 hide pt-page-scaleUp" src="./images/page2-1_4.png" alt="">
            <img class="common_img hide pt-page-moveIconUp" src="./images/page_up.png" alt="">
        </div>

        <div class="page page-2-2 hide">
            <img class="img_1 hide pt-page-flipInLeft" src="./images/page2-2_1.png" alt="">
            <img class="img_2 hide pt-page-flipInLeft" src="./images/page2-2_2.png" alt="">
            <img class="common_img hide pt-page-moveIconUp" src="./images/page_up.png" alt="">
        </div>
    </div>

    <div class="page-group pg-group3 hide" data-row="2">
        <div class="page page-3-1 hide">
            <img class="img_1 hide pt-page-moveFromTop" src="./images/page3-1_1.png" alt="">
            <img class="img_2 hide pt-page-moveCircle" src="./images/page3-1_2.png" alt="">
            <img class="img_3 hide pt-page-moveFromRight" src="./images/page3-1_3.png" alt="">
            <img class="img_4 hide pt-page-scaleUp" src="./images/page2-1_4.png" alt="">
        </div>

        <div class="page page-3-2 hide">
            <img class="img_1 hide pt-page-moveFromBottom" src="./images/page3-2_1.png" alt="">
            <img class="img_2 hide pt-page-moveCircle" src="./images/page3-2_2.png" alt="">
            <img class="img_3 hide pt-page-moveToLeft" src="./images/page3-2_3.png" alt="">
        </div>
    </div>
    ......可以增加更多内页面组
</div>
```

公共图标就是底部的箭头图标，这些都可以自定义去留。


<br>

### 初始化数据

###### *1、滑动方向*

h5 页面的滑动不外乎四个方向，因此，在初始化数据时，定义一个变量，表示四个滑动方向

``` javascript
let direction = {up: 1, right: 2, down: 3, left: 4};
```



###### 2、*横纵方向的两个坐标*

还要初始化两个坐标，分别表示纵向滑动的页面和横向滑动的页面

``` javascript
let last = {col: 0, row: 0}; // 表示出场页面
let now = {col: 1, row: 1}; // 表示入场页面
```



###### 3、*判断页面是否在滑动状态*

另外需注意的是，在滑动时，如果操作过快，会导致入场页面还没有稳定展示就迅速出场，为了防止这种情况，需要在滑动时判断当前页面是否在滑动的状态中

``` javascript
let isMoving = false; // 默认为 false,即没有滑动
```



###### 4、*计算文档中共有几个层级的主页面*

即计算纵向有几个主页面，横向页面为纵向页面的子页面

``` javascript
let mainPage = $(".page-group").length;
```

<br>

### 滑动事件

也即四个方向上的滑动事件，向上、向下、向左、向右。

###### *1、向上滑动*

向上滑动时，计算滑动时出场、入场页面的坐标，并判断是否有相应坐标的入场页面存在，如果存在就入场，如果不存在就保持当前页面不动。

需注意：要先判断入场主页面的子页数量，以进入相应的页面。比如，page-2-2 上滑进入 page-3-2，一一对应。

``` javascript
$(".page").on("swipeUp", function () {
	// 判断页面是否在滑动中
    if (isMoving) {
    	return; // 在滑动中则当前滑动不生效
    }

    // 计算滑动之后 lastPage 页面的坐标
    last.col = now.col;
    last.row = now.row;

    if (last.col < mainPage) {
        // 判断要出场的页面是否存在
        // 获取要出场的页面的最大序号
        let nowRow = $(this).parent('.page-group').next().attr('data-row');

        // 当要出场页面的最大序号小于当前页面,即说明该出场页面不存在,禁止滑动
        if (nowRow < last.row) {
        	return;
        }

        // 计算滑动之后进场页面的坐标
        now.col = last.col + 1;
        now.row = last.row;

        movePage(direction.up);
    }
})
```



###### *2、向下滑动*

逻辑与向下滑动类似

``` javascript
// 向下滑动
$(".page").on("swipeDown", function () {
    let _this = $(this);
    // 判断页面是否在滑动中
    if (isMoving) {
        return; // 在滑动中则当前滑动不生效
    }

    // 计算滑动之后 lastPage 页面的坐标
    last.col = now.col;
    last.row = now.row;

    if (last.col > 1) {
        // 判断要出场的页面是否存在
        // 获取要出场的页面的最大序号
        let lastRow = $(this).parent('.page-group').prev().attr('data-row');

        // 当要出场页面的最大序号小于当前页面,即说明该出场页面不存在,禁止滑动
        if (lastRow < now.row) {
            return;
        }

        // 计算滑动之后进场页面的坐标
        now.col = last.col - 1;
        now.row = last.row;

        movePage(direction.down);
    }
});
```



###### *3、向左滑动*

向左右滑动时，要注意是否会出现空白页，因此需要获取当前主页面的子页数量。由于出场页面是从 0 开始算的，但页面和 css 的命名都是从 1 开始的，因此滑动坐标需 >1，同时 <= mainPage(当前组的页面数量)。

``` javascript
// 向左滑动
$(".page").on("swipeLeft", function () {
    // 判断页面是否在滑动中
    if (isMoving) {
    	return; // 在滑动中则当前滑动不生效
    }

    // 计算滑动之后 lastPage 页面的坐标
    last.col = now.col;
    last.row = now.row;

    // 获取当前页面组的数量
    let maxPage = $(this).parent(".page-group").attr("data-row");

    if (last.col > 1 && last.col <= mainPage) {
        // 如果要出场的页面序号小于当前的页面数量,即滑动,否则不动
        if (last.row < maxPage) {
            // 计算滑动之后进场页面的坐标
            now.col = last.col;
            now.row = last.row + 1;

            movePage(direction.left);
        }
    }
});
```



###### *4、向右滑动*

与向左类似

``` javascript
// 向右滑动
$(".page").on("swipeRight", function () {
	// 判断页面是否在滑动中
	if (isMoving) {
		return; // 在滑动中则当前滑动不生效
	}

	// 计算滑动之后 lastPage 页面的坐标
	last.col = now.col;
	last.row = now.row;

	if (last.col > 1 && last.col <= mainPage && last.row > 0) {
		// 如果当前页面序号大于1(使 now.row 不能等于1),则可以滑动,否则,不滑动
		if (last.row > 1) {
			// 计算滑动之后进场页面的坐标
			now.col = last.col;
			now.row = last.row - 1;

			movePage(direction.right);
		}
	}
});
```

<br>


### 定义一个滑动函数

四个方向的滑动都需要用到这个函数。

页面滑动时，出入场页面增加或移除相应的样式，匹配相应的方向，表示显隐状态。

页面中的每一个图片都要在初始状态隐藏(.hide {display: none;})，这样就不会在页面没有显示时就完成了过渡效果。

在滑动完成后，出场页面图片会停留在当前页面，因此需要在动画执行完成后进行清除，即隐藏。

要注意动画执行完成后，调整页面滑动状态 isMoving。



实际上就是根据方向匹配相应的 css 过渡动画，执行显隐状态的切换。

``` javascript
// 定义一个滑动的功能函数
function movePage(dir) {
	// 	初始化参与动画的页面
	let lastPage = ".page-" + last.col + "-" + last.row;
	let nowPage = ".page-" + now.col + "-" + now.row;

	// 初始两个动画类
	let inClass = "" // 进场动画类
	let outClass = "" // 出场动画类

	// 匹配方向
	switch (dir) {
		case direction.up:
			outClass = "pt-page-moveToTop";
			inClass = "pt-page-moveFromBottom";
			break;
		case direction.right:
			outClass = "pt-page-moveToRight";
			inClass = "pt-page-moveFromLeft";
			break;
		case direction.bottom:
			outClass = "pt-page-moveToBottom";
			inClass = "pt-page-moveFromTop";
				break;
			case direction.left:
				outClass = "pt-page-moveToLeft";
				inClass = "pt-page-moveFromRight";
				break;
		}

	// 将动画类加到参与动画的页面上
	$(lastPage).addClass(outClass).addClass("hide");
	$(nowPage).removeClass("hide").addClass(inClass);

	// 滑动时设置当前滑动状态为 true
	isMoving = true;

	// 动画执行完清除动画类
	setTimeout(function () {
		$(lastPage).find("img").addClass("hide");
		$(lastPage).removeClass(outClass).removeClass("page-current").addClass("hide");
		$(lastPage).parent('.page-group').removeClass('page-current').addClass('hide');

		$(nowPage).parent('.page-group').removeClass('hide').addClass('page-current');
		$(nowPage).find("img").removeClass("hide");
		$(nowPage).removeClass("hide").addClass("page-current").removeClass(inClass);

		// 动画执行完停止滑动状态
		isMoving = false;
	}, 600);
}
```



> 最后要注意的是，禁止页面的默认触摸事件，在 css 中的 .page 里设置 touch-action: none 即可。

完整的项目：[zeptojs-h5](https://www.github.com/godread/zepto-h5)

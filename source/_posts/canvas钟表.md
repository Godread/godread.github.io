---
title: canvas钟表
categories:
  - html5&css3
tags:
  - canvas
  - cases
abbrlink: 8f5b3dc2
date: 2017-09-10 12:33:14
---

canvas可以绘制很多图形，这个实例绘制的是一个对准当前时间的钟表。
其组成是：外层空心圆盘、时针刻度、时针、分针、秒针、表座、秒头。
绘制时注意清除每次叠加的图形。

<!-- more -->

### css部分
``` css
* {
    margin: 0;
    padding: 0;
}

html,
body {
    height: 100%;
    overflow: hidden;
    background: pink;
}

#clock {
    background: gray;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate3d(-50%, -50%, 0);
}
```

### html部分
``` html
<h2 align="center" style="margin-top: 100px; color: blue;">canvas-时针</h2>
<canvas id="clock" width="400" height="400">
    <span>你的浏览器不支持 canvas 元素,请升级当前浏览器或使用其他主流浏览器</span>
</canvas>
```

### js部分
``` javascript
window.onload = function() {
    var clock = document.getElementById('clock');

    if (clock.getContext) {
        var ctx = clock.getContext('2d');

        setInterval(() => {
            // 清除每次叠加的图形
            ctx.clearRect(0, 0, clock.width, clock.height);
            clockMove();
        }, 1000);

        clockMove();

        function clockMove() {
            // 初始化
            ctx.save();
            ctx.lineWidth = 8;
            ctx.strokeStyle = 'black';
            ctx.lineCap = 'round';
            ctx.translate(200, 200);
            ctx.rotate(-90 * Math.PI / 180);
            ctx.beginPath();

            // 外层空心圆盘
            ctx.save();
            ctx.strokeStyle = '#325fa2';
            ctx.lineWidth = 14;
            ctx.beginPath();
            ctx.arc(0, 0, 140, 0, 360 * Math.PI / 180);
            ctx.stroke();
            ctx.restore();

            // 时针刻度
            ctx.save();
            for (var i = 0; i < 12; i++) {
                ctx.beginPath();
                ctx.moveTo(100, 0);
                ctx.lineTo(120, 0);
                ctx.rotate(30 * Math.PI / 180);
                ctx.stroke();
            }
            ctx.restore();

            // 分针刻度
            ctx.save();
            ctx.lineWidth = 4;
            for (var i = 0; i < 60; i++) {
                if (i % 5 != 0) {
                    ctx.beginPath();
                    ctx.moveTo(117, 0);
                    ctx.lineTo(120, 0);
                    ctx.stroke();
                }
                ctx.rotate(6 * Math.PI / 180);
            }
            ctx.restore();

            // 时针、分针、秒针、表座、秒头
            var date = new Date();
            var s = date.getSeconds();
            var m = date.getMinutes();
            var h = date.getHours();
            h = h > 12 ? h - 12 : h;

            // 时针
            ctx.save();
            ctx.lineWidth = 14;
            ctx.rotate(h * 30 * Math.PI / 180);
            ctx.beginPath();
            ctx.moveTo(-20, 0);
            ctx.lineTo(80, 0);
            ctx.stroke();
            ctx.restore();

            // 分针
            ctx.save();
            ctx.lineWidth = 10;
            ctx.rotate(m * 6 * Math.PI / 180);
            ctx.beginPath();
            ctx.moveTo(-28, 0);
            ctx.lineTo(112, 0);
            ctx.stroke();
            ctx.restore();

            // 秒针
            ctx.save();
            ctx.lineWidth = 6;
            ctx.strokeStyle = '#d40000';
            ctx.fillStyle = '#d40000';
            ctx.rotate(s * 6 * Math.PI / 180);
            ctx.beginPath();
            ctx.moveTo(-30, 0);
            ctx.lineTo(83, 0);
            ctx.stroke();

            // 表座
            ctx.beginPath();
            ctx.arc(0, 0, 10, 0, 360 * Math.PI / 180);
            ctx.fill();

            // 秒头
            ctx.beginPath();
            ctx.arc(96, 0, 10, 0, 360 * Math.PI / 180);
            ctx.stroke();
            ctx.restore();

            ctx.restore();
        }
    }
}
```






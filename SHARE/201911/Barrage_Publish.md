### 使用canvas实践视频弹幕

#### 1.常见的弹幕方案

没有弹幕的视频是没有灵魂的，细看各视频网站的弹幕实现方式主要有以下三种：


- dom + position (腾讯视频)
- dom + css transform (优酷，blibli)
- cavans（blibli）


相比第三个方案和第一个和第二个方案会频繁地触发回流和重绘，性能表现较差，不推荐使用。

#### 2.基于canvas的弹幕小demo

![输入图片说明](https://images.gitee.com/uploads/images/2019/1114/151254_52e7f67f_1663282.png "屏幕截图.png")

已实现的功能：
- 渲染预设的弹幕数据
- 可以发送自定义弹幕


代码实现：

DOM结构

```
<div class="wrap">
    <h1>canvas弹幕小demo</h1>
    <div class="main">
        <canvas id="canvas"></canvas>
        <video src="http://vfx.mtime.cn/Video/2019/03/18/mp4/190318231014076505.mp4" id="video" controls width="720" height="360"></video>
    </div>
    <div class="content">
        <input type="text" id="text">
        <input type="button" value="发弹幕" id="btn">
        <div>速度: 1<input type="range" id="speed" max="10" min="1" step="1">10</div>
        <div>透明度: 0<input type="range" id="opacity" max="1" min="0" step="0.01">1</div>
        <div>字体大小: 20<input type="range" id="fontSize" max="40" min="20">40</div>
        <div>字体颜色: <input type="color" id="color"></div>
        <p>弹幕位置:</p>
        <div>
            <input type="radio" id="random" name="position" value="random" checked>
            <label for="random">随机显示</label>
        </div>
        <div>
            <input type="radio" id="upperHalf" name="position" value="upperHalf">
            <label for="upperHalf">上半部分</label>
        </div>
        <div>
            <input type="radio" id="latterrHalf" name="position" value="latterrHalf">
            <label for="latterrHalf">下半部分</label>
        </div>
    </div>
</div>
```

预设的弹幕数据
```
let data = [{ 
    value: '弹幕time5', 
    time: 5, 
    color: 'red', 
    speed: 1, 
    fontSize: 22, 
    range: [0, 0.5]
}, { 
    value: '弹幕time10', 
    time: 10, 
    color: '#00a1f5', 
    speed: 1, 
    fontSize: 30, 
    range: [0, 0.5]
}, {
    value: '弹幕15', 
    time: 15
}]
```

获取DOM节变量实例以及设置默认值
```
// 获取到所有需要的dom元素
let doc = document
let canvas = doc.getElementById('canvas')
let video = doc.getElementById('video')
let $txt = doc.getElementById('text')
let $btn = doc.getElementById('btn')
let $speed = doc.getElementById('speed')
let $opacity = doc.getElementById('opacity')
let $color = doc.getElementById('color')
let $fontSize = doc.getElementById('fontSize')
let $radios = doc.getElementsByName('position')

// 设置默认值
$fontSize.value = 20 // 选取的字号大小
$opacity.value = 1 // 设置透明度
$speed.value = 1 // 设置速度
```

定义CanvasBarrage类和Barrage类来管理弹幕的渲染

CanvasBarrage类
```
class CanvasBarrage {
    // 构造函数，初始化变量值
    constructor(canvas, video, opts = {}) {}
    // 渲染函数
    render() {}
    // 清除画布
    clear() {}
    // 渲染弹幕
    renderBarrage() {}
    // 增加弹幕
    add() {}
    // 回放
    replay() {}
}
```
这里列举两个比较关键的方法
```
render() {
    // 渲染的第一步是清除原来的画布，方便复用写成clear方法来调用
    this.clear()
    // 渲染弹幕
    this.renderBarrage()
    // 如果没有暂停的话就继续渲染
    if (this.isPaused === false) {
        // 通过requestAnimationFrame渲染动画，递归进行渲染
        requestAnimationFrame(this.render.bind(this))
    }
}
```
```
renderBarrage() {
    // 首先拿到当前视频播放的时间
    // 要根据该时间来和弹幕要展示的时间做比较，来判断是否展示弹幕
    let time = this.video.currentTime
    
    // 遍历所有的弹幕，每个barrage都是Barrage的实例
    this.barrages.forEach(barrage => {
        // 用一个flag来处理是否渲染，默认是false
        // 并且只有在视频播放时间大于等于当前弹幕的展现时间时才做处理
        if (!barrage.flag && time >= barrage.time) {
            // 判断当前弹幕是否有过初始化了
            // 如果isInit还是false，那就需要先对当前弹幕进行初始化操作
            if (!barrage.isInit) {
                barrage.init()
                barrage.isInit = true
            }
            // 弹幕要从右向左渲染，所以x坐标减去当前弹幕的speed即可
            barrage.x -= barrage.speed
            barrage.render() // 渲染当前弹幕
            
            // 如果当前弹幕的x坐标比自身的宽度还小了，就表示结束渲染了
            if (barrage.x < -barrage.width) {
                barrage.flag = true // 把flag设为true下次就不再渲染
            }
        }
    })
}
```
Barrage类

```
class CanvasBarrage {
    // 构造函数，初始化变量值
    constructor(canvas, video, opts = {}) {}
    // 初始化函数
    init() {}
    // 渲染函数
    render() {}
}
```
这里列举两个比较关键的方法
```
init() {
    // 如果数据里没有涉及到下面4种参数，就直接取默认参数
    this.color = this.obj.color || this.context.color
    this.speed = this.obj.speed || this.context.speed
    this.opacity = this.obj.opacity || this.context.opacity
    this.fontSize = this.obj.fontSize || this.context.fontSize
    this.range = this.obj.range || this.context.range

    // 为了计算每个弹幕的宽度，我们必须创建一个元素p，然后计算文字的宽度
    let p = document.createElement('p')
    p.style.fontSize = this.fontSize + 'px'
    p.innerHTML = this.value
    document.body.appendChild(p)
    // 把p元素添加到body里了，这样就可以拿到宽度了
    // 设置弹幕的宽度
    this.width = p.clientWidth
    // 得到了弹幕的宽度后，就把p元素从body中删掉吧
    document.body.removeChild(p)
    
    // 设置弹幕出现的位置
    this.x = this.context.canvas.width
    this.y = this.context.canvas.height * getRandomArbitrary(this.range[0], this.range[1])
    // 做下超出范围处理
    if (this.y < this.fontSize) {
        // 超出上边的时候
        this.y = this.fontSize
    } else if (this.y > this.context.canvas.height - this.fontSize) {
        // 超出下边的情况
        this.y = this.context.canvas.height - this.fontSize
    }
}
```
```
render() {
    // 设置画布文字的字号和字体
    this.context.ctx.font = `${this.fontSize}px Arial`
    // 设置画布文字颜色
    this.context.ctx.fillStyle = this.color
    // 设置透明度
    this.context.ctx.globalAlpha = this.opacity
    // 绘制文字
    this.context.ctx.fillText(this.value, this.x, this.y)
}
```

到这个阶段，基本的渲染流程就已经完成了

下面是一些相关的一些事件监听及工具函数
```
// 获取随机颜色
function getColor() {
    return '#' + ('00000' + (Math.random() * 0x1000000 << 0).toString(16)).slice(-6)
}

// 获取指定范围的随机数
function getRandomArbitrary(min, max) {
    return Math.random() * (max - min) + min
}

// 发送弹幕
function send() {
    let value = $txt.value  // 输入的内容
    let time = video.currentTime // 当前视频时间
    let color = $color.value   // 选取的颜色值
    let fontSize = $fontize.value // 选取的字号大小
    let opacity = $opacity.value // 设置透明度
    let speed = $speed.value // 设置速度
    let range = [0, 1] // 设置显示范围
    for(let i = 0 ; i< $radios.length; i++) {
        if ($radios[i].checked) {
            switch($radios[i].id) {
                case 'upperHalf': 
                    range = [0, 0.5]
                    break;
                case 'latterrHalf': 
                    range = [0.5, 1]
                    break;
                case 'random': 
                    range = [0, 1]
                    break;    
            }
        }
    }

    let obj = { value, time, color, fontSize, opacity, speed, range }
    // 添加弹幕数据
    canvasBarrage.add(obj)
    $txt.value = '' // 清空输入框
}
// 点击按钮发送弹幕
$btn.addEventListener('click', send)

// 回车发送弹幕
$txt.addEventListener('keyup', e => {
    let key = e.keyCode
    key === 13 && send()
})

// 开始
video.addEventListener('play', () => {
    canvasBarrage.isPaused = false
    canvasBarrage.render()
})

// 暂停
video.addEventListener('pause', () => {
    // isPaused设为true表示暂停播放
    canvasBarrage.isPaused = true
})

// 监听进度条拖动
video.addEventListener('seeked', () => {
    // 调用CanvasBarrage类的replay方法进行回放，重新渲染弹幕
    canvasBarrage.replay()
})
```

最后创建一个canvasBarrage的实例就完成啦！！！

```
// 创建CanvasBarrage实例
let canvasBarrage = new CanvasBarrage(canvas, video, { data })
```

待实现的功能：
- 使用socket连接动态获取数据
- 如何分配弹幕弹道
- 如何避免弹幕碰撞重叠
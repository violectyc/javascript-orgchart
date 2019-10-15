# 基于orgChart.js 实现横纵向排列的组织结构图

##### 部分截图如下

![1571108003790](C:\Users\violetyc\AppData\Roaming\Typora\typora-user-images\1571108003790.png)

基本代码如下入口函数在fetch

###### html

```html
  <div class="loading">
                <span></span>
                <span></span>
                <span></span>
                <span></span>
                <span></span>
        </div> 
    <div id="chart-container" >
        <button id="export">export</button>
    </div>
```

###### css

```css
body {
            font-family: "Segoe UI", "Helvetica Neue", Helvetica, Arial, sans-serif;
        }
        #chart-container {
            display: block;
            height: calc(100vh - 60px);
            width: calc(100% - 24px);
            border: 2px dashed #aaa;
            border-radius: 5px;
            overflow: auto;
            text-align: center;
            margin: 0 auto;
        }
        .orgchart{
            cursor: pointer;
        }
        #export {
            position: fixed;
            top: 0;
            right: 0;
            z-index: 100;
            display: none;
        }
        .loading{
            display: none;
            width: 80px;
            height: 40px;
            margin: 0 auto;
            margin-top:100px;
            position: absolute;
            z-index: 100;
        }
        .loading span{
            display: inline-block;
            width: 8px;
            height: 100%;
            border-radius: 4px;
            background: lightgreen;
            -webkit-animation: load .8s ease infinite;
        }
        @-webkit-keyframes load{
            0%,100%{
                height: 40px;
                background: lightgreen;
            }
            50%{
                height: 70px;
                margin: -15px 0;
                background: lightblue;
            }
        }
        .loading span:nth-child(2){
            -webkit-animation-delay:0.0s;
        }
        .loading span:nth-child(3){
            -webkit-animation-delay:0.2s;
        }
        .loading span:nth-child(4){
            -webkit-animation-delay:0.4s;
        }
        .loading span:nth-child(5){
            -webkit-animation-delay:0.6s;
        }
```

##### javascript

```javascript
let _scale = 0.3;
let w = null;
let _w = null;
let h = null;
let _h = null;
let _target = null;
// loading
const loading = visible => {
    const chartWrapWidth = $(".loading").width();
    const chartWrapHeight = $(".loading").height();
    $('.loading').css({
        'display': 'block',
        'top': '50%',
        'left': '50%',
        'margin-top': - chartWrapWidth / 2 + 'px',
        'margin-left': - chartWrapHeight / 2 + 'px',
        'visibility': visible
    });
};
// loading('visible');
const ENUMCOLOR = {
    1: "rgb(236, 206, 152)",
    2: "rgb(229, 131, 144)",
    3: "rgb(196, 156, 206)",
    4: "rgb(143, 209, 233)",
    5: "rgb(156, 212, 149)",
    6: "rgb(196, 198, 197)"
};
const templete = data => {
    console.log(data.level);
    if (data.level == 1) {
        return `
    <span data-level="${data.level}" style="display:inline-block;width:520px;height:60px;line-height:60px;
    background: linear-gradient(rgb(207, 9, 0), rgb(244, 96, 0));
    font-size: 22px;
    color: #fff;
    font-weight: 700;
    box-shadow: 0px 2px 2px rgb(0,0,0);"><span 
        style="white-space:nowrap;font-size:24px;letter-spacing:2px;">${data.name}</span></span>
    `;
    } else if (data.level == 2) {
        return `
    <span data-level="${data.level}" style="display:inline-block;width:520px;height:60px;line-height:60px; box-shadow: 0px 2px 2px rgb(0,0,0);
    background:linear-gradient(180deg, rgb(245, 191, 66), rgb(250, 216, 167));font-size: 20px;position:relative;">
    <span style="display:block;position:absolute;left:50%;margin-left:-60px;top:-30px;text-align:center;line-height:30px;font-size:16px;width:60px;height:30px;">${data.value}</span>
    <span style="white-space:nowrap;font-size:22px;font-weight:700;letter-spacing:1px;">${data.name}</span></span>
    `;
    }
    else if (data.level >= 3) {

        return `<span data-level="${data.level}" class="y-content" style="box-sizing:border-box;display:inline-block;border:1px solid rgb(115,115,115);border-radius:3px;
        width:60px;height:520px;text-align:center;">
        <span style="display:inline-block;width:100%;height:30px;color:rgb(8, 8, 8);font-weight:500;line-height:30px;text-align:center;background-color:${ENUMCOLOR[data.type]}">${data.value}</span>    
                <span style="letter-spacing:2px;writing-mode:tb-rl;padding-top:10px;font-size:18px;color:rgb(8,8,8);font-weight:700;">${data.name}</span>
            </span>`;
    }
};
const initCompleted = () => {
    w = $("#chart-container").find(">div").width();
    _w = $("#chart-container").width();
    h = $("#chart-container").find(">div").height();
    _h = $("#chart-container").height();
    _target = $('#chart-container').find('.orgchart');
    // 缩放原点
    _target.css({
        'transform-origin': (w / 2 - _w / 2, h / 2 - _h / 2, 0),
        '-webkit-transform-origin': (w / 2 - _w / 2, h / 2 - _h / 2, 0),
        'transform': 'scale(' + _scale + ')',
        '-webkit-transform': 'scale(' + _scale + ')'
    });
    $('#chart-container').scrollLeft((w * _scale) / 2 - _w / 2);
    $('#chart-container').scrollTop((h * _scale));
    // loading('hidden');
}
const init = res => {
    const options = {
        'data': res.downward,
        'nodeContent': 'value',
        'pan': true,
        'zoom': false,
        'toggleSiblingsResp': true,
        'nodeTemplate': templete,
        'draggable': false,
        'zoomoutLimit': .6,
        'initCompleted': initCompleted
    };
    const chart = $('#chart-container').orgchart(options);

    const dom = $('.y-content');

    dom.each(function (index, item) {
        $(item).closest('table').css({
            'display': 'inline-block'
        });
    });

}
const _fixType = type => {
    type = type.toLowerCase().replace(/jpg/i, 'jpeg');
    let r = type.match(/png|jpeg|bmp|gif/)[0];
    return 'image/' + r;
};
const fileDownload = downloadUrl => {
    let aLink = document.createElement('a');
    aLink.style.display = 'none';
    aLink.href = downloadUrl;
    aLink.download = "下载文件名xxx.png";
    document.body.appendChild(aLink);
    aLink.click();
    document.body.removeChild(aLink);
    let oContainer = document.querySelector("#canvasContainer");
    document.body.removeChild(oContainer);
    controlScale(_scale);
}
const eportImg = () => {
    controlScale(0.8);
    const oContainer = document.createElement('div');
    oContainer.setAttribute('id', 'canvasContainer');
    document.body.append(oContainer);
    html2canvas(_target.get(0)).then(function (canvas) {
        oContainer.appendChild(canvas);
        setTimeout(function () {
            const type = 'png';
            const oCanvas = document.querySelector('#canvasContainer').getElementsByTagName('canvas')[0];
            let imgData = oCanvas.toDataURL(type);
            imgData = imgData.replace(_fixType(type), 'image/octet-stream');
            fileDownload(imgData);
        }, 0);
    });
}



fetch('./d3.json').then(res => res.json()).then(res => {
    init(res);
});
// 入口函数

const controlScale = scale => {

    if (_target != null) {
        _target.css({
            'transform': 'scale(' + scale + ')',
            '-webkit-transform': 'scale(' + scale + ')'
        });
    }
}
const onMouseScroll = e => {
    e.preventDefault();
    let wheel = e.originalEvent.wheelDelta || -e.originalEvent.detail;
    const delta = Math.max(-1, Math.min(1, wheel));
    if (delta < 0) {//向下滚动 放大
        if (_scale >= 0.9) {
            return;
        } else {
            _scale += 0.1;
            controlScale(_scale);
        }
    } else {//向上滚动 缩小
        if (_scale <= 0.3) {
            _scale = 0.3;
            //  $('#chart-container').scrollTop((h));
            return;
        } else {
            _scale -= 0.1;
            controlScale(_scale);
        }
    }
}

$(document).on('mousewheel DOMMouseScroll', onMouseScroll);

((function () {
    setInterval(() => {
        $('.orgchart').css({
            'cursor': 'pointer'
        })
    }, 1);
})()) //取悦boss

```

##### 部分数据格式如下



![1571109034104](C:\Users\violetyc\AppData\Roaming\Typora\typora-user-images\1571109034104.png)

##### 实现了缩放，脱拽，导出，如果要自定义其他节点层级或样式复写下templete这个函数即可。


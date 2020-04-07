---
title: 原生JavaScript实现拖放效果
date: 2020-04-07 21:49:43
tags: [JavaScript]
---

一个简单的拖放效果

![](https://pic.downk.cc/item/5e8c8667504f4bcb043ffa0c.png)

#### 实现过程

##### HTML 很简单

```
<div class="droppable">
    <div class="draggable" draggable="true"></div>
</div>
<div class="droppable"></div>
<div class="droppable"></div>
<div class="droppable"></div>
<div class="droppable"></div>
```

>class 为 droppable 是用于放置被拖动的元素, class 为 draggable 是可被拖动的元素. draggable 属性用于标识元素是否允许使用 拖放操作 API 拖动. true 表示元素可以被拖动. 详情查看 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/draggable)

##### 样式

```
body {
    background-color: rgb(179, 6, 6);
}
.draggable {
    background-image: url('http://source.unsplash.com/random/150x150');
    position: relative;
    height: 150px;
    width: 150px;
    top: 5px;
    left: 5px;
    cursor: pointer;
}
.droppable {
    display: inline-block;
    height: 160px;
    width: 160px;
    margin: 10px;
    border: 3px solid rgb(179, 6, 6);
    background-color: white;
}
.dragging {
    border: 4px solid white
}
.drag-over {
    background-color: #f4f4f4;
    border-style: dashed;
}
.invisible {
    display: none;
}
```

>被拖动元素使用 `http://source.unsplash.com/random/150x150` 随机生成图片, dragging 为被拖动元素的边框样式, drag-over 为被拖动元素要放置但还没放下时的样式.

##### js 实现

通过监听 draggable 和 droppable 的相关事件

```
// 获取被拖动元素
const draggable = document.querySelector('.draggable');
// 获取接收拖动元素[数组]
const droppables = document.querySelectorAll('.droppable');

// 监听 draggable 的相关事件
draggable.addEventListener('dragstart', dragStart);
draggable.addEventListener('dragend', dragEnd);

function dragStart() {
    this.className += ' dragging';
    setTimeout(() => {
        this.className = 'invisible';
    }, 0);
}

function dragEnd() {
    this.className = 'draggable';
}

// 监听 droppable 的相关事件
for (const droppable of droppables) {
    droppable.addEventListener('dragover', dragOver);
    droppable.addEventListener('dragleave', dragLeave);
    droppable.addEventListener('dragenter', dragEnter);
    droppable.addEventListener('drop', dragDrop);
}

function dragOver(e) {
    e.preventDefault();
}

function dragLeave(e) {
    this.className = 'droppable';
}

function dragEnter(e) {
    this.className += ' drag-over';
}

function dragDrop(e) {
    this.className = 'droppable';
    this.append(draggable);
}
```

>当 draggable 元素被拖动时, 原来容器中的 draggable 不会消失, 需要手动将其隐藏. 如果同步操作会立马处罚 dragend 事件, 导致无法进行拖动, 所以在 setTimeout 的回调中异步设置以确保拖动操作开始后, 再将其隐藏.

>在 dragOver 中我们阻止了默认行为, 是因为当被拖动元素放到指定位置并松开鼠标时, 在这一时刻会先触发 dragover 事件然后再触发 drop 事件, 但在 dragover 中默认行为可能会阻止 drop 事件, 所以需要 e.preventDefault() 阻止默认行为.
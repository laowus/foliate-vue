<script setup>
import { open } from './libs/reader.js';
import { onMounted } from 'vue';
const currbook = './by.azw3';

onMounted(() => {
  if (currbook) open(currbook).catch(e => console.error(e))
})

</script>
<template>
  <div id="dimming-overlay" aria-hidden="true"></div>
  <div id="side-bar">
    <!-- 左边目录 -->
    <div id="side-bar-header">
      <!-- 封面 -->
      <img id="side-bar-cover">
      <div>
        <!-- 书名 -->
        <h1 id="side-bar-title"></h1>
        <!-- 作者 -->
        <p id="side-bar-author"></p>
      </div>
    </div>
    <!-- 目录 -->
    <div id="toc-view"></div>
  </div>
  <div id="header-bar" class="toolbar">
    <button id="side-bar-button" aria-label="Show sidebar">
      <svg class="icon" width="24" height="24" aria-hidden="true">
        <path d="M 4 6 h 16 M 4 12 h 16 M 4 18 h 16" />
      </svg>
    </button>
    <!-- 拖动位置 -->
    <div class="title-bar-dragger" id="chapter-title"></div>
  </div>
  <div id="nav-bar" class="toolbar">
    <!-- 上一页切换 -->
    <button id="left-button" aria-label="Go left">
      <svg class="icon" width="24" height="24" aria-hidden="true">
        <path d="M 15 6 L 9 12 L 15 18" />
      </svg>
    </button>
    <!-- 进度条 -->
    <input id="progress-slider" type="range" min="0" max="1" step="any" list="tick-marks">
    <datalist id="tick-marks"></datalist>
    <!-- 下一页切换 -->
    <button id="right-button" aria-label="Go right">
      <svg class="icon" width="24" height="24" aria-hidden="true">
        <path d="M 9 6 L 15 12 L 9 18" />
      </svg>
    </button>
  </div>
</template>

<style>
:root {
  /* 设置背景色 */
  --active-bg: rgba(0, 0, 0, .05);
}

/* 测试浏览器是否支持CSS功能 */
@supports (color-scheme: light dark) {
  @media (prefers-color-scheme: dark) {
    :root {
      --active-bg: rgba(255, 255, 255, .1);
    }
  }
}

html {
  height: 100%;
}

body {
  margin: 0 auto;
  height: 100%;
  font: menu;
  font-family: system-ui, sans-serif;
}

#drop-target {
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  visibility: hidden;
}

#drop-target h1 {
  font-weight: 900;
}

#file-button {
  font: inherit;
  background: none;
  border: 0;
  padding: 0;
  text-decoration: underline;
  cursor: pointer;
}

.icon {
  display: block;
  fill: none;
  stroke: currentcolor;
  stroke-width: 2px;
}

.empty-state-icon {
  margin: auto;
}

/* 导航栏的设置 
    内部元素两边对齐,中间自适应
    */
.toolbar {
  box-sizing: border-box;
  /* 绝对定位的元素的位置相对于最近的已定位父元素 */
  position: absolute;
  z-index: 1;
  display: flex;
  /* 设置次轴方向的对齐方式 垂直方向居中 */
  align-items: center;
  /* 在水平方向上，两边元素贴边，其余元素均匀分布 */
  justify-content: space-between;
  width: 100%;
  height: 48px;
  padding: 6px;
  transition: opacity 250ms ease;
  visibility: hidden;
}

/* 内部按钮 */
.toolbar button {
  padding: 3px;
  border-radius: 6px;
  background: none;
  border: 0;
  color: GrayText;
}

.toolbar button:hover {
  background: rgba(0, 0, 0, .1);
  color: currentcolor;
}

/* 顶部 */
#header-bar {
  top: 0;
}

/* 底部 */
#nav-bar {
  bottom: 0;
}

#progress-slider {
  /* 当‌flex-grow的值大于0‌时，元素会根据其flex-grow的值按比例分配剩余空间。 */
  flex-grow: 1;
  margin: 0 12px;
  visibility: hidden;
}

/* 目录栏 */
#side-bar {
  visibility: hidden;
  box-sizing: border-box;
  z-index: 2;
  /* 左上角显示 */
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 320px;
  transform: translateX(-320px);
  display: flex;
  flex-direction: column;
  background: Canvas;
  color: CanvasText;
  /* 添加阴影 */
  box-shadow: 0 0 0 1px rgba(0, 0, 0, .2), 0 0 40px rgba(0, 0, 0, .2);
  transition: visibility 0s linear 300ms, transform 300ms ease;
}

/* 目录栏出现后的动画 */
#side-bar.show {
  visibility: visible;
  transform: translateX(0);
  transition-delay: 0s;
}

/* 遮罩层 用于点击关闭目录栏 */
#dimming-overlay {
  visibility: hidden;
  position: fixed;
  z-index: 2;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, .2);
  opacity: 0;
  transition: visibility 0s linear 300ms, opacity 300ms ease;
}

#dimming-overlay.show {
  visibility: visible;
  /* 透明度 */
  opacity: 1;
  transition-delay: 0s;
}

/* 目录页顶部 */
#side-bar-header {
  padding: 1rem;
  display: flex;
  /* 底部边框 */
  border-bottom: 1px solid rgba(0, 0, 0, .1);
  align-items: center;
}

/* 封面显示 */
#side-bar-cover {
  /* 视窗高度的10% */
  height: 10vh;
  min-height: 60px;
  max-height: 180px;
  border-radius: 3px;
  border: 0;
  background: lightgray;
  /* 外部阴影 */
  box-shadow: 0 0 1px rgba(0, 0, 0, .1), 0 0 16px rgba(0, 0, 0, .1);
  /* 设置水平方向的左右外边距 */
  margin-inline-end: 1rem;
}

/* 判断img 这个节点的 src 是否有 */
#side-bar-cover:not([src]) {
  display: none;
}

#side-bar-title {
  margin: .5rem 0;
  /* 继承他的父元素的样式 */
  font-size: inherit;
}

#side-bar-author {
  margin: .5rem 0;
  font-size: small;
  color: GrayText;
}

#toc-view {
  padding: .5rem;
  overflow-y: scroll;
}

#toc-view li,
#toc-view ol {
  margin: 0;
  padding: 0;
  list-style: none;
}

#toc-view a,
#toc-view span {
  display: block;
  border-radius: 6px;
  padding: 8px;
  margin: 2px 0;
}

#toc-view a {
  color: CanvasText;
  text-decoration: none;
}

#toc-view a:hover {
  background: var(--active-bg);
}

#toc-view span {
  color: GrayText;
}

#toc-view svg {
  margin-inline-start: -24px;
  padding-inline-start: 5px;
  padding-inline-end: 6px;
  fill: CanvasText;
  cursor: default;
  transition: transform .2s ease;
  opacity: .5;
}

#toc-view svg:hover {
  opacity: 1;
}

#toc-view [aria-current] {
  font-weight: bold;
  background: var(--active-bg);
}

#toc-view [aria-expanded="false"] svg {
  transform: rotate(-90deg);
}

#toc-view [aria-expanded="false"]+[role="group"] {
  display: none;
}

.menu-container {
  position: relative;
}

.menu,
.menu ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.menu {
  visibility: hidden;
  position: absolute;
  right: 0;
  background: Canvas;
  color: CanvasText;
  border-radius: 6px;
  box-shadow: 0 0 0 1px rgba(0, 0, 0, .2), 0 0 16px rgba(0, 0, 0, .1);
  padding: 6px;
  cursor: default;
}

.menu.show {
  visibility: visible;
}

.menu li {
  padding: 6px 12px;
  padding-left: 24px;
  border-radius: 6px;
}

.menu li:hover {
  background: var(--active-bg);
}

.menu li[aria-checked="true"] {
  background-position: center left;
  background-repeat: no-repeat;
  background-image: url('data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%2224%22%20height%3D%2224%22%3E%3Ccircle%20cx%3D%2212%22%20cy%3D%2212%22%20r%3D%223%22%2F%3E%3C%2Fsvg%3E');
}

.popover {
  background: Canvas;
  color: CanvasText;
  border-radius: 6px;
  box-shadow: 0 0 0 1px rgba(0, 0, 0, .2), 0 0 16px rgba(0, 0, 0, .1), 0 0 32px rgba(0, 0, 0, .1);
}

.popover-arrow-down {
  fill: Canvas;
  filter: drop-shadow(0 -1px 0 rgba(0, 0, 0, .2));
}

.popover-arrow-up {
  fill: Canvas;
  filter: drop-shadow(0 1px 0 rgba(0, 0, 0, .2));
}

/* 顶部章节名字显示 */
#chapter-title {
  flex: 1;
  font-size: 14px;
  color: gainsboro;
  align-items: center;
  display: flex;
  justify-content: center;
}
</style>

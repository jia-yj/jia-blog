---
title: "借助 AI 搭建博客"
date: 2026-03-18
categories: ["日志"]
---

*本文由 AI 生成。*

这个网站是和 Claude 一起搭的，断断续续花了几个小时。记录一下过程中几个有意思的地方。

## 技术选型

Hugo + PaperMod + Vercel + Cloudflare R2。构建快，托管简单，图片有公开 URL。PaperMod 的默认风格不够用，大部分工作花在自定义布局上。

## 摄影画廊

大图 + 缩略图条 + 键盘切换。导航按钮一开始放在容器内，竖图时会被遮住点不到，改成 `position: fixed` 固定在视口两侧才解决。

```css
.photo-nav-btn {
  position: fixed;
  top: 50%;
  transform: translateY(-50%);
}
```

中间加过淡入淡出效果，看了一眼说不好，直接还原。

## 首页

一张大图 + 一句话。图片要突破 PaperMod 容器限制：

```css
.home-hero {
  width: 100vw;
  position: relative;
  left: 50%;
  transform: translateX(-50%);
}
```

图片大小来回调了五六次，字体也试了霞鹜文楷和思源宋体，最后都放弃，觉得默认的更干净。

## 足迹地图

Leaflet.js 画世界地图，去过的省份和国家填色。法属圭亚那和法国共用 ISO 代码 250，直接着色会把南美洲也染上，用经度过滤解决：

```javascript
if (id === 250) {
  if (bounds.getCenter().lng < -30) return defaultStyle;
}
```

## 编辑排版

图片不是均匀网格，而是有大有小、左右错位。每种 layout 对应固定规则，不能随意发挥：

```yaml
blocks:
  - type: single
    layout: large-left  # 永远是 72% 宽靠左
    spaceAfter: md
  - type: pair
    images:
      - { src: "...", size: large }
      - { src: "...", size: small }
```

手机端单独写了一套降级规则，保留大小感，去掉极端偏移。

排版调了十几次，"靠右""放大""不好回来"……执行是 AI 的事，判断还是得自己来。

## 感受

效率确实高，但有几类问题 AI 给不了答案：视觉好不好看，方向对不对，审美上的取舍。这些只有真的看到才知道。

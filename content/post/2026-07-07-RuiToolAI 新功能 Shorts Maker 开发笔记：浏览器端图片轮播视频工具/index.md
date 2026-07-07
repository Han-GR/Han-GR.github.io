---
title: RuiToolAI 新功能 Shorts Maker 开发笔记：浏览器端图片轮播视频工具
description: ""
slug: 2026-07-07-RuiToolAI 新功能 Shorts Maker 开发笔记：浏览器端图片轮播视频工具
date: 2026-07-07
image: shorts.jpg
categories:
  - 前端
  - 效率&工具
tags:
  - Canvas
  - MediaRecorder
  - ffmpeg
  - RuiToolAI
---

## 1. 为什么做这个功能

RuiToolAI 准备在 YouTube 上做内容宣传。不做硬广，把用户爱看的东西，做成 Shorts 短视频。

问题来了：用什么工具做？市面上的视频编辑软件要么太重，要么要导出再上传，流程割裂。最简单的方式是：在 RuiToolAI 网站上直接生成图片，然后一键拼成轮播视频——**生成到发布，一个网站搞定**。

于是就有了 Shorts Maker。免费开放给所有用户。

## 2. 核心功能一览

操作起来很简单，三步出片：

1. **选图**：本地上传图片，或从文生图历史记录中挑选
2. **配乐**：上传背景音乐，拖动滑块调节音量，预览实时生效
3. **导出**：点击导出，浏览器本地录制 WebM，直接下载

全程纯浏览器端处理，不上传任何文件到服务器。页面右侧有一个手机 Mockup 预览窗口，实时播放当前视频效果，所见即所得。

## 3. 技术实现

整个工具跑在浏览器里，核心依赖三个 Web API：

- **Canvas + `captureStream`**：把逐帧绘制的画面转成视频流
- **MediaRecorder**：录制视频流输出 WebM
- **Web Audio API**：解码音频文件，混入视频流

### 3.1 Canvas 录制：从 rAF 到 setInterval 的踩坑

最初的实现用 `requestAnimationFrame` 驱动录制循环：

```javascript
const render = (now) => {
  const elapsedMs = now - startTime;
  drawShortsFrame(ctx, { elapsedMs, ... });
  if (elapsedMs >= totalDurationMs) {
    recorder.stop();
    return;
  }
  requestAnimationFrame(render);
};
```

测试时一切正常。但导出的视频偶尔会在某一张图片上卡住，画面停在那里不动。

**原因**：`requestAnimationFrame` 在浏览器标签页不可见时会**被节流**（降到 1fps 甚至暂停）。用户导出时切到别的标签页，rAF 的时间戳 `now` 就停住了，elapsed 不再推进，canvas 一直画同一帧，录出来的视频就卡死在那张图上。

**修复**：改用 `setInterval` 驱动，用 `performance.now()` 计算真实流逝时间，不受标签页可见性影响：

```javascript
const intervalMs = Math.round(1000 / EXPORT_FPS);
const timer = setInterval(() => {
  const elapsedMs = Math.min(performance.now() - startTime, totalDurationMs);
  drawShortsFrame(ctx, { elapsedMs, ... });
  if (elapsedMs >= totalDurationMs) {
    clearInterval(timer);
    recorder.stop();
    resolve();
  }
}, intervalMs);
```

另外 `MediaRecorder.onstop` 回调的绑定时机也有讲究。`recorder.stop()` 是异步的，浏览器需要先 flush 最后一批数据才触发 `onstop`。如果在 Promise 创建时就绑定 `onstop`，中间如果有其他操作关闭了相关上下文，可能导致 `onstop` 永远不触发。正确做法是在 `recorder.stop()` **之前**才绑定：

```javascript
recorder.onstop = () => resolveRecording(new Blob(chunks, { type: mimeType }));
recorder.onerror = () => rejectRecording(new Error("Video export failed."));
recorder.stop();
```

### 3.2 音频处理：绕过浏览器自动播放策略

背景音乐需要同时满足两个需求：预览时能听到，导出时能混入视频。

**预览**：直接用 `<audio>` 元素，`src` 设为 `URL.createObjectURL(file)`。

**导出**：不能用 `<audio>`，因为 `captureStream` 只捕获 Canvas 的画面。需要用 Web Audio API 创建独立的音频轨道，手动连接到 `MediaStream`：

```javascript
const audioContext = new AudioContext();
const arrayBuffer = await musicTrack.file.arrayBuffer();
const audioBuffer = await audioContext.decodeAudioData(arrayBuffer);

const destination = audioContext.createMediaStreamDestination();
const gainNode = audioContext.createGain();
gainNode.gain.value = musicVolume / 100;

const source = audioContext.createBufferSource();
source.buffer = audioBuffer;
source.loop = musicTrack.shouldLoop;
source.connect(gainNode);
gainNode.connect(destination);

// 把音频轨道加到导出流中
for (const track of destination.stream.getAudioTracks()) {
  exportStream.addTrack(track);
}
```

这里踩过一个坑：一开始用 `createMediaElementSource(audioElement)` 配合 `audioElement.play()`，但浏览器的自动播放策略会拦截 `.play()` 调用，导致 `The audio track could not start` 错误。改用 `AudioBufferSourceNode` + `decodeAudioData` 完全绕过了这个问题。

另一个坑：用 `fetch(musicTrack.url)` 读取 blob URL 的音频数据，在某些浏览器中会报 `Failed to fetch`。因为 blob URL 在不同上下文之间不可访问。改为直接用 `musicTrack.file.arrayBuffer()` 读取原始文件。

### 3.3 MP4 转码的放弃：ffmpeg.wasm 不是银弹

最初计划支持 MP4 导出，用 ffmpeg.wasm 在浏览器端把 WebM 转成 MP4。

遇到的问题：

1. **31MB 的 wasm 文件**：免费版 Cloudflare Workers 单文件限制 25MB，超限无法部署。改 CDN 加载后，COEP `require-corp` 模式下 CDN 资源缺少 CORP 头，浏览器拒绝加载。改成 `credentialless` 模式解决了加载问题。

2. **`libx264` 编码在 wasm 中极慢**：转码几分钟没反应，用户以为卡死。改 `-c:v copy` 跳过视频重编码，但 VP8/VP9 in MP4 兼容性差，iOS Safari 不认。

3. **进度条不可靠**：ffmpeg.wasm 的 `progress` 事件经常不触发，返回负数或超大值。用定时器模拟进度也只能让用户看到数字在动，实际转码可能已经挂了。

综合考虑，**WebM 已经足够**。YouTube 完全支持 WebM 上传，Android Chrome 也能播放。唯一问题是 iOS Safari 不支持 WebM 本地播放，但目标是上传到短视频平台，不是本地播放。最终决定只保留 WebM 导出，删除所有 ffmpeg 相关代码。

### 3.4 音量实时预览：不触发重渲染的 DOM 操作

音量滑块需要两个效果：拖动时实时改变预览音量，导出时按比例混音。

最初把 `volume` 放在 `musicTrack` state 里，拖动滑块时 `setMusicTrack` 触发重渲染，`<audio>` 元素的 `loop` prop 被 React 重新设置，导致音频播放中断。

**修复**：把 `volume` 拆成独立 state `musicVolume`，`onChange` 里直接操作 DOM：

```javascript
onChange={(e) => {
  const vol = Number(e.target.value);
  setMusicVolume(vol);
  // 直接操作 DOM，不触发 <audio> 重渲染
  if (previewAudioRef.current) {
    previewAudioRef.current.volume = vol / 100;
  }
}}
```

`<audio>` 元素不会被 React 重新渲染，播放状态保持不变。

## 4. 交互细节

几个小但重要的交互设计：

- **历史图片空状态**：没有历史图片时，显示引导按钮跳转到文生图功能，而不是空着一片白
- **图片悬浮 + 号**：鼠标移到历史图片上显示半透明覆盖层和 + 号，点击添加，干净不占空间
- **Ending Card**：每段视频末尾自动加上 RuiTool AI 的结尾卡，目前强制开启，后续有可能作为付费功能
- **导出提示**：导出中显示橙色警告文字，提醒用户不要切换标签页，但其实切换也没事，只要不关掉就行

## 5. 总结

Shorts Maker 是一个典型的"浏览器能力边界探索"项目。Canvas 录制、Web Audio 混音、MediaRecorder 导出，这些 API 组合起来能做出一个完整的视频工具，但每个 API 都有坑：

- `requestAnimationFrame` 在后台会被节流
- `MediaRecorder.onstop` 的绑定时序影响 Promise 是否 resolve
- `createMediaElementSource` 受自动播放策略限制
- ffmpeg.wasm 在浏览器里跑视频编码仍然太慢

最终的方案是：**用 WebM，做减法，保证稳定**。功能不一定最多，但每个功能都要可靠。

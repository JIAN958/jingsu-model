# AI Studio 网页实现

> 这是一份面向初学者的源码学习笔记。读完后你能搞懂：单文件 HTML 应用是怎么把"前端 + 调 API + 本地存储"这一套跑起来的。

---

## 一、整体认识

### 1.1 

一个单 HTML 文件的 AI 图片/视频生成工具，所有代码（结构、样式、逻辑）都塞在一个 `.html` 里。

打开浏览器双击即可运行，**不需要任何后端服务器**，因为所有功能靠两件事：

1. **浏览器直接调 AI 服务的 API**（fetch 发请求）
2. **浏览器本地存储**（IndexedDB 存图片、localStorage 存设置）

### 1.2 文件结构（鸟瞰）

```
ai-studio-test-UC.html
├── <head>
│   └── <style> ... </style>      ← 所有样式（CSS）
└── <body>
    ├── <div class="topbar">       ← 顶部栏（标题、Tab、日志按钮）
    ├── <aside class="sidebar">    ← 左边栏（表单输入）
    ├── <main>                     ← 主区域（预览结果 + 历史）
    ├── <aside class="drawer">     ← 右抽屉（请求日志）
    └── <script> ... </script>     ← 所有 JS 逻辑
```

---

## 二、核心技术栈（没有任何框架！）

| 技术 | 用途 | 类比 |
|---|---|---|
| **原生 HTML** | 页面结构 | 房子的骨架 |
| **原生 CSS** | 视觉样式 | 房子的装修 |
| **原生 JavaScript** | 交互逻辑 | 房子的电路水管 |
| **CSS Grid** | 整体布局 | 把房子分成几个房间 |
| **fetch API** | 调用 AI 服务 | 打电话给外卖店 |
| **IndexedDB** | 浏览器本地数据库 | 抽屉里的相册 |
| **localStorage** | 浏览器小型存储 | 便签纸 |
| **FileReader** | 读取本地图片 | 扫描仪 |

**重点：没有用 Vue/React 这些框架，所有交互都是 `document.getElementById(...)` 这种最原始的写法。** 适合学习"浏览器到底能做什么"。

---

## 三、布局

### 3.1  CSS Grid

CSS Grid 是把一个矩形划成几格，然后告诉每个区块"你坐哪一格"。

代码里这段：

```css
body {
  display: grid;
  grid-template-rows: 52px 1fr;       /* 上面 52 像素高，下面占剩余空间 */
  grid-template-columns: 360px 1fr;   /* 左边 360 像素宽，右边占剩余 */
  grid-template-areas:
    "topbar topbar"     /* 第一行：topbar 占满 */
    "sidebar main";     /* 第二行：左 sidebar，右 main */
}
```

效果：

```
┌──────────────────────────────────┐
│           topbar (52px)          │
├──────────┬───────────────────────┤
│          │                       │
│ sidebar  │        main           │
│ (360px)  │     (剩余空间)         │
│          │                       │
└──────────┴───────────────────────┘
```

### 3.2 颜色为什么"看起来很专业"？

代码顶部定义了一堆 **CSS 变量**（`--bg-base`, `--fg`, `--accent`...）。整套页面只用这十几个色值，所以视觉上很统一。

```css
:root {
  --bg-base: #0c0c0f;     /* 最深的底色 */
  --bg-elev-1: #15151a;   /* 稍亮一点 */
  --accent: #a78bfa;      /* 紫色点缀 */
}
```

后面所有用到颜色的地方都写 `var(--bg-base)`，要改主题色只改这一处。

---

## 四、主要功能模块拆解

### 4.1 双 Tab 切换（图像 / 视频）

**思路**：两个 sidebar 区域并排存在，根据当前 Tab 显示一个、隐藏一个。

```javascript
function switchMode(mode) {
  currentMode = mode;
  document.getElementById('sidebar-img').style.display = mode === 'img' ? '' : 'none';
  document.getElementById('sidebar-vid').style.display = mode === 'vid' ? '' : 'none';
}
```

就这么朴素——没有什么"路由""页面跳转"，纯粹是显示和隐藏。

### 4.2 图片生成（核心）

#### 流程图

```
用户填写表单
    ↓
点击"生成"按钮
    ↓
imgGenerate() 函数被调用
    ↓
读取所有输入（API地址、key、prompt、变体...）
    ↓
展开成多个任务（变体数 × 每变体张数）
    ↓
并发池（最多 6 个同时跑）
    ↓
每个任务 → imgSingleRequest()
    ↓
fetch 调用 API
    ↓
拿到图片 → 显示在画廊
```

#### 关键代码片段

**发请求那一步**：

```javascript
const resp = await fetch(apiUrl, {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer ' + key,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ model, prompt, size, quality })
});
const data = await resp.json();
```

这就是浏览器跟 AI 服务"对话"的全部——发 POST、附带 key 和参数、等响应。

**把返回的 base64 图片变成可显示的图**：

```javascript
const bin = atob(item.b64_json);              // base64 解码成字节
const bytes = new Uint8Array(bin.length);
for (let i = 0; i < bin.length; i++) bytes[i] = bin.charCodeAt(i);
const blob = new Blob([bytes], { type: 'image/png' });   // 包装成 Blob
const url = URL.createObjectURL(blob);                    // 生成临时 URL
// 现在 <img src={url}> 就能显示了
```

### 4.3 参考图上传

**思路**：用户选了文件 → FileReader 读成 base64 → 显示缩略图 → 发请求时塞进 FormData。

```javascript
function imgRefHandle(role, file) {
  imgRefs[role].file = file;
  const r = new FileReader();
  r.onload = e => {
    // e.target.result 就是 "data:image/png;base64,xxxxx..."
    img.src = e.target.result;   // 直接当 img 的 src 用
  };
  r.readAsDataURL(file);
}
```

**拖拽上传**就是监听 `dragover` 和 `drop` 事件：

```javascript
tile.addEventListener('drop', e => {
  e.preventDefault();
  const file = e.dataTransfer.files[0];   // 拖进来的文件
  imgRefHandle(role, file);
});
```

### 4.4 并发控制（最多同时跑 6 个）

**问题**：如果一次生成 20 张，全部并发出去会被 API 限流。

**解法**：开 6 个"工人"，谁先做完谁去拿下一个任务。

```javascript
let nextIdx = 0;
const worker = async () => {
  while (nextIdx < jobs.length) {
    const myIdx = nextIdx++;
    const job = jobs[myIdx];
    await imgSingleRequest(...job);   // 串行执行自己的任务
  }
};
// 同时启动 6 个 worker
await Promise.all([worker(), worker(), worker(), worker(), worker(), worker()]);
```

这就是经典的**生产者-消费者模式**，是非常通用的并发技巧。

### 4.5 视频生成（异步轮询）

视频不像图片那样秒出，需要 1~3 分钟。所以流程是：

```
提交任务 → 拿到 task_id → 每 5 秒查一次状态 → 完成或失败
```

```javascript
vidPollTimer = setInterval(async () => {
  const resp = await fetch(statusUrl + '?task_id=' + taskId);
  const data = await resp.json();
  if (data.output.task_status === 'Success') {
    clearInterval(vidPollTimer);   // 停止轮询
    // 显示视频
  }
}, 5000);
```

`setInterval` 就是"每隔 N 毫秒执行一次"。

### 4.6 历史记录（IndexedDB）

**localStorage 只能存几 MB 字符串**，存图片不够用。所以这里用 **IndexedDB**——浏览器自带的、能存几个 GB 的本地数据库。

可以把它想象成浏览器里的一个小数据库表：

```
items 表:
┌─────────────┬───────┬──────┬────────┐
│ id          │ type  │ blob │ ts     │
├─────────────┼───────┼──────┼────────┤
│ i_xxx_abc   │ image │ ...  │ 时间戳  │
│ v_xxx_def   │ video │ ...  │ 时间戳  │
└─────────────┴───────┴──────┴────────┘
```

代码里封装成一个 `HistoryDB` 对象，提供 4 个方法：

```javascript
HistoryDB.add(record)   // 加一条
HistoryDB.list()        // 列出所有
HistoryDB.remove(id)    // 删一条
HistoryDB.clear()       // 全清
```

**IndexedDB 的回调写法很啰嗦**（要 `onsuccess`、`onerror`），所以代码把它包成了 Promise，调用时直接 `await` 就行。这是处理"老 API"的常见技巧。

### 4.7 国际化（中英文切换）

**思路**：所有文字写成 key，渲染时根据当前语言查字典。

```javascript
const I18N = {
  zh: { 'btn.gen.image': '生成图像', 'btn.download': '下载', ... },
  en: { 'btn.gen.image': 'Generate',  'btn.download': 'Download', ... }
};

function t(key) {
  return I18N[currentLang][key];
}
```

HTML 元素上加 `data-i18n="btn.download"`，切换语言时遍历所有这种元素：

```javascript
document.querySelectorAll('[data-i18n]').forEach(el => {
  el.textContent = t(el.getAttribute('data-i18n'));
});
```

### 4.8 日志抽屉（右侧滑出）

每次发请求/收响应都调 `addLog()` 记一条。

**抽屉的"滑出"效果**靠 CSS transform：

```css
.drawer {
  transform: translateX(100%);              /* 默认推到屏幕外 */
  transition: transform .28s var(--ease-out);
}
.drawer.open {
  transform: translateX(0);                 /* 加 open 类就滑回来 */
}
```

JS 只需要给元素加/删 `open` 类，CSS 自动播放动画。

---

## 五、关键设计思想（这些是通用知识）

### 5.1 状态机思维

每个功能区都有几种**状态**，UI 跟着状态变：

| 状态 | 显示什么 |
|---|---|
| 初始 | 占位符（"结果将显示在这里"） |
| 生成中 | 旋转图标 + 进度 `3/8` |
| 完成 | 画廊 + 下载按钮 |
| 失败 | 红色错误条 |

代码里通过 `style.display` 控制不同元素的显隐来实现状态切换。

### 5.2 不污染全局

JS 用 `let` / `const` 替代 `var`，并用立即执行函数（IIFE）封装：

```javascript
const HistoryDB = (() => {
  let _db = null;       // 私有变量，外界看不到
  return {
    add(...) { ... },
    list(...) { ... }
  };
})();
```

外界只能用 `HistoryDB.add()`，碰不到 `_db`。这就是 JS 里实现"私有变量"的经典手段。

### 5.3 错误降级

API key 没填、网络断了、模型挂了——任何一步都可能失败。代码里全是 `try/catch`：

```javascript
try {
  const data = await ...;
} catch (e) {
  setStatus('错误: ' + e.message, 'error');
}
```

并且失败时让 UI 显示明确的红色提示，而不是悄悄出错。

### 5.4 内存清理

生成的图片是 Blob URL（`blob:http://...`），不主动释放会一直占内存。所以重新生成前会：

```javascript
imgResults.forEach(r => {
  if (r.isBlob) URL.revokeObjectURL(r.url);
});
```

---

## 六、读源码的建议路径

如果你想完整读懂这份代码，按这个顺序：

1. **先看 HTML 结构**（搜 `<div class="topbar">`、`<aside class="sidebar">`），心里建立"哪里是哪里"的地图
2. **看 CSS 变量定义**（顶部 `:root { ... }`），明白颜色体系
3. **看 `applyI18n()` 和 `switchMode()`**，理解最简单的"读 DOM、改 DOM"模式
4. **看 `imgSingleRequest()`**，这是最核心的"调 API"逻辑
5. **看 `imgGenerate()`**，理解并发控制和状态更新
6. **看 `HistoryDB`**，学 IndexedDB 怎么用
7. **看 `vidStartPolling()`**，学异步轮询模式

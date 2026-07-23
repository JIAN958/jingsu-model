# AI Studio

> 一个基于浏览器端的 AI 图像与视频生成工作台，单文件、零依赖、开箱即用。

<p align="center">
  <img src="https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white" alt="HTML5" />
  <img src="https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white" alt="CSS3" />
  <img src="https://img.shields.io/badge/VanillaJS-F7DF1E?logo=javascript&logoColor=black" alt="VanillaJS" />
  <img src="https://img.shields.io/badge/Single%20File-Yes-8A2BE2" alt="Single File" />
  <img src="https://img.shields.io/badge/License-MIT-green" alt="License" />
</p>

---

## 目录

- [项目简介](#项目简介)
- [核心特性](#核心特性)
- [快速开始](#快速开始)
- [功能详解](#功能详解)
  - [图像生成](#图像生成)
  - [图像编辑](#图像编辑)
  - [视频生成](#视频生成)
  - [切分工具](#切分工具)
- [多区域节点支持](#多区域节点支持)
- [本地存储](#本地存储)
- [技术实现](#技术实现)
- [文件结构](#文件结构)
- [使用示例](#使用示例)
- [常见问题](#常见问题)
- [许可协议](#许可协议)

---

## 项目简介

**AI Studio** 是一个纯前端实现的 AI 内容生成工作台，封装在一个独立的 HTML 文件中。它集成了多家主流 AI 服务商的图像与视频生成 API，提供统一的交互界面，支持多区域节点切换、多语言（中/英）、历史收藏、日志追踪等功能。

本项目面向开发者和创作者，无需搭建任何后端服务，只需在浏览器中打开即可使用。

---

## 核心特性

| 特性 | 说明 |
|------|------|
| 🖼️ **图像生成** | 支持 GPT Image 2、DALL·E、Flux 及 Gemini 等模型 |
| ✏️ **图像编辑** | 上传源图与遮罩，进行局部重绘编辑 |
| 🎬 **视频生成** | 集成 Seedance 2.0 视频生成接口 |
|  **多区域节点** | 国内、海外（洛杉矶）、Gemini 原生三种节点可选 |
| 🌐 **双语界面** | 一键切换 中文 / English |
| 🗂️ **形态变体** | 批量生成多个姿态/场景变体，支持并发请求 |
| 🔲 **宫格切分** | 将大图按 2×2 或 3×3 切分为子图 |
| 💾 **本地收藏** | 基于 IndexedDB 的历史记录与收藏管理 |
| 📜 **请求日志** | 可折叠的右侧日志抽屉，记录所有 API 请求与响应 |
| 🖼️ **参考图上传** | 支持内容参考、色板参考两类参考图 |
| ⌨️ **快捷键支持** | `⌘/Ctrl + L` 快速打开日志 |

---

## 快速开始

### 1. 获取代码

```bash
git clone https://github.com/JING958/ai-studio.git
cd ai-studio
```

### 2. 直接打开

```bash
# 方式一：直接双击打开
ai-studio-UC.html

# 方式二：使用本地服务器（推荐，避免跨域限制）
npx serve .
# 或
python -m http.server 8080
```

> ⚠️ **注意**：由于浏览器安全策略，建议通过本地 HTTP 服务器打开，以确保图片下载、视频播放等功能正常工作。

### 3. 配置 API

在界面左侧边栏中填入：
- **API 地址**：你的图像/视频生成服务地址
- **API Key**：你的认证密钥（Bearer Token）
- **模型**：如 `gpt-image-2`、`doubao-seedance-2-0-260128` 等

---

## 功能详解

### 图像生成

1. 在 **Prompt** 输入框中描述你想要生成的图像
2. （可选）在 **形态变体** 中输入多个变体描述，每行一个
   - 支持 `{变体}` / `{variant}` 占位符替换
   - 逗号在同一行内不会分割变体
3. 选择 **尺寸**、**质量**、**格式**、**每变体张数**
4. （可选）上传 **参考图**（内容参考 / 色板参考）
5. 点击 **生成图像** 按钮

**高级选项：**
- **单角色独立形态**：自动追加提示词，强制单图单姿态，避免多宫格

### 图像编辑

切换到 **编辑** 模式：
1. 上传 **源图**（必填）
2. （可选）上传 **遮罩图**（透明区域为编辑区）
3. 输入编辑提示词
4. 点击生成

### 视频生成

切换到 **视频生成** 标签：
1. 配置提交地址与状态地址
2. 输入文本 Prompt（支持中英文，建议不超过 500 字）
3. （可选）上传首帧/尾帧图片、参考视频/音频 URL
4. 选择分辨率、时长、宽高比、是否生成音频等参数
5. 点击 **提交生成任务**，系统将自动轮询任务状态

### 切分工具

对生成的图片进行宫格切分：
1. 点击顶部 **切分工具** 标签
2. 选择来源图片（当前生成或历史收藏）
3. 选择输出尺寸：**等分**（每张为原图 1/N）或 **原图**（每张保持原图分辨率）
4. 点击 **2×2** 或 **3×3** 进行切分
5. 支持单张下载或收藏到历史

---

## 多区域节点支持

| 节点 | 说明 | 适用模型 |
|------|------|----------|
| 🇨🇳 **国内** | 中国大陆节点 | GPT Image、DALL·E、Flux |
| 🇺🇸 **海外** | 美国洛杉矶节点，访问更快，数据不回国 | GPT Image、DALL·E、Flux |
| 💎 **Gemini** | Google Gemini 原生图像生成 | Gemini Pro Image |

节点切换后会自动更新对应的 API 地址和默认模型。

---

## 本地存储

所有数据均存储在浏览器本地，不上传至任何服务器：

- **历史收藏**：使用 IndexedDB（`ai-studio` 数据库）存储图片和视频 Blob
- **界面状态**：使用 `localStorage` 保存语言设置、区域节点、历史面板展开状态等
- **API 配置**：当前未持久化，建议自行保存密钥（出于安全考虑）

---

## 技术实现

### 技术栈

- **HTML5**：语义化标签与结构化布局
- **CSS3**：CSS 变量主题、Grid / Flexbox 布局、过渡动画
- **Vanilla JavaScript**：无框架依赖，原生 DOM 操作
- **IndexedDB**：本地数据库用于历史收藏

### 架构设计

```
ai-studio-UC.html
├── 样式层（CSS Variables + Grid Layout）
│   ├── 暗色主题设计系统
│   └── 响应式断点支持
├── 结构层（HTML Semantics）
│   ├── Topbar（标签切换、区域选择）
│   ├── Sidebar（参数配置面板）
│   ├── Main（预览区域 + 历史面板）
│   └── Drawer（日志抽屉）
├── 交互层（JavaScript Modules）
│   ├── I18N（中英双语）
│   ├── Region（多节点管理）
│   ├── Image Mode（图像生成/编辑）
│   ├── Video Mode（视频生成/轮询）
│   ├── History（IndexedDB 收藏）
│   ├── Logs（请求日志记录）
│   └── Split Tool（宫格切分）
└── 数据层（Browser APIs）
    ├── Fetch API（HTTP 请求）
    ├── IndexedDB（本地存储）
    └── FileReader（图片预览）
```

### 关键设计

- **单文件架构**：所有代码（HTML/CSS/JS）内联在一个文件中，无需构建工具，直接部署
- **并发控制**：图像生成使用 `MAX_CONCURRENCY = 6` 限制并发请求数
- **轮询机制**：视频生成采用定时轮询（每 5 秒）查询任务状态
- **Blob URL 管理**：生成结果使用 `URL.createObjectURL()` 创建临时链接，切换时自动 `revokeObjectURL`

---

## 文件结构

```
.
├── ai-studio-UC.html          # 主程序文件（单文件应用）
├── README.md                   # 本白皮书
├── 宫格切分功能-实现文档.md     # 宫格切分功能详细文档
├── 形态变体功能-使用指南.md     # 形态变体功能使用指南
├── 网页实现-学习文档.md         # 前端实现学习笔记
├── api.md                      # API 接口文档
├── doubao-seedance-2-0.md      # Seedance 视频生成文档
├── gpt-image-2 API.md          # GPT Image 2 API 文档
└── ...                         # 其他辅助文档
```

---

## 使用示例

### 图像生成示例

**Prompt**: `a beautiful flower`

**形态变体**：
```
standing in a sunlit garden, full body
running on a beach at sunset, dynamic pose
sitting by a window reading, half body
```

**配置**：
- 尺寸：1024×1024
- 质量：高
- 格式：PNG
- 每变体张数：2

**结果**：将生成 6 张图片（3 个变体 × 2 张），展示不同姿态下的花朵。

### 视频生成示例

**Prompt**: `A cat playing with a ball of yarn in a cozy living room`

**配置**：
- 分辨率：720p
- 时长：5s
- 宽高比：16:9 横屏
- 音频：有声

---

## 常见问题

### Q: 为什么图片下载失败？
A: 请确保通过本地 HTTP 服务器打开页面，而非直接双击打开文件。浏览器安全策略会阻止 `file://` 协议下的某些操作。

### Q: 支持哪些图像模型？
A: 目前支持 `gpt-image-2`、`gpt-image-1`、`dall-e-3`、`dall-e-2`、`flux-pro`、`flux-dev`、`gemini-3-pro-image-preview`、`gemini-2.5-flash-image` 等。

### Q: 视频生成一直显示“轮询中”？
A: 请检查：
1. API Key 是否正确
2. 提交地址和状态地址是否匹配
3. 网络连接是否正常
4. 查看右侧日志抽屉获取详细错误信息

### Q: 历史收藏会占用多少空间？
A: 取决于 IndexedDB 的存储配额，通常浏览器允许每个域名存储数百 MB。可以在历史面板底部查看当前已用空间。

### Q: 如何清除所有数据？
A: 点击历史面板右上角的 **清空** 按钮，或在浏览器设置中清除 `ai-studio` 的 IndexedDB 数据。

---

## 许可协议

本项目采用 [MIT License](LICENSE) 开源协议。

---

<p align="center">
  Made with ️ by <a href="https://github.com/JING958">JING958</a>
</p>

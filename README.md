# Nano Article Cover

智能文章封面生成器，基于 Vue 3 + Vite + Tailwind CSS 开发。用户只需输入文章标题，即可通过 AI 自动生成精美的文章封面海报。

## 效果预览
![](./demo.png)

## ✨ 功能特性

- **智能生成**：根据输入的文章标题，自动构建 Prompt 并调用 AI 接口生成高质量封面图。
- **流式响应**：支持 API 流式响应解析，实时展示生成进度条。
- **响应式设计**：完美适配桌面端与移动端，提供流畅的用户体验。
- **图片预览**：支持点击生成结果进行全屏大图预览。
- **便捷下载**：提供“下载原图”功能，自动保存为 PNG 格式。
- **交互优化**：
  - 标题输入框支持一键清除。
  - 加载状态动画与错误提示。
  - 键盘 Enter 快捷提交。

## 🛠️ 技术栈

- **前端框架**：[Vue 3](https://vuejs.org/) (Composition API)
- **构建工具**：[Vite](https://vitejs.dev/)
- **开发语言**：[TypeScript](https://www.typescriptlang.org/)
- **样式库**：[Tailwind CSS v4](https://tailwindcss.com/)
- **HTTP 请求**：原生 `fetch` API (配合 `ReadableStream` 处理流式数据)

## 🚀 快速开始

### 1. 环境准备

确保您的本地环境已安装 [Node.js](https://nodejs.org/) (推荐 v16+)。

### 2. 安装依赖

```bash
npm install
```

### 3. 启动开发服务器

```bash
npm run dev
```

启动后访问 `http://localhost:5173/` 即可使用。

### 4. 构建生产版本

```bash
npm run build
```

构建产物将输出到 `dist` 目录。

## 📂 项目结构

```
.
├── public/              # 静态资源
├── src/
│   ├── assets/          # 图片等资源
│   ├── components/      # Vue 组件 (目前主要逻辑在 App.vue)
│   ├── App.vue          # 核心应用组件 (包含所有业务逻辑)
│   ├── main.ts          # 应用入口
│   └── style.css        # 全局样式 (Tailwind 引入)
├── index.html           # HTML 模板
├── package.json         # 项目依赖配置
├── postcss.config.js    # PostCSS 配置
├── tailwind.config.js   # Tailwind 配置
├── tsconfig.json        # TypeScript 配置
└── vite.config.ts       # Vite 配置
```

## 📝 注意事项

- 本项目调用了第三方 AI 绘图接口，请确保网络环境可以正常访问该 API。
- API Key 目前硬编码在源码中，生产环境建议通过环境变量或后端代理进行管理。

## 📧 联系方式

有问题联系：243027571@qq.com

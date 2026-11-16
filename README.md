# Paper Margin 网页版 + Agent插件

## 人工写的注意事项（必看）
- 1.最新版本：**20261116**
- 2.目前非常的初版，功能还比较简陋，欢迎填issue！
- 3.后续会在releases上传一个适用于DeepSeek Harness和Reasonix Studio的插件压缩包。

# （下面都是AI写的，看不看随你）

首版已实现：Vite + TypeScript、PDF.js 原页预览、本地 Tesseract OCR（英文 / 简体中文＋英文）、OpenAI-compatible Chat Completions、逐段中文讲解及可选译文、全文引用总结、拖动分栏、任务 JSON 保存/恢复、离线 HTML 导出。

## 本地运行

```powershell
cd web
npm ci
npm run assets
npm run dev
```

浏览器打开终端显示的地址。生产构建：

```powershell
npm run build
npm run preview
```

不要双击源码 index.html；网页应用及 OCR worker 需要 HTTP(S) 环境。导出的阅读 HTML 可以直接离线打开。

## 使用

1. 选择不超过30页、30 MiB 的 PDF，并明确选择是否附中文译文。
2. 选择扫描页语言。OCR 在浏览器本地运行，不向模型发送页面图片。首次会从本站加载 worker、WASM 与语言包，资源较大。
3. 点击解析，核对原页与扫描页识别情况。文字块由几何启发式合并，复杂多栏、表格和公式不保证正确。
4. 填 Base URL、模型 ID、API Key，可先测试连接。Base URL 例：`https://api.deepseek.com/v1`；程序追加 `/chat/completions`。具体模型 ID 以服务商为准。
5. 勾选发送确认，再开始精读。只发送原文块、局部上下文及已生成的总结依据，不发送 PDF 图片。实际费用取决于模型、文本量、输出和重试。
6. 可取消，已完成批次保留在内存。导出任务 JSON 后可在下次恢复。解析中断不能当作全文完成，需重新完整解析。
7. 完成全部批注与总结后可导出自包含离线 HTML。分界线可拖动、方向键调整、双击重置。

模型服务需支持浏览器 CORS。CORS 拒绝不能在纯 GitHub Pages 内绕过；本应用不会把密钥发送到公共代理。没有真实 Key 测试不能证明某个服务商可用。

## GitHub Pages 发布

本目录尚未关联 GitHub 仓库，未执行上传或发布。

方式一：在现有仓库的 CI 中运行 `npm ci`、`npm run assets`、`npm run build`（工作目录指向 web），将 `web/dist` 作为 Pages artifact 发布。Pages 设置选择 GitHub Actions。

方式二：把 `dist` 内容放入专用于 Pages 的分支或发布目录，并按 GitHub 仓库设置指定该目录。不要上传 `.env`、Key、任务 JSON 或用户论文。

`vite.config.ts` 使用相对 `base: './'`，适用于仓库子路径。发布产物必须包含 `assets/` 与 `ocr/`，不能只上传 index.html。OCR 全部资源较大，首次加载和移动端内存需要留意。`public/ocr` 是 `npm run assets` 生成目录，无需提交源仓库。

## 隐私与数据

- Key 只在当前页面输入值中，不写 localStorage、任务 JSON 或导出 HTML。
- 没有自动保存论文；刷新/关页会失去当前内存任务，离开页面会提示。
- 任务文件包含论文图片、文字和批注，需自己妥善保管。
- 模型内容被转义为文本，不执行返回的 HTML、脚本或工具指令。恢复文件会校验页面、坐标、段落和引用。
- 仅允许 HTTPS API（本机开发 localhost HTTP 例外），禁止跨源重定向；更换 Base URL 后需重新确认发送。
- PDF 与 OCR 本地处理不等于全程离线：精读会把文本发送至用户配置的服务商。

## 验证与限制

2026-11-16：TypeScript 与 Vite 生产构建通过；npm audit 报告0个已知漏洞（不代表不存在漏洞）。Chromium 实测文本 PDF 解析、模拟 API 连接/批注/总结、无 Key 的任务导出、离线 HTML 定位、390px 无横向溢出，以及真实英文扫描 PDF 的本地 Tesseract OCR。

未使用付费 API Key；没有验证真实模型质量、中英文混合扫描精度、复杂公式、多栏阅读顺序、全部30页资源上限、Safari/Firefox、真实 GitHub Pages 域名和截图像素级审美。OCR 不支持承诺准确数学公式或所有图内文字；已带文本层但含局部图片文字的混合页不会自动整页 OCR。

对长响应、模型不合规 JSON 或引文不匹配，本批次不会写入，可重新开始继续；对429/5xx最多尝试3次。请求超时120秒。严格 JSON 模式不是所有服务都兼容，因此使用提示与本地校验，不保证每个模型都一次通过。

首版没有流式输出、自动 IndexedDB 论文存储、云 OCR、Responses 协议或多用户后端。所有依赖按其自身许可证分发，请保留第三方许可并在商业发布前审查。

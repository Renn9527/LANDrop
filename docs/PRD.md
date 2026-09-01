# LANDrop 2.0 产品需求文档 (PRD)

## 1. 产品定位与目标

LANDrop 是一个面向局域网的极简、轻量、高安全性的跨平台传输工具（Desktop Utility / Local Transfer Tool）。
- **目标用户**：需要在同局域网（家庭 Wi-Fi、办公室 LAN、热点）下快速传输大文件、文件夹、多文件、长文本、剪贴板的多设备用户（Windows、macOS、Linux、iOS、Android 等）。
- **核心价值**：零配置、无需外网/云端中转、单可执行文件开箱即用、无营销与 AI 视觉噪音、稳定可靠的高速内网传输。

## 2. 核心架构与技术边界

- **后端**：Go 标准库为主，内置 HTTP Server、mDNS 服务广播与发现、SSE（Server-Sent Events）单向事件流、单二进制发布（`go:embed`）。
- **前端**：Preact + TypeScript + Vite，纯原生 CSS Variables (Tokens)，零重型 UI 库依赖，单页应用架构（SPA）。
- **网络与安全**：纯局域网 Direct HTTP/HTTPS，支持 4 位临时 PIN 保护、One-Time 阅后即焚、TTL 超时自动清理、路径穿透与文件名安全校验。

## 3. 功能模块与用户心智模型

1. **统一发送 (Send Panel)**：
   - 文件队列 (File Queue)：支持拖拽/多选，选后进入待发列表，点击“发送”按需上传，支持进度条、速度与 ETA 计算。
   - 文本传输 (Text Editor)：快速输入或粘贴文本，实时显示大小，支持 URL 快速识别。
   - 剪贴板中转 (Clipboard Panel)：读取/推送剪贴板内容，支持浏览器与服务端双向交互。
2. **设备发现与连接 (Nearby Devices)**：
   - mDNS 自动发现局域网对端设备，点击直接切换目标设备。
   - 手动连接 (Manual Connect)：支持直接输入 IP:Port。
   - 手机扫码 (QR Connect)：弹窗展示直连二维码，手机扫码免配置打开。
3. **活动记录 (Activity)**：
   - 历史传输按天分组（今天/昨天/日期），支持文件名/对端/文本内容快速搜索与类型过滤。
4. **系统设置 (Settings)**：
   - 设备名称自定义并实时广播更新。
   - 主题模式 (System / Light / Dark)。
   - 语言切换 (简体中文 / English)。

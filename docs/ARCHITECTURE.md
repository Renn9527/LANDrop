# LANDrop 2.0 架构说明文档 (ARCHITECTURE)

## 1. 总体架构概览

LANDrop 采用单进程 + 内置轻量 Web GUI 架构，提供纯内网点对点通信：

```text
+-----------------------------------------------------------------+
|                         Browser / Web UI                        |
|  (Preact + TypeScript + CSS Tokens + Vite Build -> go:embed)    |
+-------------------------------+---------------------------------+
                                | HTTP REST / SSE Stream
+-------------------------------v---------------------------------+
|                       Go Backend Server                         |
|  +------------------+  +------------------+  +---------------+  |
|  |   HTTP Handler   |  |    SSE Broker    |  | mDNS Manager  |  |
|  |  (routes / SPA)  |  |   (real-time)    |  |  (discovery)  |  |
|  +--------+---------+  +--------+---------+  +-------+-------+  |
|           |                     |                    |          |
|  +--------v---------+  +--------v---------+          |          |
|  |  Transfer Store  |  |  Clipboard Mgr   |          |          |
|  | (state/stream)   |  | (push/get/watch) |          |          |
|  +--------+---------+  +------------------+          |          |
|           |                                          |          |
|  +--------v---------+  +------------------+          |          |
|  | Local Temp Disk  |  | Config / History |          |          |
|  |   (/tmp/landrop) |  |   (JSON Store)   |          |          |
|  +------------------+  +------------------+          |          |
+-----------------------------------------------------------------+
```

## 2. 后端主要模块与职责

| 文件 | 职责说明 |
| :--- | :--- |
| `main.go` | CLI 命令入口与路由分发（`serve`, `send`, `recv`, `devices`, `clipboard`, `history`, `version`），CLI 端文件/文本收发逻辑。 |
| `server.go` | HTTP/HTTPS Server 生命周期管理、可用端口探测、本机 IP 检测、优雅退出与信号处理、后台 TTL 清理。 |
| `handler.go` | HTTP 路由注册（`SetupRoutes`）、Web UI embed 静态资源分发与 SPA fallback、文件/文本收发 HTTP Handler、Range 下载处理。 |
| `app_api.go` | 设置管理 API (`/settings`)、传输历史 API (`/history`)、过滤与数据转换。 |
| `transfer.go` | 核心传输存储模型（`TransferStore`, `TransferItem` / `Transfer`）、临时文件存储与流式落盘、下载状态机与单次消费。 |
| `events.go` | Server-Sent Events (SSE) 事件代理 (`SSEBroker`)，向 Web 端与 CLI 订阅者实时推送广播通知与传输进度。 |
| `mdns.go` | 基于 mDNS/DNS-SD 的局域网设备服务广播 (`_landrop._tcp`) 与设备发现管理。 |
| `clipboard.go` / `clipboard_*.go` | 剪贴板内容暂存、SSE 广播、本地系统剪贴板监听与推送。 |
| `history.go` | 传输历史记录持久化 (`history.json`)，支持最大 500 条记录与清空操作。 |
| `settings.go` | 应用持久化配置 (`config.json`)，支持设备名规范化与设置读取/写入。 |
| `pin.go` | 4 位数字 PIN 认证中间件与验证逻辑。 |
| `tls.go` | 自签名 SSL/TLS 证书生成，支持 HTTPS 安全传输模式。 |
| `qrcode.go` | 终端 ASCII 二维码生成与 HTTP 二维码图片渲染。 |

## 3. 前端模块职责与架构

```text
web/src/
├── main.tsx             # 应用挂载入口
├── app.tsx              # 主应用组件与路由切换 (/ , /activity , /settings)
├── api/                 # 统一 API 客户端（无散落 fetch）
│   ├── client.ts        # 基础 fetch 封装与错误处理
│   ├── info.ts          # /info 服务端状态与模式
│   ├── devices.ts       # /devices 局域网设备列表
│   ├── history.ts       # /history 历史记录与清空
│   ├── settings.ts      # /settings 配置读取与修改
│   └── transfers.ts     # 传输发起、上传、取消、下载
├── state/               # 响应式状态管理
│   ├── app.ts           # 全局配置、语言、主题、网络状态
│   ├── devices.ts       # 设备在线状态、设备列表
│   └── transfers.ts     # 待发送队列、活跃传输与进度
├── hooks/               # 自定义 Hooks
│   └── useEvents.ts     # SSE 实时连接与事件分发、断线重连
├── components/          # 原子 UI 组件与通用组件
│   ├── Button.tsx
│   ├── IconButton.tsx
│   ├── Input.tsx
│   ├── Tabs.tsx
│   ├── Dialog.tsx
│   ├── Toast.tsx
│   ├── EmptyState.tsx
│   ├── ProgressBar.tsx
│   ├── Header.tsx
│   ├── SendPanel.tsx
│   ├── FilePicker.tsx
│   ├── FileQueue.tsx
│   ├── TextEditor.tsx
│   ├── ClipboardPanel.tsx
│   ├── DeviceList.tsx
│   ├── DeviceRow.tsx
│   ├── ManualConnectDialog.tsx
│   ├── QRDialog.tsx
│   ├── TransferList.tsx
│   └── TransferRow.tsx
├── pages/               # 页面级组件
│   ├── Home.tsx         # 发送 + 设备 + 最近传输
│   ├── Activity.tsx     # 完整历史与筛选
│   └── Settings.tsx     # 系统设置与偏好
├── locales/             # 双语本地化包
│   ├── zh-CN.ts
│   ├── en.ts
│   └── index.ts
└── styles/              # 纯 CSS Tokens 样式体系
    ├── tokens.css       # 色彩、间距、圆角变量
    ├── reset.css        # 现代化基础 Reset
    ├── base.css         # 全局文字、控件样式
    ├── layout.css       # 响应式 Grid/Flex 布局
    └── components.css   # 原子组件样式
```

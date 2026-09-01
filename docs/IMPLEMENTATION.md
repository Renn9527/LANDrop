# LANDrop 2.0 Implementation Plan

> 目标：将 LANDrop 从单文件 Web Demo 重构为可维护、轻量、成熟的局域网传输工具  
> 执行方式：按 Phase 顺序完成，不允许跳过基础阶段直接重写核心协议

---

# 0. 总体执行原则

本次重构必须遵守以下原则。

## 0.1 不一次性推倒重做

禁止：

```text
一次性重写整个项目
一次性替换所有 API
一次性重写 CLI
一次性改传输协议
一次性替换 mDNS
引入 WebRTC
```

必须：

```text
每一个 Phase 都可独立运行
每一个阶段结束后项目必须可发布
每次修改保持现有功能尽量可用
```

---

# 0.2 保留核心架构

必须继续保留：

```text
Go
HTTP
mDNS
SSE
CLI
Go embed
单二进制发布
```

最终用户仍然应该只需要：

```text
landrop.exe
```

或：

```text
./landrop
```

---

# 0.3 前端设计硬性约束

禁止：

```text
Tailwind
shadcn/ui
Material UI
Ant Design
Bootstrap
Framer Motion

渐变背景
玻璃拟态
backdrop-filter
大面积阴影
大圆角 Card
Bento Grid
Hero
蓝紫渐变
Emoji 功能图标
999px 普通按钮
营销型副标题
```

LANDrop 是：

```text
Desktop Utility
System Tool
Local Transfer Tool
```

不是：

```text
SaaS Dashboard
Landing Page
AI Demo
```

---

# 0.4 每阶段必须验证

后端：

```bash
go test ./...
```

前端：

```bash
npm run typecheck
npm run build
```

最终：

```bash
go build .
```

Windows：

```powershell
go build -o landrop.exe .
```

---

# 1. Phase 0 — Baseline & Safety Net

目标：

> 在修改任何 UI 或核心代码前建立可回归基线。

---

## Task 0.1 — 确认现有功能

手工验证：

```text
landrop serve
```

浏览器访问后确认：

- 首页可打开
- 文件上传正常
- 文本发送正常
- QR 可加载
- Devices 可读取
- History 可读取
- Settings 可保存
- SSE 可连接
- Clipboard API 不报致命错误

---

## Task 0.2 — 运行现有测试

执行：

```bash
go test ./...
```

记录当前：

```text
PASS
FAIL
Skipped
```

如果当前已有失败：

不要在 UI 重构时顺手大规模修复。

先记录为：

```text
Known Baseline Issue
```

---

## Task 0.3 — 建立开发文档

新增：

```text
docs/
├── IMPLEMENTATION.md
└── ARCHITECTURE.md
```

当前文件保存为：

```text
IMPLEMENTATION.md
```

---

## Task 0.4 — ARCHITECTURE 初稿

记录：

```text
main.go
server.go
handler.go
transfer.go
events.go
mdns.go
clipboard.go
history.go
settings.go
pin.go
tls.go
qrcode.go
```

职责。

不要改代码。

---

## Phase 0 验收

必须满足：

```text
go test ./...
```

结果已记录。

并能够启动：

```bash
go run . serve
```

---

# 2. Phase 1 — Frontend Bootstrap

目标：

> 建立新的 Preact + TypeScript + Vite 前端，但暂不删除旧 UI。

版本建议：

```text
v1.3-dev
```

---

# 2.1 新建前端工程

目录：

```text
web/
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/
└── dist/
```

---

## Task 1.1 — 初始化依赖

仅安装必要依赖：

```text
preact
vite
typescript
@preact/preset-vite
```

如非必要，不增加其他 dependency。

---

## Task 1.2 — package.json

至少提供：

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "typecheck": "tsc --noEmit"
  }
}
```

---

## Task 1.3 — Vite Build

目标：

```text
web/dist/index.html
web/dist/assets/*
```

必须支持相对或根路径资源，使 Go Server 可正常提供。

---

# 2.2 建立目录

创建：

```text
web/src/
├── main.tsx
├── app.tsx
│
├── api/
├── components/
├── pages/
├── state/
├── hooks/
├── utils/
├── locales/
└── styles/
```

---

# 2.3 建立 Design Tokens

创建：

```text
web/src/styles/tokens.css
```

基础：

```css
:root {
  --bg: #f6f6f4;

  --surface: #ffffff;
  --surface-hover: #f2f2f0;

  --text: #181818;
  --text-secondary: #727272;

  --border: #e4e4e1;
  --border-strong: #cececa;

  --accent: #3267d6;

  --success: #27845a;
  --warning: #a86c17;
  --danger: #c43d3d;
}
```

---

# 2.4 建立 Base CSS

创建：

```text
reset.css
base.css
layout.css
components.css
```

要求：

```text
系统字体
无 Web Font
无 CDN
无 gradient
无 backdrop-filter
普通元素无 shadow
```

---

# 2.5 Dark Mode

第一阶段可以只做 CSS Token。

使用：

```css
@media (prefers-color-scheme: dark)
```

暂不做设置里的 Theme 切换。

---

# 2.6 创建 UI 基础组件

必须创建：

```text
Button.tsx
IconButton.tsx
Input.tsx
Tabs.tsx
Dialog.tsx
Toast.tsx
EmptyState.tsx
ProgressBar.tsx
```

---

## Button 规范

支持：

```text
primary
secondary
danger
ghost
```

禁止：

```text
gradient button
pill button
```

圆角：

```text
8px
```

---

## Dialog

必须：

- Escape 关闭
- 点击遮罩可关闭
- Focus trap
- 默认 focus
- aria-modal
- role=dialog

---

## Toast

仅实现：

```text
success
error
info
```

不做复杂动画。

---

# Phase 1 验收

执行：

```bash
cd web
npm run typecheck
npm run build
```

要求：

```text
0 TypeScript errors
dist 正常生成
```

旧 `desktop.html` 仍然保留。

---

# 3. Phase 2 — API Client Layer

目标：

> 前端不允许直接在各组件中散落 fetch。

---

# 3.1 创建统一 Client

文件：

```text
web/src/api/client.ts
```

实现：

```ts
apiGet()
apiPost()
apiDelete()
apiPut()
```

统一处理：

- JSON
- HTTP error
- timeout 可选
- error object

---

# 3.2 API Error

定义：

```ts
type ApiError = {
  status: number
  code?: string
  message: string
}
```

组件不直接判断：

```text
response.status === xxx
```

---

# 3.3 Info API

创建：

```text
api/info.ts
```

读取：

```text
GET /info
```

映射：

```ts
type AppInfo = {
  name: string
  version: string
  os: string
  addr: string
  one_time: boolean
}
```

---

# 3.4 Devices API

```text
api/devices.ts
```

读取：

```text
GET /devices
```

---

# 3.5 History API

```text
api/history.ts
```

支持：

```text
GET /history
DELETE /history
```

---

# 3.6 Settings API

```text
api/settings.ts
```

支持现有：

```text
GET /settings
POST /settings
```

---

# 3.7 Transfer API

```text
api/transfers.ts
```

暂时封装旧 API：

```text
POST /send/file
POST /send/text
GET /preview/:token
GET /recv/:token
```

不要此阶段新增 V2 API。

---

# Phase 2 验收

禁止出现：

```text
Home.tsx 内大量 fetch()
DeviceRow.tsx 内 fetch()
Settings.tsx 内 fetch()
```

所有请求必须进入：

```text
src/api/
```

---

# 4. Phase 3 — App State & SSE

目标：

> 建立基础 App State 和实时更新机制。

---

# 4.1 App State

创建：

```text
state/app.ts
```

维护：

```text
appInfo
language
connectionState
```

---

# 4.2 Device State

```text
state/devices.ts
```

维护：

```text
devices
loading
lastSync
```

提供：

```text
setDevices()
upsertDevice()
markOffline()
```

---

# 4.3 Transfer State

```text
state/transfers.ts
```

第一阶段维护前端传输状态：

```ts
type FrontendTransfer = {
  id: string
  type: 'file' | 'text'
  name?: string
  size: number
  status:
    | 'pending'
    | 'uploading'
    | 'ready'
    | 'completed'
    | 'failed'
  progress?: number
  url?: string
}
```

这是前端临时模型。

后续由 Go Transfer V2 替换。

---

# 4.4 SSE Hook

创建：

```text
hooks/useEvents.ts
```

封装：

```text
EventSource('/events')
```

监听现有：

```text
file_ready
device_found
device_lost
clipboard
settings_updated
done
```

---

# 4.5 SSE Reconnect

EventSource 自带基础 reconnect。

但在：

```text
open
```

重新成功连接后：

执行：

```text
refresh devices
refresh history
```

避免只依赖增量事件。

---

# 4.6 低频 Device Fallback

当前 5 秒一次 polling 改为：

```text
30 seconds
```

只作为 fallback。

---

# Phase 3 验收

断开 Wi-Fi 再恢复后：

- SSE 能恢复
- Devices 能重新同步
- History 能重新获取
- 页面无需刷新

---

# 5. Phase 4 — New App Shell

目标：

> 建立新的首页骨架。

---

# 5.1 Header

创建：

```text
components/Header.tsx
```

桌面：

```text
LANDrop                  LANDrop-PC ●     Settings     More
```

禁止显示永久 IP 卡片。

---

# 5.2 Device Menu

点击：

```text
LANDrop-PC
```

弹出：

```text
LANDrop-PC

Windows
192.168.1.10:53217

● 局域网可访问

复制地址
手机连接
```

---

# 5.3 Main Layout

桌面：

```text
Main Content
+
Sidebar
```

建议：

```css
grid-template-columns: minmax(0, 2fr) minmax(260px, 1fr);
```

---

# 5.4 首页区域

首页只允许出现：

```text
Send Panel
Nearby Devices
Recent Transfers
```

禁止首页重新加入：

```text
独立 QR Card
独立 CLI Card
独立 History Card
独立 Received Card
```

---

# Phase 4 验收

1440×900 下：

首页不滚动或仅少量滚动即可看到：

```text
Send
Devices
Recent Transfers
```

---

# 6. Phase 5 — Send Panel

目标：

> 统一文件、文本、剪贴板发送入口。

---

# 6.1 SendPanel

创建：

```text
components/SendPanel.tsx
```

顶部：

```text
发送到 LANDrop-PC
```

Tabs：

```text
文件
文本
剪贴板
```

---

# 6.2 Tabs State

切换 Tab 不刷新页面。

保持每个 Tab 的临时内容。

例如：

用户文件 Queue 不因切换到文本后消失。

---

# 7. Phase 6 — File Picker & Queue

目标：

> 改变当前“选完立刻上传”的行为。

---

# 7.1 FilePicker

创建：

```text
FilePicker.tsx
```

支持：

```text
点击选择
拖拽
多文件
```

---

# 7.2 Queue

创建：

```text
FileQueue.tsx
```

数据：

```ts
type PendingFile = {
  id: string
  file: File
  name: string
  size: number
}
```

---

# 7.3 Queue UI

例如：

```text
3 个文件 · 428 MB

photo.jpg                   4.2 MB      ×
video.mp4                 421.3 MB      ×
notes.pdf                   2.8 MB      ×

+ 添加文件

清空                         发送
```

---

# 7.4 不立即上传

选择文件时：

禁止调用：

```text
POST /send/file
```

只有点击：

```text
发送
```

之后才开始。

---

# 7.5 Upload Progress

现有后端继续使用：

```text
XMLHttpRequest
```

以获取 upload progress。

封装到：

```text
api/transfers.ts
```

不要写在组件。

---

# 7.6 上传中

UI：

```text
video.mp4

上传中

██████████░░░░ 72%

304 MB / 421 MB
```

---

# 7.7 上传完成

显示：

```text
已准备

复制链接
二维码
```

不要占据巨大 Result Card。

---

# 7.8 文件上限

Phase 6 暂时仍遵守后端 2GB。

但 UI 不要把：

```text
Max 2 GB
```

写成大提示。

仅选择超过限制时显示错误。

---

# Phase 6 验收

测试：

```text
1 file
5 files
中文文件名
0 byte
拖拽
取消 Queue item
清空
upload failure
```

---

# 8. Phase 7 — Text Panel

创建：

```text
TextEditor.tsx
```

---

# 8.1 UI

```text
文本

[ textarea ]

368 B

从剪贴板粘贴              发送
```

---

# 8.2 Size

使用：

```ts
new Blob([text]).size
```

显示 Byte 大小。

---

# 8.3 Button State

发送按钮：

```text
空文本 → disabled
超过 max → disabled
```

---

# 8.4 URL 检测

第一阶段只做：

```text
http://
https://
```

检测。

如果是 URL：

显示轻量：

```text
链接
github.com
```

不请求外部 Metadata。

---

# Phase 7 验收

测试：

```text
中文
emoji
URL
多行
10MB boundary
```

---

# 9. Phase 8 — Clipboard Panel

创建：

```text
ClipboardPanel.tsx
```

---

# 9.1 移除 Header Clipboard Emoji

删除旧：

```text
📋
```

---

# 9.2 当前剪贴板

UI：

```text
当前剪贴板

npm run build

刷新                       发送
```

---

# 9.3 Browser Clipboard

优先：

```text
navigator.clipboard
```

权限失败时：

```text
无法读取浏览器剪贴板
```

不要弹大错误页。

---

# 9.4 Server Clipboard

继续兼容：

```text
GET /clipboard
POST /clipboard/push
```

根据当前已有行为接入。

---

# Phase 8 验收

必须验证：

```text
secure context
non-secure LAN
clipboard permission denied
empty clipboard
```

---

# 10. Phase 9 — Nearby Devices

目标：

> Nearby Devices 成为首页核心辅助区域。

---

# 10.1 DeviceList

创建：

```text
DeviceList.tsx
DeviceRow.tsx
```

---

# 10.2 Device Row

显示：

```text
● macOS 设备
  192.168.1.20
```

IP 使用次级字体。

---

# 10.3 排序

优先：

```text
online
↓
last_seen
↓
name
```

---

# 10.4 点击设备

保持当前技术行为：

```text
打开目标 LANDrop 地址
```

例如：

```text
http://192.168.1.20:53217
```

可以：

```text
same tab
```

优先于新 Tab。

因为用户逻辑是在：

```text
切换目标设备
```

而不是浏览外链。

---

# 10.5 Offline Device

离线设备：

```text
降低视觉权重
显示最近在线时间
不可误导为当前在线
```

---

# 10.6 Manual Connect

创建：

```text
ManualConnectDialog.tsx
```

入口：

```text
+ 手动连接
```

---

# 10.7 Address Validation

支持：

```text
192.168.1.10:53217
http://192.168.1.10:53217
https://...
hostname.local:53217
```

禁止执行：

```text
javascript:
data:
file:
```

---

# Phase 9 验收

测试：

```text
0 devices
1 device
10 devices
online/offline
manual address
invalid address
mDNS unavailable
```

---

# 11. Phase 10 — QR Dialog

创建：

```text
QRDialog.tsx
```

---

# 11.1 删除 QR Card

旧首页：

```text
Scan Card
```

完全删除。

---

# 11.2 新入口

使用：

```text
手机连接
```

---

# 11.3 Dialog

显示：

```text
连接 LANDrop-PC

QR

192.168.1.10:53217

复制地址
```

QR 继续使用：

```text
/qr?size=...
```

---

# Phase 10 验收

手机扫描后必须进入当前 LANDrop 页面。

---

# 12. Phase 11 — Recent Transfers

目标：

> 合并 Received 与 Recent History 心智模型。

---

# 12.1 TransferList

创建：

```text
TransferList.tsx
TransferRow.tsx
```

---

# 12.2 首页限制

最多：

```text
5 records
```

底部：

```text
查看全部
```

跳转：

```text
/activity
```

---

# 12.3 Row 类型

支持：

```text
file
text
```

---

# 12.4 状态

第一阶段映射：

```text
success
failed
interrupted
uploading
ready
```

---

# 12.5 不使用大 Card

每条 Transfer 使用：

```text
row
separator
```

---

# Phase 11 验收

首页至少能正确显示：

```text
完成
失败
中断
进行中
文本
文件
```

---

# 13. Phase 12 — Activity Page

目标：

> 将历史从首页管理面板迁出。

---

# 13.1 Route

如果不引入 Router Library：

使用极简 path switch。

例如：

```ts
window.location.pathname
```

支持：

```text
/
/activity
/settings
```

不为三个页面引入大型 Router。

---

# 13.2 Activity 页面

文件：

```text
pages/Activity.tsx
```

---

# 13.3 分组

按：

```text
今天
昨天
日期
```

分组。

---

# 13.4 搜索

支持：

```text
filename
peer
text
```

---

# 13.5 Filter

默认隐藏。

点击：

```text
筛选
```

展开：

```text
类型
方向
状态
设备
```

---

# 13.6 Clear History

放：

```text
More
→ Clear
```

必须 Confirm。

---

# Phase 12 验收

历史为空时：

```text
还没有传输记录
```

不能显示一堆禁用 filter controls。

---

# 14. Phase 13 — Settings Page

目标：

> 先建立完整 Settings UI 框架，只对已有后端能力真正保存。

---

# 14.1 Page

创建：

```text
pages/Settings.tsx
```

---

# 14.2 Sections

```text
设备
接收
分享
安全
外观
数据
高级
```

---

# 14.3 Phase 13 可实际修改

当前只真正支持的设置先接：

```text
device_name
```

---

# 14.4 未实现 Setting

如果后端还没支持：

显示为：

```text
disabled
```

或暂时不展示。

禁止：

```text
前端看似可保存
实际上后端无效果
```

---

# 14.5 Theme

可以优先在前端 LocalStorage 实现：

```text
system
light
dark
```

但要明确这是 UI 设置。

---

# 14.6 Language

支持：

```text
zh-CN
en
```

保存：

```text
localStorage
```

后续再同步后端。

---

# Phase 13 验收

刷新浏览器后：

```text
theme
language
```

仍保持。

Device Name 后端持久化正常。

---

# 15. Phase 14 — i18n Refactor

目标：

> 删除旧 desktop.html 中巨型 I object。

---

# 15.1 Locale Files

创建：

```text
locales/zh-CN.ts
locales/en.ts
```

---

# 15.2 文案 Key

避免：

```text
sf
sth
dv
rch
```

这种缩写。

使用：

```text
send.file
send.text
devices.title
history.empty
```

---

# 15.3 禁止营销文案

翻译文件里同样禁止：

```text
Seamlessly
Effortlessly
Powerful
Modern
```

---

# Phase 14 验收

所有可见文本必须来自：

```text
locale
```

或真正无需翻译的：

```text
LANDrop
IP
URL
```

---

# 16. Phase 15 — Mobile Layout

目标：

> 真正重新组织手机 UI。

---

# 16.1 Breakpoint

推荐：

```text
< 720px
```

进入移动布局。

---

# 16.2 Mobile Header

```text
LANDrop          LANDrop-PC
```

---

# 16.3 Sidebar

Nearby Devices 不再永久占右栏。

可以：

```text
设备按钮
→ Drawer / Page
```

---

# 16.4 Bottom Navigation

移动端：

```text
首页
传输
设备
设置
```

如果“设备”没有独立 URL：

可以通过：

```text
/?panel=devices
```

或 Drawer。

第一版可以：

```text
首页
传输
设置
```

设备放首页按钮。

---

# 16.5 Transfer Rows

移动端不得横向溢出。

长文件名：

```text
ellipsis
```

但可以点击查看完整。

---

# 16.6 Touch Target

所有主要操作：

```text
min-height: 44px
```

---

# Phase 15 验收

必须检查：

```text
320×568
375×667
390×844
430×932
```

---

# 17. Phase 16 — Go Embed Migration

目标：

> 正式让 Go 使用新的 Vite Build。

---

# 17.1 Embed

从：

```text
web/desktop.html
```

迁移为：

```text
web/dist/*
```

---

# 17.2 Static Assets

Go Server 正确处理：

```text
/
assets/*.js
assets/*.css
```

---

# 17.3 SPA Fallback

仅对：

```text
/
/activity
/settings
```

返回 `index.html`。

API：

```text
/api
/info
/devices
...
```

不能被 SPA fallback 吞掉。

---

# 17.4 Cache

`index.html`：

```text
no-cache
```

hashed assets：

可以：

```text
Cache-Control: public, max-age=31536000, immutable
```

---

# Phase 16 验收

执行：

```bash
cd web
npm run build
cd ..
go build .
```

然后：

```bash
./landrop serve
```

不依赖：

```text
node
npm
dev server
```

运行。

---

# 18. Phase 17 — Remove Legacy Frontend

只有新 UI 完整可用后执行。

---

# 18.1 删除

```text
web/desktop.html
```

---

# 18.2 删除旧 JS/CSS

所有旧：

```text
inline CSS
inline JS
inline i18n
```

必须消失。

---

# Phase 17 验收

仓库中不得存在：

```text
一个几万行/几十 KB 的单页面 UI 文件
```

---

# 19. Phase 18 — Transfer Core V2

版本建议：

```text
v1.4
```

此阶段开始重构 Go Transfer。

---

# 19.1 新状态模型

定义：

```go
type TransferStatus string
```

状态：

```text
pending
ready
transferring
completed
failed
interrupted
cancelled
expired
```

---

# 19.2 新 Transfer Struct

建立：

```go
type Transfer struct {
    ID               string
    Token            string
    Direction        string
    Type             string
    Name             string
    Size             int64
    CreatedAt        int64
    ExpiresAt        int64
    Status           TransferStatus
    Peer             string
    BytesTransferred int64
    DownloadLimit    int
    DownloadCount    int
}
```

---

# 19.3 向后兼容

现有：

```text
TransferItem
```

不要立刻删除。

第一阶段可以：

```text
Transfer wrapper
```

逐步迁移。

---

# 20. Phase 19 — TTL & Cleanup

---

# 20.1 TTL

真正使用：

```text
ExpiresAt
```

---

# 20.2 默认 TTL

推荐：

```text
Web share = 1 hour
```

如果不想改变旧行为：

第一版可以：

```text
current session
```

但必须有真实 cleanup。

---

# 20.3 Cleanup Worker

启动 server 时：

```go
go cleanupLoop()
```

建议：

```text
60 seconds
```

---

# 20.4 Cleanup

删除：

```text
expired transfer
expired temp file
consumed one-time file
```

---

# 20.5 Shutdown

Server shutdown：

必须 cleanup 临时目录。

---

# Phase 19 验收

测试：

```text
TTL 5 seconds test-only
expired token returns Gone / Not Found
temp file removed
```

---

# 21. Phase 20 — File Storage Refactor

目标：

> 文件不再大量进入 RAM。

---

# 21.1 Threshold

从：

```text
100 MB
```

改为：

```text
<= 1 MB
```

或文件统一 tempfile。

推荐：

```text
Text → RAM
File → Temp
```

---

# 21.2 Streaming Upload

浏览器上传：

```text
multipart
↓
temporary file
```

避免：

```text
io.ReadAll()
```

读取大文件。

---

# 21.3 Disk Error

必须处理：

```text
disk full
permission
temp creation failure
```

---

# Phase 20 验收

上传：

```text
500 MB
2 GB+
```

时 LANDrop 内存不应随文件大小线性增长。

---

# 22. Phase 21 — Remove 2GB Artificial Limit

---

# 22.1 后端

移除：

```text
maxFileSize = 2GB
```

硬编码限制。

---

# 22.2 Multipart

确保：

```text
http.MaxBytesReader
```

不再限制到 2GB。

如果保留保护：

改为：

```text
configurable limit
```

---

# 22.3 前端

删除：

```text
2 GB max
```

限制逻辑。

---

# 22.4 测试

至少测试：

```text
>2 GB
>4 GB
```

可以使用 sparse file。

Linux/macOS：

```bash
truncate -s 5G test.bin
```

Windows 可使用：

```powershell
fsutil file createnew test.bin 5368709120
```

---

# 23. Phase 22 — Transfer Progress V2

---

# 23.1 Bytes

后端记录：

```text
BytesTransferred
```

---

# 23.2 SSE

新增：

```text
transfer.progress
```

建议不要每个 chunk 都广播。

节流：

```text
200~500ms
```

---

# 23.3 Speed

前端计算：

```text
delta bytes / delta time
```

---

# 23.4 ETA

```text
remaining / speed
```

低速或速度不稳定时：

允许隐藏 ETA。

---

# Phase 22 验收

UI：

```text
4.2 GB / 8.7 GB
48 MB/s
约 1 分 28 秒
```

---

# 24. Phase 23 — Cancel & Retry

---

# 24.1 Cancel

API：

```text
DELETE /api/v2/transfers/:id
```

行为：

```text
cancel context
close stream
remove temp partial if needed
status = cancelled
```

---

# 24.2 Retry

失败：

```text
retry
```

如果支持 Range：

```text
resume
```

否则：

```text
restart
```

---

# 25. Phase 24 — API V2

现在才开始建立：

```text
/api/v2
```

---

# 25.1 Routes

```text
GET    /api/v2/info

GET    /api/v2/devices

GET    /api/v2/transfers
POST   /api/v2/transfers/files
POST   /api/v2/transfers/text

GET    /api/v2/transfers/:id
DELETE /api/v2/transfers/:id

GET    /api/v2/history
DELETE /api/v2/history

GET    /api/v2/settings
PUT    /api/v2/settings

GET    /api/v2/events
```

---

# 25.2 旧 API

继续保留：

```text
/send/file
/send/text
/recv
/history
/events
...
```

至少跨一个正式版本。

---

# 26. Phase 25 — Multi-Item Transfer

版本建议：

```text
v2.0
```

---

# 26.1 数据模型

增加：

```go
type TransferFile struct {
    ID       string
    Name     string
    Size     int64
    MimeType string
    FilePath string
}
```

Transfer：

```go
Items []TransferFile
```

---

# 26.2 多文件上传

不再自动强制打 ZIP。

保存：

```text
Transfer
├── item1
├── item2
└── item3
```

---

# 26.3 Receive 页面

显示：

```text
12 个文件
428 MB
```

单文件列表。

---

# 26.4 单独下载

API：

```text
GET /api/v2/share/:token/items/:itemID
```

---

# 26.5 Download All

提供：

```text
全部下载
```

浏览器能力允许时：

逐个触发需谨慎。

更推荐：

```text
下载为 ZIP
```

作为全部下载选项。

---

# 26.6 ZIP

ZIP 变为：

```text
动态打包下载
```

或临时缓存 ZIP。

不再上传时强制创建 ZIP。

---

# 27. Phase 26 — Settings Backend Expansion

---

# 27.1 AppConfig

扩展：

```go
type AppConfig struct {
    DeviceName           string
    DefaultExpiry        int64
    DefaultDownloadLimit int
    Theme                string
    Language             string
}
```

---

# 27.2 可选后续

```text
ReceiveDirectory
AutoAccept
FileSizeLimit
```

只在真正可实现时增加。

---

# 27.3 Config Migration

旧：

```json
{
  "device_name": "..."
}
```

必须仍可读取。

新增字段使用默认值。

---

# 28. Phase 27 — Security Pass

---

# 28.1 Token

检查：

- 随机性
- 长度
- 不可预测
- URL safe

---

# 28.2 Filename

测试：

```text
../../file
..\..\file
/
\
NULL
Windows reserved name
very long UTF-8
```

---

# 28.3 Manual URL

限制：

```text
http
https
```

禁止：

```text
javascript
file
data
```

---

# 28.4 MIME

下载文件时：

至少避免：

```text
inline HTML execution
```

默认：

```text
attachment
```

---

# 28.5 PIN

测试：

```text
wrong PIN
missing PIN
correct PIN
```

---

# 28.6 One-Time

重点测试：

```text
preview does not consume
HEAD does not consume
partial range does not consume prematurely
interrupted does not consume
full download consumes
```

---

# 29. Phase 28 — Performance Pass

---

# 29.1 Frontend Bundle

检查：

```bash
npm run build
```

目标：

```text
JS gzip < 70 KB
CSS gzip < 20 KB
```

如果超标：

检查：

```text
dependency
icons
router
state library
```

优先删除依赖。

---

# 29.2 Idle CPU

启动后静置。

确保不存在：

```text
高频 setInterval
requestAnimationFrame loop
DOM polling
```

---

# 29.3 Device Polling

目标：

```text
SSE primary
30s fallback
```

---

# 29.4 History

如果数据量：

```text
500 records
```

页面仍应流畅。

---

# 30. Phase 29 — Final UX Pass

逐项检查 AI 味。

---

# 30.1 搜索 CSS

检查是否出现：

```text
gradient
backdrop-filter
blur(
border-radius: 999
box-shadow
```

每一个都人工确认是否真的必要。

---

# 30.2 Card Audit

任何：

```text
card
panel
container
```

都必须问：

> 这个 border/background 是否真的承担信息层级？

如果只是为了“看起来现代”：

删除。

---

# 30.3 Text Audit

删除：

```text
快速
轻松
无缝
强大
现代
智能
美观
```

等装饰性文案。

---

# 30.4 Icon Audit

禁止 Emoji。

统一 SVG。

---

# 30.5 Empty State Audit

禁止：

```text
巨大插图
巨大 Icon
空状态营销文案
```

---

# 31. 最终测试矩阵

必须完成。

---

## File

```text
0 B
1 KB
1 MB
100 MB
1 GB
>2 GB
>4 GB
```

---

## Filename

```text
English
中文
日本語
emoji
spaces
very long name
same name
```

---

## Multi File

```text
2
10
100
1000 small files
```

---

## Network

```text
normal Wi-Fi
slow Wi-Fi
disconnect
reconnect
device shutdown
browser refresh
```

---

## Browser

```text
Chrome
Edge
Firefox
Safari
iPhone Safari
Android Chrome
```

---

## Security

```text
PIN
TLS
one-time
expired
used token
invalid token
```

---

## UI

```text
light
dark
Chinese
English
320 width
375 width
390 width
430 width
1280
1440
1920
2560
```

---

# 32. Release Checklist

发布前：

```bash
go test ./...
```

必须通过。

```bash
cd web
npm run typecheck
npm run build
```

必须通过。

```bash
cd ..
go build .
```

必须通过。

---

# 33. README Update

README 更新内容：

```text
新截图
新首页
安装
使用方式
手机扫码
CLI
安全说明
```

删除过时 UI 描述。

---

# 34. Screenshots

至少准备：

```text
Desktop Light
Desktop Dark
Mobile
Send File
Transfer Progress
```

不要放十几张。

README 首屏一两张足够。

---

# 35. GitHub Release

Release Artifact：

```text
Windows amd64
macOS amd64
macOS arm64
Linux amd64
Linux arm64
```

继续保持。

---

# 36. 推荐 Commit 切分

不要让 AI 一个 Commit 改 50 个文件并同时改协议。

推荐：

```text
chore(web): bootstrap preact frontend

feat(web): add base design system

feat(web): add api client layer

feat(web): add application shell

feat(web): add file send queue

feat(web): add text and clipboard panels

feat(web): redesign nearby devices

feat(web): add qr connection dialog

feat(web): add transfer activity view

feat(web): add settings page

feat(web): add responsive mobile layout

refactor(server): serve vite build assets

remove(web): remove legacy desktop html

refactor(transfer): introduce transfer state model

feat(transfer): add expiration and cleanup

refactor(storage): stream file uploads to disk

feat(transfer): remove artificial 2gb limit

feat(transfer): add progress events

feat(transfer): add cancel and retry

feat(api): introduce v2 transfer api

feat(transfer): add multi-item transfers
```

---

# 37. Coding Agent 工作规则

每次开始一个 Task：

```text
1. 阅读相关现有文件
2. 说明准备修改哪些文件
3. 只完成当前 Task
4. 不顺手重构无关代码
5. 运行测试
6. 修复本 Task 引入的问题
7. 总结修改
```

---

# 38. Agent 禁止行为

禁止：

```text
因为“最佳实践”直接换框架
因为“现代化”引入 Tailwind
因为“体验更好”引入大型 UI 库
因为“更实时”换成 WebSocket
因为“P2P”换成 WebRTC
因为“代码整洁”大规模重写 Go
因为“方便”删除 CLI
因为“安全”默认破坏 LAN 使用体验
```

---

# 39. Agent 遇到不确定需求时

优先级：

```text
1. 保持现有行为
2. 保持兼容
3. 选择更简单方案
4. 不新增依赖
5. 不扩展产品范围
```

---

# 40. Phase 完成报告模板

每个 Phase 完成后输出：

```text
## Completed

- ...

## Files Changed

- ...

## Behavior Changed

- ...

## Compatibility

- ...

## Tests

- go test ./...
- npm run typecheck
- npm run build

## Remaining

- ...
```

---

# 41. 第一轮实际执行范围

不要一开始就把整个 IMPLEMENTATION.md 全扔给 Agent 让它自由发挥。

第一轮只执行：

```text
Phase 0
↓
Phase 1
↓
Phase 2
↓
Phase 3
↓
Phase 4
```

做到：

> 新前端框架 + API 层 + State + SSE + 首页 Shell

但暂时还没有完整传输 UI。

---

# 42. 第二轮

执行：

```text
Phase 5
↓
Phase 6
↓
Phase 7
↓
Phase 8
↓
Phase 9
↓
Phase 10
↓
Phase 11
```

完成：

```text
文件
文本
剪贴板
设备
QR
最近传输
```

---

# 43. 第三轮

执行：

```text
Phase 12
↓
Phase 13
↓
Phase 14
↓
Phase 15
↓
Phase 16
↓
Phase 17
```

完成：

```text
Activity
Settings
i18n
Mobile
Go Embed
旧 UI 删除
```

此时发布：

```text
LANDrop 1.3
```

---

# 44. 第四轮

执行：

```text
Phase 18
↓
Phase 19
↓
Phase 20
↓
Phase 21
↓
Phase 22
↓
Phase 23
```

完成 Transfer Core。

发布：

```text
LANDrop 1.4
```

---

# 45. 第五轮

执行：

```text
Phase 24
↓
Phase 25
↓
Phase 26
↓
Phase 27
↓
Phase 28
↓
Phase 29
```

完成：

```text
API V2
Multi-item
Settings backend
Security
Performance
UX polish
```

发布：

```text
LANDrop 2.0
```

---

# 46. 最重要的开发顺序

必须坚持：

```text
先重构产品界面
↓
再重构 Transfer Core
↓
最后升级协议与多文件模型
```

禁止反过来。

LANDrop 当前最大的短板不是传输协议。

而是：

```text
用户体验
信息架构
前端可维护性
```

---

# 47. LANDrop 1.3 Definition of Done

完成以下内容即可发布 1.3：

```text
Preact
TypeScript
Vite

新首页
新 Header
File Queue
Text
Clipboard
Nearby Devices
QR Dialog
Recent Transfers
Activity
Settings
Mobile

Go embed Vite dist

删除 desktop.html
```

同时：

```text
现有 Go Transfer Protocol 不大改
现有 CLI 不破坏
现有 API 保持
```

---

# 48. LANDrop 1.4 Definition of Done

```text
Transfer Status
TTL
Cleanup
File Streaming
取消 2GB 限制
Progress
Speed
ETA
Cancel
Retry
```

---

# 49. LANDrop 2.0 Definition of Done

```text
Multi-item Transfer
单文件下载
批量下载
ZIP 可选
API V2
完整 Settings
Transfer Lifecycle
Security Pass
Performance Pass
```

---

# 50. 最终产品验收

最终用户流程必须做到：

```text
启动 LANDrop
↓
浏览器打开
↓
拖入文件
↓
选择目标设备
↓
发送
↓
看到实时进度
↓
完成
```

用户不需要理解：

```text
mDNS
HTTP Server
Token
SSE
IP
Transfer Store
```

这些属于实现细节。

LANDrop 最终应该给人的感觉是：

> 一个真正安装在电脑里的系统工具，只不过它的 UI 恰好运行在浏览器里。

而不是：

> 一个 AI 写出来的局域网传输网页。

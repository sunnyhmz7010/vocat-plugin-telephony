<div align="center">
  <h1>电话助手 · Telephony Assistant</h1>
  <p>为 VoCat 提供通话面板、来电规则、语音留言和 SIP 网关。</p>
</div>

<p align="center">
  <a href="https://github.com/sunnyhmz7010/vocat-plugin-telephony/blob/main/go.mod"><img src="https://img.shields.io/github/go-mod/go-version/sunnyhmz7010/vocat-plugin-telephony?color=3b82f6" alt="Go version" /></a>
  <a href="https://github.com/sunnyhmz7010/vocat-plugin-telephony/blob/main/LICENSE"><img src="https://img.shields.io/github/license/sunnyhmz7010/vocat-plugin-telephony?color=10b981" alt="License" /></a>
  <a href="https://github.com/sunnyhmz7010/vocat-plugin-telephony/actions/workflows/github-code-scanning/codeql"><img src="https://github.com/sunnyhmz7010/vocat-plugin-telephony/actions/workflows/github-code-scanning/codeql/badge.svg?branch=main" alt="CodeQL" /></a>
</p>

---

## ✨ 为什么做这个插件

[VoCat](https://github.com/sunnyhmz7010/VoCat) 负责连接蜂窝模组并提供通话接口。电话助手把拨号、联系人、来电处理和音频回放放在一个面板中，并提供后台规则、语音信箱及软话机接入，方便在浏览器关闭后继续处理来电。

插件随 VoCat 启动，使用宿主的设备与 SIM；它不是独立电话服务，也不负责切换 eSIM 配置文件。

## 🚀 核心能力

- **通话面板**：选择设备拨号、接听、挂断，通过浏览器麦克风接入通话。
- **来电规则与联系人**：按顺序匹配号码、前缀、联系人、陌生或匿名来电，支持设备限定和规则试算。
- **通话记录与录音**：后台收集通话状态，按方向、未接或录音筛选，回放、下载及清理录音。
- **语音留言**：自动接听，播放本地 WAV 提示音，按静音或时长结束录制，设置容量与保留期。
- **事件通知**：独立配置 Webhook、Telegram、PushPlus，编辑通知模板并预览或发送测试消息。
- **保号任务**：按小时间隔拨号或发短信，可附加线路空闲时长条件。
- **SIP 网关**：软话机通过 UDP 和 SHA-256 Digest 注册，使用选定设备的 SIM 拨打及接听电话。

## ⚡ 快速开始

### 📋 前置要求

- 已部署支持插件系统的 VoCat，目标主机为 Linux `arm64` 或 `amd64`；通话自动化需要可用的 VoWiFi 通道。
- 在 VoCat 主机上开启开发者模式：执行 `sudo vocat develop on`，再按现有部署方式重启 VoCat，使插件入口生效。
- 构建机具备 Git、Go **1.25.0 或更新的兼容版本**、Bash、`jq`、`zip`、`sed`、`awk`，以及 `shasum` 或 `sha256sum`。建议在 Linux 构建。
- 浏览器语音需要 HTTPS 等安全上下文、麦克风授权，以及 Web Audio / AudioWorklet / WebSocket 支持；宿主嵌入页面也必须允许使用麦克风。

### 📦 安装与运行

当前从源码构建，版本由 [vocat-plugin.json](./vocat-plugin.json) 决定；`main` 中的修复可能晚于清单版本号。

```bash
git clone https://github.com/sunnyhmz7010/vocat-plugin-telephony.git
cd vocat-plugin-telephony
bash ./build.sh
```

脚本先运行测试，再交叉编译并生成两个包，同时打印 SHA-256。当前清单版本为 `0.1.4`：

| VoCat 主机架构 | 插件包 |
| --- | --- |
| `aarch64` / `arm64` | `dist/telephony-assistant-0.1.4-linux-arm64.zip` |
| `x86_64` / `amd64` | `dist/telephony-assistant-0.1.4-linux-amd64.zip` |

在 VoCat 的 **系统设置 → 插件 → 上传插件包** 中选择对应架构的 ZIP，可填写脚本输出的 SHA-256 进行校验。启用后，从侧栏短信入口下方打开「电话助手」。后端由 VoCat 启动，无需另外运行二进制。

**升级前备份插件数据目录。** 当前宿主要求先卸载再安装同名插件，卸载会删除该插件目录，包括 `data/` 中的配置、凭据、联系人、记录和音频。构建脚本也会清空并重建 `dist/`，不要在那里存放需保留的文件。

## 📖 使用说明

**先选择运行模式**

| 模式 | 凭据与执行位置 | 使用范围 |
| --- | --- | --- |
| 面板模式，默认 | 手动通话使用浏览器当前 VoCat 会话，不保存管理员凭据 | 拨号、接听、挂断、浏览器语音及配置管理；页面关闭后面板逻辑停止 |
| 服务端模式，主动开启 | 插件保存管理员凭据，并从本机登录 VoCat | 后台规则、通话记录收集、自动录音、留言、保号任务及 SIP 网关 |

自动规则建议使用服务端模式。当前面板规则存在已知实现差异：「放行」可能触发接听，「接听」规则未执行，延迟接听也可能被跳过。请勿把面板规则视为与服务端等价；规则试算只检查匹配结果，不验证实际通话动作。

开启服务端模式：进入「设置」，勾选「启用」，填写本机 VoCat 地址（默认 `https://127.0.0.1:7575`）、管理员账号密码和证书路径，再点「保存并连接」。确认状态显示会话有效后使用后台功能。客户端只接受 loopback 地址；优先配置证书，跳过证书校验需要在设置中明确启用。

**拨号、记录与录音**

在「拨号」选择设备并输入号码。通话接通后，点击「接入语音」并授权麦克风；接听本身不会自动打开麦克风。

「通话记录」由插件后台轮询形成，可筛选未接或有录音的通话。「处理记录」记录规则动作与结果。两者不是完整的运营商通话账单：插件停止、服务端未连接或通话完全落在两次轮询之间时，可能没有记录。

自动录音默认关闭，在「通话记录」中启用并选择录制方向、时长上限、总容量和保留期。录音内容取决于宿主媒体接口提供的音频，不保证完整保留双方音频或网络上的所有媒体包。

**来电规则与联系人**

先在「联系人」维护号码与姓名，再在「来电规则」按优先级排列规则，第一条命中即生效。服务端支持放行、拒接、接听和语音留言；接听及留言可以延迟执行。保存规则后用试算检查常用号码、陌生号码与匿名来电是否落入预期分支。

号码按后 **9 位有效数字**比较，短于 9 位的号码整串匹配。例如 `+447700900123` 与 `07700900123` 可命中同一联系人；这种简化规则也可能把后 9 位相同的不同号码视为同一个号码。

**语音留言**

启用服务端模式后，在「语音留言」勾选「启用」，设置录制上限、静音时长、容量与保留期并保存，再添加动作设为「语音留言」的来电规则。

提示音需手动放到插件数据目录，填写相对路径，例如 `greeting.wav`；格式为 **8000 Hz、单声道、16 位 PCM WAV**。界面不提供上传或文字转语音功能。路径留空会直接录音；文件缺失或播放失败时，当前实现会继续录制，因此配置后应实际拨入检查提示音。

**通知模板与保号任务**

「通知」中的通道独立于 VoCat 的通知配置。可为来电、拒接、接听、未接、留言和保号结果分别设置标题与正文，先预览再发送测试通知。

例如，将标题设为 `未接来电：{{display}}`，正文设为 `设备 {{device_name}}，号码 {{caller}}，时间 {{time}}`。可用占位符：

| 内容 | 占位符 |
| --- | --- |
| 来电与联系人 | `{{caller}}`、`{{number}}`、`{{contact}}`、`{{display}}`、`{{called}}` |
| 设备 | `{{device_id}}`、`{{device_name}}` |
| 处理结果 | `{{action}}`、`{{rule}}`、`{{duration}}`、`{{recording}}`、`{{result}}` |
| 时间与事件 | `{{time}}`、`{{timestamp}}`、`{{event}}` |

未知占位符会原样保留，便于发现拼写错误。

「保号任务」支持电话或短信，按小时间隔调度，可限制为仅在线路空闲一定时长后执行。空闲判断依据当前插件进程观察到的通话活动，重启后会重新积累，不代表完整线路历史。需要切换 eSIM 的任务应使用宿主相应功能。

**接入 SIP 软话机**

先确认服务端模式已成功连接，再打开「SIP 网关」，启用网关，设置独立的 SIP 用户名与随机密码、承载通话的设备、监听地址及来源白名单。保存后网关会重新应用配置。

| 客户端设置 | 配置要求 |
| --- | --- |
| 服务器 / Domain | VoCat 主机可达的 IP 地址 |
| 端口 | 网关监听端口，默认 `5060` |
| 传输 | UDP |
| 用户名 / 密码 | 插件中配置的 SIP 账号，不是 VoCat 管理员账号 |
| Digest 认证 | SHA-256，`qop=auth` |
| 音频编码 | G.711 PCMA 或 PCMU，8000 Hz |

客户端必须支持 [RFC 8760](https://www.rfc-editor.org/rfc/rfc8760.html) 的 SHA-256 Digest；仅支持 MD5 的旧客户端不能注册。注册列表出现终端后可尝试外呼；所选 SIM 来电时，已注册终端同时振铃，首个接听者建立通话。具体客户端版本需实际验证。

网关默认关闭，启用时默认监听 `0.0.0.0:5060`，会接收所有网卡上的请求。SIP 端口独立于 VoCat 的 HTTP 访问控制；建议只在受控内网使用，并配置 `allowed_sources` 为客户端 IP 或 CIDR，空白名单不限制来源。网关实施 Digest 认证和失败锁定，但未提供 SIP TLS / SRTP。请使用密码管理器生成的高熵随机密码：SHA-256 Digest 仍存在抓包后的离线弱口令猜测风险，在线失败锁定不能消除它。[RFC 7616 §5.1](https://www.rfc-editor.org/rfc/rfc7616.html#section-5.1)

## 🧠 功能细节

- **宿主与插件边界**：VoCat 反向代理会移除会话 Cookie、CSRF 与 Authorization 头，插件后端因此需要单独登录才能操作通话；手动通话由浏览器直接使用当前会话。
- **后台执行**：每秒轮询可用设备的通话，仅对具有稳定通话 ID 的 VoWiFi 状态执行自动规则；跳过未运行设备及 `wifi_410`。保号任务按分钟检查是否到期。
- **数据落盘**：状态保存在 `VOCAT_PLUGIN_DATA_DIR` 下的 JSON 文件。管理员密码、SIP 密码和通知密钥为明文存储，读取接口返回掩码；Linux 状态文件写入权限为 `0600`，这不等于加密。备份同样包含敏感数据。
- **会话恢复**：复用已登录会话，遇到 401 尝试重新登录一次；遇到 429 按 `Retry-After` 抑制后续登录，避免连续重试。
- **音频路径**：浏览器通过 AudioWorklet 转换 PCM，SIP 网关通过 RTP/G.711 与宿主音频 WebSocket 桥接。留言与录音按容量和保留期清理。
- **前端部署**：原生 HTML/CSS/JavaScript 随插件打包，不依赖 CDN 或 npm 构建。删除确认使用页面内按钮，适配宿主 iframe 的弹窗限制。
- **当前限制**：拨号键盘只编辑号码，通话中不发送 DTMF；SIP 不声明 `telephone-event`。客户端自行产生的带内音能否用于语音菜单，取决于客户端与线路。拒接通过宿主挂断接口执行，插件不提供自定义 SIP 拒接码。

## 🧱 技术栈

- 后端：Go 1.25，标准库 HTTP 服务与 JSON 文件存储。
- WebSocket：[`github.com/coder/websocket`](https://github.com/coder/websocket)。
- 前端：原生 HTML、CSS、JavaScript、Web Audio 和 AudioWorklet。
- 通信与音频：SIP/UDP、SHA-256 Digest、RTP/G.711、8000 Hz PCM/WAV。
- 构建：Bash、`jq`、`zip`；`CGO_ENABLED=0` 交叉编译。
- 目标平台：Linux `arm64` / `amd64`，由 VoCat 插件系统管理生命周期。

## 🗂️ 项目结构

```text
vocat-plugin-telephony/
├── .github/             # Issue 表单与 PR 模板
├── main.go              # HTTP 接口、配置与网关生命周期
├── vocat-plugin.json    # 插件清单、权限、入口与架构映射
├── build.sh             # 测试、交叉编译与 ZIP 打包
├── internal/
│   ├── contacts/        # 联系人及号码归一化
│   ├── engine/          # 轮询、规则执行、录音与保号调度
│   ├── media/           # 宿主音频 WebSocket
│   ├── notify/          # 模板与通知通道
│   ├── recorder/        # 通话音频录制
│   ├── rtp/             # RTP 会话与 G.711 编解码
│   ├── rules/           # 有序来电策略
│   ├── sip/             # SIP 消息与 Digest 认证
│   ├── sipgw/           # 注册、呼入呼出与音频桥接
│   ├── store/           # 配置、通话记录与留言索引
│   ├── vocat/           # 宿主 API 与会话管理
│   ├── voicemail/       # 留言录制接口
│   └── wav/             # WAV 读写
├── web/                 # 面板、浏览器语音及 AudioWorklet
├── CONTRIBUTING.md      # 开发与贡献指南
├── SECURITY.md          # 安全报告渠道
└── LICENSE              # GPL-3.0
```

## 👨‍💻 本地开发

### 🧰 环境

使用 Go 1.25.0 或更新的兼容版本；打包工具同「快速开始」。前端无需安装 npm 依赖。完整测试优先使用 Linux，SIP/RTP 测试需要本机 UDP 通信正常。

插件后端由宿主提供 `VOCAT_PLUGIN_LISTEN` 与 `VOCAT_PLUGIN_DATA_DIR`；直接运行而未配置这两个变量会退出。开发测试服务仅监听 `127.0.0.1` 或 `::1`。

### ⚙️ 命令

在仓库根目录执行：

```bash
go vet ./...
go test ./...
go build -o bin/telephony-dev .
bash ./build.sh
```

修改 Go 文件后运行 `gofmt`。Windows 下文件权限与 Unix 绝对路径测试有已知平台差异；请在验证记录中注明失败或跳过项，不将部分测试通过描述为全套通过。具体要求见 [CONTRIBUTING.md](./CONTRIBUTING.md)。

问题反馈和功能建议请使用 [Issue 表单](https://github.com/sunnyhmz7010/vocat-plugin-telephony/issues/new/choose)，附插件及 VoCat 版本、运行模式和脱敏后的复现信息。

## 🔐 安全报告

如果发现安全问题，请不要公开披露细节。请优先参考仓库中的 [SECURITY.md](./SECURITY.md) 提交安全报告。

## 📄 许可证

本项目基于 [GPL-3.0](./LICENSE) 开源。

<div align="center">
  <sub>Built with ❤️ by Sunny</sub>
</div>

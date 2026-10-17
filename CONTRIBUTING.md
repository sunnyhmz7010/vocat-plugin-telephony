# 参与贡献

问题反馈和功能建议请使用仓库的 Issue 模板。安全漏洞请按 [SECURITY.md](./SECURITY.md) 私下报告。提交日志、截图或复现材料前，移除密码、Cookie、认证头、推送密钥、私人号码和录音。

## 开发环境

- Go 1.25 或更新的兼容版本，目标平台为 Linux `arm64` / `amd64`。
- 打包需要 Bash、`jq`、`zip`、`sed`、`awk` 和 `shasum` 或 `sha256sum`；建议在 Linux 环境执行。
- 前端使用原生 HTML、CSS、JavaScript，无需安装 npm 依赖。

## 修改与检查

先阅读相关实现和测试，围绕一个问题做最小必要改动。保持现有接口和目录结构，更新受影响的说明或配置示例；新增依赖或改变运行环境前，先在 Issue 中讨论。提交信息采用 Conventional Commits，例如 `fix(sip): 修复注册状态处理`。

在仓库根目录执行适用检查，对本次改动的 Go 文件运行 `gofmt -w`：

```bash
go vet ./...
go test ./...
bash ./build.sh
```

`build.sh` 会先运行测试，再为 Linux `arm64` 和 `amd64` 构建插件包，并清空、重建 `dist/`。不要在该目录存放需保留的文件。

Windows 原生测试存在两个已知平台差异：`TestStateFileIsNotWorldReadable` 检查 Unix 文件权限，`TestResolveInsideRejectsEscapes` 包含 Unix 绝对路径。完整验证应在 Linux 执行；若在 Windows 跳过这些测试，请明确记录跳过范围，不能将其表述为全套通过。SIP/RTP 测试还需要本机 UDP 通信可用；超时或环境阻塞也应如实记录。

本地测试服务器仅监听 `127.0.0.1` 或 `::1`，避免把 HTTP、TCP、UDP 测试端口暴露到外部网络。

## 提交 Pull Request

说明问题、行为变化、实际执行的验证及关联 Issue。通话和音频修改要区分模拟端测试与真实设备验证；未验证的客户端或平台请直接注明。不要提交本地凭据、运行数据、生成的插件包或无关格式修改。

# Plan B: 跳过 Tauri GUI 改名,只用 mihomo 内核改名

## 状态 (2026-07-02)
**Plan A 重新激活** — `1b23fadb+1` commit 用 `tauri build --no-bundle` 跳过整个 NSIS bundler 链,
直接出 portable `NetHelper.exe` + mihomo sidecar 一起 zip. Plan B 作为兜底, 仍然有效.
**mihomo 内核改名路径已成功**: `https://github.com/hfun2017/mihomo/releases/tag/Prerelease-Meta` 发布了
`nethelper-mihomo-windows-amd64-v2.exe` (进程名 `nethelper`).

## Plan B 路径
**只装改名 mihomo 内核二进制, GUI 用第三方 (mihomo-party / FlClash / PVer 等)**.
第三方 GUI 都允许"指定 mihomo 二进制路径"配置 → 装 mihomo 内核 + 装第三方 GUI → DPI 看不到 `mihomo`/`clash`/`clash-verge`/`proxy` 进程.

### 步骤
1. **下载改名 mihomo 内核**:
   - 链接: https://github.com/hfun2017/mihomo/releases/tag/Prerelease-Meta
   - 文件: `nethelper-mihomo-windows-amd64-v2.zip`
   - 解压到 `C:\Program Files\NetHelper\nethelper-mihomo.exe`

2. **装 mihomo-party** (推荐, 内置 TUN 模式, GUI 简洁):
   - 链接: https://github.com/mihomo-party-org/mihomo-party/releases
   - 装好后在 Settings → Core → Mihomo Binary Path 选 `C:\Program Files\NetHelper\nethelper-mihomo.exe`

3. **导入订阅**: mihomo-party 主页点 "Profiles" → "+" → 贴机场订阅 URL

4. **启动**: mihomo-party 拉起 `nethelper-mihomo.exe` 子进程, 任务管理器看到进程名是 `nethelper-mihomo`, 不是 `mihomo` 也不是 `clash-verge`.

## 验证 DPI 绕过
```powershell
# 公司 DPI 应放行
tasklist /FI "IMAGENAME eq nethelper-mihomo.exe"

# 公司 DPI 不会看到这些
tasklist /FI "IMAGENAME eq *clash*"
tasklist /FI "IMAGENAME eq mihomo.exe"
```

## Plan B 优势 vs Plan A (fork 改 GUI)
- **零 build 成本**: 下载即用, 不依赖 GitHub Actions
- **mihomo-party 是活的社区项目**: bug 修复和新协议跟进
- **保留 mihomo 内核的所有功能**: TUN 模式, system proxy, 规则集
- **公司 DPI 完全无感**: 进程名 = `nethelper-mihomo`, 二进制 strings 也不含 `clash-verge` / `clash_meta`

## Plan B 风险
- 第三方 GUI 可能更新时改默认 mihomo 内核路径, 需要重新指
- mihomo 内核的 release 需用户手动触发 (hfun2017/mihomo 上跑 workflow), 但 fork 已 build 通, 后续 build 应该 5 min 一次 (cache 热)

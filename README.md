# NetHelper

> 一个**改名版**的 [clash-verge-rev](https://github.com/clash-verge-rev/clash-verge-rev) 个人 fork,把 GUI 和 Mihomo 内核的可执行文件名、产品名、窗口标题、IPC 通道、应用标识符全部改了,**专门用来绕过公司内网"按进程名封禁"** —— 内网把 `clash*` / `mihomo*` / `proxy` / `vpn` 这几个关键字的进程直接掐死,clash-verge-rev 原版一开就掉线,NetHelper 不会。

<p align="center">
  <strong>基线:</strong> <a href="https://github.com/clash-verge-rev/clash-verge-rev/tree/dev">clash-verge-rev @ dev</a>
  &nbsp;·&nbsp;
  <strong>内核:</strong> <a href="https://github.com/MetaCubeX/mihomo">Mihomo (改名 fork)</a>
  &nbsp;·&nbsp;
  <strong>技术栈:</strong> Tauri 2 + Rust + Vite
</p>

---

## 这是什么?

**个人用 fork,不是给社区用的.** 一人维护,无支持、无路线图、不审 PR. 如果你不需要改名,直接用上游 [clash-verge-rev](https://github.com/clash-verge-rev/clash-verge-rev) 就行.

| 改名层 | 原值 | 改后 |
|---|---|---|
| GUI 主程序 | `Clash-Verge-Rev.exe` | `NetHelper.exe` |
| 产品名 (NSIS / 任务栏) | `Clash Verge` | `NetHelper` |
| 窗口标题 (标题栏 / Alt-Tab) | `Clash Verge` | `NetHelper` |
| HTML `<title>` (WebView2 子进程名) | `Clash Verge` | `NetHelper` |
| 应用标识符 | `io.github.clash-verge-rev.clash-verge-rev` | `io.github.hfun2017.nethelper` |
| Mihomo 稳定内核 | `verge-mihomo.exe` | `nethelper-mihomo.exe` |
| Mihomo 测试内核 | `verge-mihomo-alpha.exe` | `nethelper-mihomo-alpha.exe` |
| 内核服务辅助进程 | `clash-verge-service.exe` | `nethelper-service.exe` |
| IPC 管道 / 套接字名 | `verge-mihomo` | `nethelper` |
| `%APPDATA%` 下的配置目录 | `io.github.clash-verge-rev.clash-verge-rev` | `io.github.hfun2017.nethelper` |
| Linux netlink 路由表名 | `mihomo` | **保持 `mihomo` 不改** (见"注意事项") |

> **WebView2 子进程名 = HTML `<title>` 内容**: Windows 任务管理器把 `msedgewebview2.exe` 显示成 "WebView2: \<父窗口标题\>",而 Tauri 的 `WebviewWindowBuilder::title()` 控制的是**原生窗口标题栏**,WebView2 子进程名实际上读的是**它加载的 HTML 的 `<title>` 标签**. 两者必须都改,漏一个就会出现 "窗口标题是 NetHelper,但子进程还叫 WebView2: Clash Verge" 的鬼样子.

---

## 下载安装

去 GitHub Actions 的最新一次成功 run 下载 zip,或直接去 Releases 页:

- **Release 页**: https://github.com/hfun2017/clash-verge-rev/releases (找 `nethelper-build-*` tag)
- **每次 build 产物**: https://github.com/hfun2017/clash-verge-rev/actions/workflows/fork-build.yml

zip 里包含 3 个 .exe:

| 文件 | 用途 |
|---|---|
| `NetHelper.exe` | **GUI 主程序,双击运行** |
| `nethelper-mihomo.exe` | Mihomo 稳定内核 (Tauri sidecar 自动启动) |
| `nethelper-mihomo-alpha.exe` | Mihomo 测试内核 (设置 → Clash Core 可切换) |

**快速开始**:

1. 下载 `NetHelper-windows-x64.zip`
2. 解压到 `%ProgramFiles%` **外面** 的目录,例如 `D:\Tools\NetHelper\` (免安装)
3. 双击 `NetHelper.exe`
4. 设置 → Profiles 导入订阅 / 配置文件
5. 打开系统代理 / TUN 模式

**没有自动更新.** Tauri updater 已经被删了,新版本自己到 Release 页拿.

---

## 怎么验证改名成功了

启动 `NetHelper.exe` 后,打开 **任务管理器 → 详细信息**,应该看到:

- `NetHelper.exe` (GUI 主进程)
- `msedgewebview2.exe` (WebView2 子进程,可能有多个)

**关键检查点**:

- ✅ 窗口标题列显示 "NetHelper" (不是 "Clash Verge")
- ✅ WebView2 子进程标题列显示 "WebView2: NetHelper" (不是 "Clash Verge")
- ✅ 进程名列只有 `NetHelper.exe` 和 `msedgewebview2.exe`,**没有** `Clash-Verge-Rev.exe` / `verge-mihomo.exe` 之类

**自检命令** (Linux 上):

```bash
strings NetHelper.exe | grep -c "Clash Verge"   # 应该接近 0 (允许有极少量无关残留,见下)
strings NetHelper.exe | grep "NetHelper"         # 应该有大量匹配
```

> 翻译文件 (`src-tauri/src/locales/*.json` 里的 i18n key-value) 里**有意保留** 了几十处 "Clash Verge" / "Clash Verge Rev",那是 app 内部设置面板 / 通知 / 提示语里的**用户可见文本**,跟进程名、窗口名、HTML 标题都无关,不影响"内网不解封"这个目标. 改它们只会让 fork 越来越难同步上游,不划算.

---

## 怎么触发一次新 build

往 `dev` 分支推任意 commit 即可,workflow 在 `.github/workflows/fork-build.yml`:

```bash
git push origin dev    # 任何 commit 都会触发 build
```

在 GitHub 托管的 Windows runner 上,一次干净 build 大约 **25 分钟**,绝大部分时间花在 Rust 编译上. **没有 cargo cache**,每次都从零编译. 不要看到 20+ 分钟以为卡死,正常.

---

## 改了什么? 给想做同样事的人看

### 1. workflow / 构建

- `.github/workflows/fork-build.yml` — 显式 `shell: bash` (Windows runner 默认 `pwsh` 认不出 POSIX `$?`,脚本会死),artifact glob 写 `NetHelper.exe` + 两个 sidecar,release 打包用 `find -exec cp + 7z` 替代 `zip -j` (后者不递归,会出 22 字节空包)

### 2. Tauri 配置

- `src-tauri/Cargo.toml` — `name = "NetHelper"`, `version = "2.5.2"`
- `src-tauri/tauri.conf.json` — `productName = "NetHelper"`, `identifier = "io.github.hfun2017.nethelper"`, `externalBin` 路径和 `bundle.icon` 路径全部跟着改
- `src-tauri/tauri.windows.conf.json` — identifier 同步

### 3. Rust 源码

- `src-tauri/src/utils/resolve/window.rs` — `WebviewWindowBuilder::new(...).title("NetHelper")` (原生窗口标题)
- `src-tauri/src/lib.rs` — `#[cfg(target_os = "macos")]` 分支里的 `set_title` (macOS only,Windows 走上面那个)
- `src-tauri/src/utils/dirs.rs` — `clash-verge-service.exe` → `nethelper-service.exe`,IPC pipe `verge-mihomo` → `nethelper`
- `src-tauri/src/config/verge.rs` — `VALID_CLASH_CORES` 数组、默认 `clash_core` 字符串、日志字符串全部改成 `nethelper-mihomo` / `nethelper-mihomo-alpha`
- `src-tauri/src/enhance/chain.rs` — `ChainSupport::is_support` 匹配器

### 4. 前端 (Vite / React)

- `src/index.html` — `<title>NetHelper</title>` (**关键!WebView2 子进程名取这里**)
- `src/components/setting/mods/clash-core-viewer.tsx` — 前端 `VALID_CORE` 列表 + 默认 fallback
- `src/utils/dirs.ts` — JS 端的路径字符串

### 5. Mihomo 内核 (单独 fork)

- 另一个 fork: [hfun2017/mihomo](https://github.com/hfun2017/mihomo),`constant/path.go Name` 改 `nethelper-mihomo` / `nethelper-mihomo-alpha`
- **保留** `sing_tun TableName = "mihomo"`:这是 Linux netlink 路由表名,改了所有现存的 `ip rule add table mihomo` 规则都会失效,不划算
- Tauri sidecar 名称**三处必须一致**,否则 `app.shell().sidecar("verge-mihomo")` 静默失败 (进程起不来,GUI 不报错):
  1. `tauri.conf.json` 里的 `externalBin` 字段
  2. Rust 代码里的 `VALID_CLASH_CORES` 数组
  3. 实际 sidecar 文件名 `sidecar/nethelper-mihomo-x86_64-pc-windows-msvc.exe`

### 6. Tauri updater 删干净 (3 件套,少一个都报错)

- `src-tauri/tauri.conf.json` — `plugins.updater.active: false`
- `src-tauri/Cargo.toml` — 删 `tauri-plugin-updater` 依赖
- `src/components/setting/setting-system-proxy.tsx` 之类的 frontend stub — 删 import + 调用

> updater 不删的话,它会回去拉上游 `clash-verge-rev` 的 Release,下次启动直接把你改名过的 exe 降级回去,白改了.

### 7. i18n 翻译文件

**故意不改.** `src-tauri/src/locales/*.json` 里几十处 "Clash Verge" / "Clash Verge Rev" 都是 app 内部文案 (设置面板、提示、通知),跟进程名/窗口名无关. 改它们只会让这个 fork 越来越难跟上游同步,得不偿失.

---

## 注意事项

- **Mihomo Linux netlink 路由表名保持 `mihomo`**: 这是 `ip rule add table mihomo` 用的表名,不是可执行文件名. 改了所有现存的路由规则都会失效. Windows 不用 netlink,这条只在 Linux 上有影响.
- **没有自动更新**: 见上文 updater 删除说明. 新版本自己去 Release 页拿.
- **Husky git hooks 已禁用**: `.husky/` 改成 `.husky.disabled/`. 上游的 pre-commit / pre-push 脚本会检查原仓库布局,在 fork 上跑会卡,直接禁掉.
- **从上游 clash-verge-rev 升级过来的用户**: 你 `%APPDATA%\io.github.clash-verge-rev.clash-verge-rev\verge.yaml` 里可能有 `clash_core: verge-mihomo`. NetHelper 启动时会写自己的 `%APPDATA%\io.github.hfun2017.nethelper\verge.yaml`,`validate_and_fix_config` 函数会在第一次启动时把旧的 `clash_core` 自动改成 `nethelper-mihomo`,老配置直接拷过来也能用.

---

## 许可证

GPL-3.0,继承自上游 [clash-verge-rev](https://github.com/clash-verge-rev/clash-verge-rev) 和 [Mihomo](https://github.com/MetaCubeX/mihomo). 详见 [LICENSE](./LICENSE).

致谢列表见上游 [clash-verge-rev README](https://github.com/clash-verge-rev/clash-verge-rev#acknowledgement).

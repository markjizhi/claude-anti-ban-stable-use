---
name: claude-cleanup
description: 在 macOS 上审计、备份、清理或重置 Claude Code 与 Claude Desktop 本机状态。用户要清理本地缓存、排查登录态或明确要求选择性重置时使用。包含可选的本地 ID 轮换、钥匙串清理、桌面端数据移除、精简模式和台北时区设置；不用于规避封禁或平台风控。
metadata:
  compatibility: macOS；Python 3.10+；写操作需要真实终端；仅支持默认 Claude 配置目录。
---

# Claude 本机清理

使用随附脚本执行，不要临时拼接 `rm`。脚本采用窄白名单、完整备份、一次最终知情确认和失败即停止。

## 环境与范围

- macOS 和 Python 3.10+；保持 `SKILL.md` 与 `scripts/claude_cleanup.py` 同在本目录。下方命令均从 Skill 根目录执行；在其他目录调用时使用脚本的实际完整路径。
- 先核实用户要清缓存还是重置具体状态；可选项不能因为“清理”二字全部打开。
- 本版只支持默认 `~/.claude` 和 `~/.claude.json`。检测到其他 `CLAUDE_CONFIG_DIR`、关键写路径或其父目录为符号链接时停止写操作，报告需要人工检查，不要为了运行脚本擅自移动目录或取消用户配置。
- 本地缓存、ID 和登录态不能代表或清除平台服务器上的账号记录。不承诺账号解封或避免封号。

## 运行顺序

1. 如果当前执行者就是 Claude Code，只运行只读审计并提醒用户切换到 Codex 或普通终端。不要让正在使用 `~/.claude` 的 Claude Code 自己修改该目录。

   ```bash
   python3 scripts/claude_cleanup.py --audit
   ```

2. 如果由 Codex 或普通终端执行，先运行审计并向用户说明：
   - 第一个写操作是完整复制并校验 `~/.claude`，同时备份存在的 `~/.claude.json`；对照内容哈希、权限、目录和链接目标，不仅比较体积；
   - 可再生缓存和日志会自动纳入最终清单，不逐项询问；
   - 身份、钥匙串、桌面端登录态、应用、精简模式和时区属于风险操作，仍由用户选择；
   - 所有移除只进入时间戳废纸篓目录，不会永久删除或清空废纸篓；
   - 保护路径和遥测设置保持不变。

3. 用户明确要求继续后，在真实终端运行：

   ```bash
   python3 scripts/claude_cleanup.py
   ```

   不要用管道代答，不要增加跳过确认参数。若只有自动安全清理且 Claude 正在运行，脚本直接跳过安全目标，不要求退出；若用户选择风险操作，再交由用户正常退出并复检。脚本不能自动结束进程。

4. 脚本显示展开后的完整清单。用户输入一次 `CONFIRM` 后才开始写盘。若需要修改时区，开机密码只能由用户在 macOS 原生 `sudo` 隐藏提示中输入；不要让用户把密码发给执行者，也不要读取或保存。

5. 结束后报告备份路径、废纸篓批次、实际选择、遥测继承结果和时区。任何中途停止都报告为部分完成，不能笼统声称清理成功。

## 自动安全清理

在 Claude Code 和 Claude Desktop 都已退出后，自动把下列现存目标纳入最终清单，无需逐项提问：

```text
~/.claude/cache
~/.claude/stats-cache.json
~/.claude/telemetry
~/.claude/usage-data
~/.claude/usage.jsonl
~/.claude/usage.with-fix.jsonl
~/Library/Caches/claude-cli-nodejs
~/Library/Caches/com.anthropic.claudefordesktop*
~/Library/Logs/Claude
~/Library/Logs/DiagnosticReports/Claude*
~/Library/Application Support/CrashReporter/Claude*
```

这些目标必须仍通过脚本白名单检查，并且只移入废纸篓。

## 绝对保护范围

绝不删除或移动：

```text
~/.claude/projects、sessions、history.jsonl、file-history、debug
~/.claude/skills、plugins、hooks、commands、scripts、agents、mcp-servers
~/.claude/backups、CLAUDE.md、settings.json、settings.local.json
任何项目目录、Git 仓库、Codex 会话，以及未明确列入白名单的路径
```

`settings.json` 只允许结构化修改被用户明确选择的键。保留其他 env、hooks、plugins、权限、模型、MCP、状态栏和秘密。目标与保护路径重叠时立即停止。

## 风险操作

- **本地身份轮换**：轮换 `~/.claude.json` 的 `userID`、`machineID`，清除已知账号缓存字段，并把同一组新值同步到 `~/.claude/backups/.claude.json.backup.*`。仅用于隐私和排障，不宣称能改变平台关联判断。
- **钥匙串**：删除 `Claude Code-credentials` 会退出 CLI 登录，单独选择。
- **Claude Desktop**：可选择仅清持久数据与登录态，或连 `/Applications/Claude.app` 一并移入废纸篓。不能自动结束应用进程。
- **最小提示词模式**：在 `settings.json` 的 `env` 中设置或移除 `CLAUDE_CODE_SIMPLE=1`。按当前官方说明，它缩减系统提示词和工具、关闭多类自动发现，也不读取 OAuth token 与钥匙串认证；显式传入的 MCP 或部分目录资源另有规则。它可能影响订阅登录方式，只清缓存时应保持原状。先核对用户安装版本和 [官方环境变量说明](https://code.claude.com/docs/en/env-vars)，不要将精简模式当成清理的必要步骤。
- **台北时区**：仅在 macOS 当前不是 `Asia/Taipei` 时询问。保留原版的可选功能；它与美国住宅网络教程的时区目标不同。先记录原时区和恢复方法，用户确实需要时才使用 `sudo systemsetup` 修改并验证。

## 遥测继承

逐值保存以下键：

```text
DISABLE_TELEMETRY
DISABLE_ERROR_REPORTING
CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC
```

当前关闭就保持关闭；当前未关闭就不主动关闭。改变遥测状态不属于本 skill 的清理范围。

本 skill 不用于规避封禁、支付检查、设备或 IP 信誉、浏览器指纹或任何平台风控。

## 恢复与交付

备份在 `~/ClaudeBackups/claude-cleanup-时间戳/`，移除项在 `~/.Trash/claude-cleanup-时间戳/`。两处都可能包含个人资料，不能上传。恢复前退出 Claude 并另存当前状态，再依据 `manifest.json` 和废纸篓批次恢复对应文件，不能整目录覆盖正在运行的配置。钥匙串凭据需要重新登录；精简模式和时区要单独恢复，文件备份不等于系统设置回滚。

如果发生错误，列出已经发生的修改、备份路径和未完成项；不得重复运行来掩盖部分失败。

# Claude 本机清理 Skill

这是一个 **macOS 本地维护工具**。它检查并按你的选择清理 Claude Code / Claude Desktop 的缓存、登录态和部分配置。原分享包叫“清理 Claude 标记”，这里把实际能处理的范围写清楚。

**它不能清理平台服务器上的记录，也不保证解封或避免封号。**

## 先准备

- macOS；Python 3.10 或更新版本。
- 下载 [claude-cleanup 完整目录](../../skills/claude-cleanup/SKILL.md)，其中的 `SKILL.md` 和 `scripts/claude_cleanup.py` 必须放在一起。
- 若使用自定义 `CLAUDE_CONFIG_DIR`，或 `.claude`、设置、备份和废纸篓路径中存在符号链接，先交给 Agent 检查。本版写操作只支持默认目录，遇到这类布局会停止。

不需要先全局安装 Skill：本机 Codex 直接读取该目录即可。想安装进自己的 Agent 时，让它根据自己的 Skill 安装方式复制整个目录；不要只复制 `SKILL.md`。

## 复制给 Codex

```text
请完整读取 skills/claude-cleanup/SKILL.md 和 skills/claude-cleanup/scripts/claude_cleanup.py。
先检查我的系统和 Python 版本，只执行只读审计，告诉我发现了什么。
我选择需要的操作后，再按 Skill 的备份和人工确认步骤执行。
不要替我输入最终 CONFIRM，不要顺带修改其他设置。
```

如果直接打开 Skill 文件夹，读取 `SKILL.md` 和 `scripts/claude_cleanup.py` 即可。

## 只读审计

在仓库根目录运行：

```bash
python3 skills/claude-cleanup/scripts/claude_cleanup.py --audit
```

审计只读取本地状态，不清理、不改设置。

用户明确选择继续后，在真实终端运行：

```bash
python3 skills/claude-cleanup/scripts/claude_cleanup.py
```

脚本展示清单；本人输入 `CONFIRM` 才开始写盘。正在运行的 Claude Code 不应清理自己使用的目录，改用 Codex 或普通终端操作。

## 默认清理和可选操作

| 类型 | 行为 |
|---|---|
| 可再生缓存与日志 | Claude 应用已退出时纳入清单，确认后移到废纸篓 |
| 本地 ID 与账号缓存 | 单独选择；不等于改变平台的账号判断 |
| 钥匙串凭据 | 单独选择；会退出 CLI 登录 |
| Claude Desktop 持久数据或应用 | 单独选择；移到废纸篓，可导致退出登录或卸载 |
| `CLAUDE_CODE_SIMPLE` | 单独选择；会改变 Claude Code 的运行和认证方式，不是清理缓存的必要步骤 |
| 台北时区 | 保留原分享包的可选功能；与美国网络教程的时区目标不同，不能顺带执行 |

`CLAUDE_CODE_SIMPLE` 的当前官方说明包括：缩减工具与自动加载范围，也会影响 OAuth / 钥匙串认证；只想清缓存时保持原状。使用前核对 [Claude Code 官方环境变量说明](https://code.claude.com/docs/en/env-vars)。

会话、项目、Skill、插件、hooks、历史和其他保护目录不在清理范围。完整边界见 [SKILL.md](../../skills/claude-cleanup/SKILL.md)。

## 备份和恢复

- 首个写操作把 `~/.claude` 完整复制到 `~/ClaudeBackups/claude-cleanup-时间戳/`，并备份存在的 `~/.claude.json`；校验文件内容哈希、权限与符号链接。
- 被移除的文件进入 `~/.Trash/claude-cleanup-时间戳/`；不要在确认恢复前清空废纸篓。
- 恢复前正常退出 Claude 应用，把当前状态另行备份，再让 Codex 根据备份清单和废纸篓批次恢复对应文件。不要把整份旧目录直接覆盖到正在使用的目录。
- 钥匙串删除后需要重新登录；时区和精简模式需要按记录单独恢复，目录备份不会自动还原它们。
- 备份可能包含账号和项目资料，只保存在自己的私有位置，不要上传。

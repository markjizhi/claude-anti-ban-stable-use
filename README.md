# Claude 防封与稳定使用实战

我是极致先生。这里放的是我在使用 Claude 过程中整理的实战文档和 Skill，和我的视频配套，持续更新。

**找到对应教程，把链接或文件交给你自己的 Codex，按你的实际环境执行。**

这里分享个人经验，供你参考，不保证照着做就一定不封号。网络配置和本地清理也不能代替平台的账号申诉。

## 先找到你需要的资料

| 你要做什么 | 点击打开 | 适用范围 |
|---|---|---|
| 搭建自己的美国 VPS＋静态住宅 IP 网络 | [开始搭建 →](docs/us-residential-network/README.md) | macOS / Windows，需自己的服务器和住宅代理 |
| 安装和配置 AdsPower 指纹浏览器 | [打开配置教程 →](docs/adspower/README.md) | macOS / Windows，需已可用的 VPS 与住宅出口 |
| 检查或清理 Claude 本机缓存和状态 | [查看 Skill 用法 →](docs/claude-cleanup/README.md) | macOS，Python 3.10+ |

第三项的完整 Skill 在 [skills/claude-cleanup](skills/claude-cleanup/SKILL.md)，包含 `SKILL.md` 和配套脚本，使用时请保留整个文件夹。

## 怎么交给 Codex

1. 打开上面的对应入口，把网页链接发给 Codex；也可以下载整个仓库，让本机 Codex 打开相应目录。
2. 复制这句话：

   ```text
   请先完整读取这份教程及其配套文件，再按我的实际环境帮我执行。
   先确认系统、已有配置和缺少的条件，已经完成的部分直接复用。
   付款、登录、验证码和系统密码由我本人完成。执行前说明会改什么，完成后按文档验证结果。
   ```

3. 美国网络教程另有一段 [完整启动话术](docs/us-residential-network/prompt.md)。清理 Skill 请先做只读审计，再决定是否清理。

Codex 需要有读取文件和操作本机终端的能力。只有聊天权限的 Agent 可以帮你读教程和规划步骤，不能直接替你配置电脑。其他 Agent 能否安装或执行 Skill，取决于其运行环境。

## 想看配套的实战视频？

我把稳定使用 Claude 的视频、教程和资料，整理成了 **《稳定使用claude实战分享》** 专栏，目前免费开放，持续更新。

目前专栏中的 14 篇内容：

1. 你好，我是极致先生
2. 保姆级教程：稳定使用claude，从环境搭建+注册+支付
3. 为什么要死磕claude，看清一个人解决复杂问题的优秀能力
4. 深度实战，用大白话讲透，claude,codex,grokbot,workbuddy，到底怎么选？
5. Claude的封号机制，实战拆解，封的不是国籍
6. claude解封秘籍：2次亲历拿回数据+解封成功+费用秒退
7. 搭建美国vps和美国静态ip的实战步骤
8. 如果你的claude连续被封，用这个skill清理claude 电脑缓存信息
9. 申请美国手机号的实战步骤！
10. 2步野路子搞定AI 境外支付
11. 申请隐私邮箱，绕过claude 验证码邮件的跟踪器
12. 安装指纹浏览器教程
13. 实战分享，稳定使用claude的3个骚操作
14. 实战分享我稳定使用claude的3个野路子

**扫码添加我的微信，备注「Claude」，我把免费专栏入口发给你。**

下面是专栏的内容截图，点击图片可以查看原图：

<a href="assets/free-column.jpg"><img src="assets/free-column.jpg" alt="稳定使用claude实战分享免费专栏的完整内容截图" width="320"></a>

**扫码添加极致先生微信，领取免费专栏：**

<a href="assets/wechat-qr.jpg"><img src="assets/wechat-qr.jpg" alt="极致先生的微信二维码，添加时请备注 Claude" width="320"></a>

## 使用前看一眼

- VPS、住宅代理和 AdsPower 的产品、价格与界面可能变化，执行时以对应官方信息为准；已有资源先复用。
- 密码、API Key、真实代理配置、日志和部署进度放在仓库外的私有目录，不要提交到公开仓库。
- 本机清理 Skill 包含备份和人工确认。它处理本地状态，不提供账号解封或不封号保证。

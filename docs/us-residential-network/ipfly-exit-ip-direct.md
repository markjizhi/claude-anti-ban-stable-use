# IPFLY 静态住宅代理出口 IP 直连教程

> 本方案和原始教程由野路子 AI 内部分享群群友 **Iron** 提供。感谢 Iron 分享这次实测和排查方法。

适用于已经搭好「Clash Verge、Shadowrocket 等客户端 → VPS 上的 Xray → IPFLY 静态住宅代理」的用户。如果客户端到 VPS 的 TCP 测试正常，经代理访问网站却频繁超时，可以从 VPS 对比 IPFLY 的域名网关和出口 IP 接入。后者通过同一个固定住宅 IP 提供 SOCKS5 接入，可绕开原域名网关这条路径。

本教程用 `203.0.113.24` 表示你购买的 IPFLY 固定出口。它是保留的文档示例地址，请在实操时换成控制台显示的地址。`5001` 是端口示例，实际使用控制台为该连接方式提供的端口。客户端照常连接原 VPS 节点。[示例 IP 地址说明](https://www.rfc-editor.org/rfc/rfc5737.html)

## 1. 在 IPFLY 后台开启出口 IP 接入

1. 登录 IPFLY，进入「住宅静态 IP」→「IP 管理」，找到自己的固定住宅 IP。
2. 打开该条记录的「设置 IP 端口连接」，在「连接方式」中勾选「出口 IP」，点击「确认」。如果界面同时显示「代理主机」，保留它，便于回滚。
3. 核对该条记录的出口 IP、端口、代理用户名和密码。后续输入的是该条记录的**代理凭据**，与 IPFLY 网站登录账号分开。

这个开关会开放出口 IP 接入。已有的 Xray 在完成第 3 步后才会走新入口。

[IPFLY 官方出口 IP 接入说明](https://www.ipfly.net/blog/zh-cn/ipfly-june-updates-global-ip/)

## 2. 在 VPS 上验证新入口

SSH 登录 VPS。将下面的示例 IP、端口和用户名替换为自己的值，只填用户名。`curl` 会在终端提示输入代理密码。

```bash
curl -q -4 --fail --silent --show-error --noproxy '' \
  --connect-timeout 5 --max-time 12 \
  --proxy 'socks5h://203.0.113.24:5001' \
  --proxy-user 'YOUR_PROXY_USERNAME' \
  --write-out '\n' https://api.ipify.org
```

重复几次，每次都应返回你的固定住宅 IP。还可以用同样的命令把代理地址改为控制台里的原域名网关和端口，对比两条链路。若新入口超时、认证失败或返回了其他出口，先核对后台的开关、端口和凭据，确认通过后再切换 Xray。

## 3. 切换 Xray 的 SOCKS5 出站

### 找到生效配置并备份

以下以 Linux `systemd` 管理的 Xray 为例。使用你的实际服务名，查看 `ExecStart` 中的 Xray 程序路径和配置路径。如果配置由面板生成，请备份面板配置，并在面板中编辑对应的 SOCKS5 出站，避免后续保存覆盖修改。

```bash
systemctl list-unit-files '*xray*'
systemctl show setup-network-xray.service -p ExecStart --value
```

按上一步的结果填写下面三个变量。如果配置路径是符号链接，填它指向的实际文件。在同一个 SSH 会话中完成后续操作，并记下打印的备份路径。备份含代理凭据，保存在 VPS 上。

```bash
XRAY_SERVICE='setup-network-xray.service'
XRAY_BIN='/usr/local/bin/xray'
XRAY_CONFIG='/etc/setup-network/xray.json'
XRAY_BACKUP="${XRAY_CONFIG}.before-ipfly-direct.$(date -u +%Y%m%dT%H%M%SZ)"

sudo cp -a -- "$XRAY_CONFIG" "$XRAY_BACKUP" &&
  printf '备份路径：%s\n' "$XRAY_BACKUP"
```

### 修改正在使用的住宅出站

备份成功后执行 `sudoedit "$XRAY_CONFIG"`。在 `outbounds` 里找到转发到 IPFLY 的 `socks` 出站，把 `settings.address` 从原代理主机改成自己的出口 IP。如果控制台为出口 IP 接入提供了不同的端口或凭据，按已通过上一步测试的值填写。

下面只展示单个出站对象。原有 `tag`、入站、路由规则及防火墙继续沿用。如果原配置使用 `settings.servers` 数组，就在对应的服务器条目中改 `address`，沿用现有结构。[Xray SOCKS 出站配置](https://xtls.github.io/config/outbounds/socks.html)

```json
{
  "tag": "residential-us",
  "protocol": "socks",
  "settings": {
    "address": "203.0.113.24",
    "port": 5001,
    "user": "YOUR_PROXY_USERNAME",
    "pass": "YOUR_PROXY_PASSWORD"
  }
}
```

### 校验配置并重启服务

配置校验通过后才会重启 Xray，现有代理连接可能短暂中断。如果 `ExecStart` 使用 `-confdir` 或加载多个文件，校验时沿用服务的配置加载参数。任何一步失败，都按下文恢复备份。

```bash
sudo "$XRAY_BIN" run -test -config "$XRAY_CONFIG" &&
  sudo systemctl restart "$XRAY_SERVICE" &&
  systemctl is-active "$XRAY_SERVICE"
```

## 4. 从客户端验收

1. 用原有 Clash Verge 或 Shadowrocket 节点连续做几次 **HTTP 或 URL 连通性测试**，确认能成功访问测试网址。TCP 测试通常只覆盖客户端到 VPS 的建连。
2. 在走同一节点的浏览器中打开 `https://api.ipify.org`，确认显示的是你购买的固定住宅 IP。如果用规则模式，先在连接记录里确认该请求命中了住宅代理节点。
3. 如果电脑有本地 SOCKS 或混合代理端口，也可以明确指定本地代理来复测。下面假设监听在 `127.0.0.1:7890`，请按客户端设置调整端口。

```bash
curl -q --fail --silent --show-error --noproxy '' \
  --proxy 'socks5h://127.0.0.1:7890' --max-time 12 \
  --write-out '\n' https://api.ipify.org

curl -q --fail --silent --show-error --noproxy '' \
  --proxy 'socks5h://127.0.0.1:7890' --max-time 12 \
  --output /dev/null --write-out 'HTTP %{http_code}，耗时 %{time_total} 秒\n' \
  https://cp.cloudflare.com/generate_204
```

验收时应看到固定住宅 IP 和 `HTTP 204`。原案例切换后，Mac 的 15 次 HTTPS 请求全部成功，出口一致。这是切换后的短时结果，实际延迟取决于 VPS 到该住宅入口的线路。

## 5. 回滚

如果 Xray 启动失败、节点持续超时或出口不符，用切换前的备份覆盖当前配置，再校验、重启和复测。若已重新登录 SSH，先按第 3 步重新设置程序、服务和配置变量，`XRAY_BACKUP` 填之前记下的**已有备份路径**。

```bash
sudo cp -a -- "$XRAY_BACKUP" "$XRAY_CONFIG" &&
  sudo "$XRAY_BIN" run -test -config "$XRAY_CONFIG" &&
  sudo systemctl restart "$XRAY_SERVICE" &&
  systemctl is-active "$XRAY_SERVICE"
```

回滚后，重新执行第 4 步，确认原节点可用且出口仍是预期的固定住宅 IP。

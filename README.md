# proxy-rules

跨设备统一的分流规则。**规则内容只存在这一处**，各客户端只放引用。

## 文件

| 文件 | 内容 | 建议策略 |
| --- | --- | --- |
| `my-direct.list` | 局域网、极空间 NAS 云服务、特斯拉 API（TeslaMate）、Cloudflare Tunnel、Steam 下载 | `DIRECT` |
| `my-japan.list` | pixiv / DLsite / DMM / ニコニコ动画 等需要日本出口的站点 | 日本节点组（`🗾Japan`） |
| `my-proxy.list` | Steam 商店 / 社区 / 静态资源；OneDrive 网页入口（DNS 污染）；微软账户页 account.microsoft.com | 主代理组（`Proxies`） |

格式为 Clash `classical` rule-provider：**每行只有匹配条件，不带策略**，同一份文件 Shadowrocket（Surge 语法）也能直接用。

`my-direct.list` 里的条目决定了 NAS 开启 TUN 之后哪些流量不被劫持，删改前请确认：
极空间远程访问、TeslaMate 抓车辆数据、cloudflared 隧道都依赖它。

## Raw URL

```
https://raw.githubusercontent.com/notornado/proxy-rules/main/my-direct.list
https://raw.githubusercontent.com/notornado/proxy-rules/main/my-japan.list
https://raw.githubusercontent.com/notornado/proxy-rules/main/my-proxy.list
```

## 各客户端怎么引用

### Clash Verge Rev（Mac / PC）

见 [`verge/README.md`](verge/)。这个版本的 Verge 把增强配置拆成 Merge / Rules / Groups 三种类型，
语法各不相同：`rule-providers` 放 Merge，三条 `RULE-SET` 放 Rules，`🗾Japan` 组放 Groups。
**在 Merge 里写 `prepend-rules` 不生效**，坑的细节记在那个 README 里。

### mihomo（NAS 旁路由）

`config.yaml` 里写同样的三个 `rule-providers`（每个加 `proxy: Proxies` 让下载走代理），
`rules` 顺序：局域网直连 → `RULE-SET,my-direct,DIRECT` → `RULE-SET,my-japan,🗾Japan` →
`RULE-SET,my-proxy,Proxies` → `GEOSITE,cn,DIRECT` → `GEOIP,cn,DIRECT` → `MATCH,Proxies`。

首次启动前把三个 `.list` 预先放到配置目录的 `ruleset/` 下，避免拉不到 GitHub 时启动失败。

### Stash（iPhone）

用 Safari 打开下面的链接安装覆写（规则集 + 公共 DNS 都在里面），然后在覆写列表里启用：

```
https://link.stash.ws/install-override/raw.githubusercontent.com/notornado/proxy-rules/main/stash/override.stoverride
```

改覆写只改 [`stash/override.stoverride`](stash/override.stoverride)，手机上在覆写列表里点更新即可。

### Shadowrocket（iPhone / iPad）

配置 → 规则 → 添加规则集，各加三条，拖到规则列表最顶部，策略分别为
`DIRECT` / 日本节点组 / 主代理组，然后手动更新一次规则集。

## DNS：节点入口域名必须按订阅的方式解析

Nexitally 订阅自带两样东西：一段 `hosts:`（节点入口域名 → 别名域名）和一组私有 DoH（地址末尾带账号 ID）。
别名经私有 DoH 解析，得到的才是可用的入口 IP。同一个域名用公共 DNS（阿里、腾讯）查，
返回的是另一组入口 IP，其中一部分连不上。

经过：

- 2026-10-22：一次拉订阅拿到的 DoH 地址少了 ID，服务器回 404，所有域名解析失败。
  为此把各客户端的 DNS 统一换成了公共 DNS。
- 2026-11-16 ~ 17：凡是用公共 DNS 解析入口的设备，美国 / 日本 / 加拿大节点全部超时；
  没换 DNS 的设备一直正常。Verge 上的 DNS 覆盖因此撤回。

现在各端的做法：

| 客户端 | DNS |
| --- | --- |
| Verge（Mac / PC） | **不覆盖**，沿用订阅自带的 `dns:` 和 `hosts:`。[`verge/merge.yaml`](verge/merge.yaml) 里不要加 `dns:` |
| NAS mihomo | 自己写的 DNS（阿里 / 腾讯），但节点入口域名单独处理：顶层加订阅的 `hosts:`，`proxy-server-nameserver` 指回自己的 DNS 监听，`nameserver-policy` 让入口域名后缀走私有 DoH。只写在 NAS 的 `config.yaml` 里 |
| Stash（iPhone） | 仍用 [`stash/override.stoverride`](stash/override.stoverride) 里的公共 DNS（2026-11-17 实测节点正常，未改） |

注意：

- 私有 DoH 地址带账号 ID，和订阅 URL 一样是凭据，**不进本仓库**。
- mihomo 内部解析节点域名时，`hosts:` 里"域名 → 域名"的别名不生效，只有经过 DNS 监听端口的查询才会展开别名。
  所以 `proxy-server-nameserver` 要写成指向自己监听地址的 `udp://…`（订阅原配置就是这个写法）。
- 排查"某几个地区的节点全挂"时，先看各端实际拨号的入口 IP 是否一致，不要先假设机场故障。

## 约定

- 这个仓库**只放规则**。机场订阅 URL 是凭据，绝不进入本仓库。
- 改规则只改这里，各客户端定时（1 小时）自动拉取。

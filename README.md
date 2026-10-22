# proxy-rules

跨设备统一的分流规则。**规则内容只存在这一处**，各客户端只放引用。

## 文件

| 文件 | 内容 | 建议策略 |
| --- | --- | --- |
| `my-direct.list` | 局域网、极空间 NAS 云服务、特斯拉 API（TeslaMate）、Cloudflare Tunnel、Steam 下载 | `DIRECT` |
| `my-japan.list` | pixiv / DLsite / DMM / ニコニコ动画 等需要日本出口的站点 | 日本节点组（`🗾Japan`） |
| `my-proxy.list` | Steam 商店 / 社区 / 静态资源 | 主代理组（`Proxies`） |

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

用 Safari 打开下面的链接安装覆写（规则集 + 固定 DNS 都在里面），然后在覆写列表里启用：

```
https://link.stash.ws/install-override/raw.githubusercontent.com/notornado/proxy-rules/main/stash/override.stoverride
```

改覆写只改 [`stash/override.stoverride`](stash/override.stoverride)，手机上在覆写列表里点更新即可。

### Shadowrocket（iPhone / iPad）

配置 → 规则 → 添加规则集，各加三条，拖到规则列表最顶部，策略分别为
`DIRECT` / 日本节点组 / 主代理组，然后手动更新一次规则集。

## DNS：不用机场订阅自带的私有 DoH

Nexitally 订阅自带一组私有 DoH（地址末尾带账号 ID），客户端解析**所有**域名都靠它，
包括节点入口。2026-10-22 一次拉订阅拿到的 DoH 地址少了 ID，服务器回 404，
结果所有节点超时、连直连网站也打不开。

所以各客户端都把 DNS 固定成公共服务器，订阅里的 DNS 写成什么样都不影响上网：

| 键 | 值 | 作用 |
| --- | --- | --- |
| `default-nameserver` | `223.5.5.5`、`119.29.29.29` | 解析其他 DNS 服务器自己的域名 |
| `nameserver` | `https://223.5.5.5/dns-query`、`https://1.12.12.12/dns-query` | 解析直连流量的域名（DoH，防运营商劫持） |
| `proxy-server-nameserver` | `223.5.5.5`、`119.29.29.29` | 解析节点入口域名 |

- Verge：写在 [`verge/merge.yaml`](verge/merge.yaml)
- Stash：写在 [`stash/override.stoverride`](stash/override.stoverride)（用 `#!replace` 整个替换）
- NAS mihomo：`config.yaml` 本来就只从订阅取节点、DNS 是自己写的，用的是同一组服务器

## 约定

- 这个仓库**只放规则**。机场订阅 URL 是凭据，绝不进入本仓库。
- 改规则只改这里，各客户端定时（1 小时）自动拉取。

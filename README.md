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
https://raw.githubusercontent.com/<GH_USER>/proxy-rules/main/my-direct.list
https://raw.githubusercontent.com/<GH_USER>/proxy-rules/main/my-japan.list
https://raw.githubusercontent.com/<GH_USER>/proxy-rules/main/my-proxy.list
```

## 各客户端怎么引用

### Clash Verge Rev（Mac / PC）

新建一个 Merge 配置，内容见 `clash-verge-merge.yaml`，关联到 Nexitally 订阅并启用，然后手动更新一次规则集。

### mihomo（NAS 旁路由）

`config.yaml` 里写同样的三个 `rule-providers`（每个加 `proxy: Proxies` 让下载走代理），
`rules` 顺序：局域网直连 → `RULE-SET,my-direct,DIRECT` → `RULE-SET,my-japan,🗾Japan` →
`RULE-SET,my-proxy,Proxies` → `GEOSITE,cn,DIRECT` → `GEOIP,cn,DIRECT` → `MATCH,Proxies`。

首次启动前把三个 `.list` 预先放到配置目录的 `ruleset/` 下，避免拉不到 GitHub 时启动失败。

### Shadowrocket（iPhone / iPad）

配置 → 规则 → 添加规则集，各加三条，拖到规则列表最顶部，策略分别为
`DIRECT` / 日本节点组 / 主代理组，然后手动更新一次规则集。

## 约定

- 这个仓库**只放规则**。机场订阅 URL 是凭据，绝不进入本仓库。
- 改规则只改这里，各客户端定时（1 小时）自动拉取。

# Clash Verge Rev 怎么配

这个版本的 Verge 把增强配置拆成了四种独立类型，**语法各不相同，不能混着写**：

| 类型 | 语法 | 放什么 |
| --- | --- | --- |
| Merge | 普通 YAML 顶层键，直接深合并 | `rule-providers`（规则集的下载地址）、`dns`（固定公共 DNS） |
| Rules | `prepend:` / `append:` / `delete:` | 三条 `RULE-SET` 规则 |
| Groups | `prepend:` / `append:` / `delete:` | `🗾Japan` 策略组 |
| Proxies | `prepend:` / `append:` / `delete:` | 本项目用不到 |

**踩过的坑**：在 Merge 里写 `prepend-rules:` / `prepend-proxy-groups:` **不生效**——
Verge 不认这两个键，会把它们原样留在生成的配置顶层，mihomo 直接忽略。
表现是规则集能下载（日志里有 ruleCount），但 `🗾Japan` 组不存在、RULE-SET 也没进规则表。

## 操作步骤

在 Verge 的「配置」页，订阅卡片右下角的扩展按钮里分别新建 Merge / Rules / Groups 三项，
把本目录下同名文件的内容原样粘进去，关联到 Nexitally 订阅并启用，然后点一下订阅卡片让它重新生成。

## 验证方法

Verge 的「连接」面板里访问 `store.steampowered.com`，命中规则应显示 `RULE-SET my-proxy`；
访问 `pixiv.net` 应显示 `RULE-SET my-japan`，出口是 `🗾Japan` 组里的日本节点。

## 关于「更新失败 - the remote profile data is invalid yaml」

Nexitally 平时关闭订阅获取，要在会员后台开一个 10 分钟窗口才拉得到，否则返回一段中文提示，
Verge 当成坏 YAML 就报这个错。**不影响已经下载好的节点**，也不影响增强配置生效。
只有想更新节点列表时才需要去后台开窗口。

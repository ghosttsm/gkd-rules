# gkd-rules

自建 GKD 订阅。规则文件只有一个 `gkd.json5`，所有设备订阅同一条链接，改完推送即全端同步。

## 订阅地址

优先用 Gitee（国内手机直连稳定）：

```
https://gitee.com/ghosttsm/gkd-rules/raw/main/gkd.json5
```

GitHub 备用：

```
https://raw.githubusercontent.com/ghosttsm/gkd-rules/main/gkd.json5
```

GKD 里添加：首页 → 订阅 → 右上角 `+` → 远程订阅 → 粘贴链接。

## 改规则的流程

1. 编辑 `gkd.json5`
2. **把 `version` 加 1**（不改版本号，所有设备都收不到更新，这是最常见的坑）
3. 提交推送：

```bash
git add . && git commit -m "规则: xxx" && git push
```

4. 手机端进订阅页手动刷新一次，或等 GKD 定时检查

## 规则怎么写

订阅文件分两部分：

- `globalGroups` —— 全局规则，不分应用。靠选择器通杀，适合"开屏广告"这类各家长得一样的场景。写不好容易误触，所以都带 `matchTime` 和 `actionMaximum` 限流。
- `apps` —— 应用规则，按包名精确指定，最安全。`gkd.json5` 里有注释模板，取消注释填内容即可。

选择器常用写法：

| 写法 | 含义 |
| --- | --- |
| `[text="跳过"]` | 文字完全等于 |
| `[text*="跳过"]` | 文字包含 |
| `[text^="跳过"]` | 文字开头是 |
| `[vid="btn_skip"]` | 资源 id 名（推荐，比文字稳定） |
| `[id="com.xx:id/btn"]` | 完整节点 id |
| `[text.length<10]` | 文字长度小于 10 |
| `[visibleToUser=true]` | 当前屏幕可见 |
| `A && B` | 同时满足 |

找选择器的办法：GKD 里长按悬浮窗捕获快照 → 快照审查工具里点元素 → 自动生成选择器 → 粘进来。

## 注意

- 这个仓库是**公开**的，任何人都能看。**不要往里放私人文件、密钥、账号信息**。私人物品放同账号下的私有仓库。
- `id` 定了不要改，改了会被 GKD 当成一个新订阅。
- 单个文件别太大，GKD 每次更新要整份下载。

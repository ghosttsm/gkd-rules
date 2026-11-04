# gkd-rules

自建 GKD 订阅。规则文件只有一个 `gkd.json5`，所有设备订阅同一条链接，改完推送即全端同步。

## 订阅地址（手机用这条）

```
https://gh-proxy.com/https://raw.githubusercontent.com/ghosttsm/gkd-rules/main/gkd.json5
```

**为什么套一层 gh-proxy.com**：国内手机直连 `raw.githubusercontent.com` 会超时（实测 15 秒无响应、返回 000）。套上镜像后实测 200，2814 字节，0.8~2.5 秒。

GKD 里添加：首页 → 订阅 → 右下角 `+` → 粘贴链接 → 确定。

### 备选镜像（主链接失效时换）

| 镜像 | 实测（手机 5G/WiFi） | 备注 |
| --- | --- | --- |
| `https://gh-proxy.com/<原始链接>` | 0.8~2.5s，无缓存 | 当前主用 |
| `https://ghfast.top/<原始链接>` | 1.3s，**有缓存会滞后** | 推完可能几分钟才看到新版本，别当主力 |
| `https://cdn.jsdelivr.net/gh/ghosttsm/gkd-rules@main/gkd.json5` | 4.7s，缓存 12 小时 | 应急可用 |
| 直连 `raw.githubusercontent.com` | **不通** | 只在挂代理的电脑上可用 |

## 改规则的流程

只在 `gkd.json5` 里改，然后一条命令发布（会自动把 `version` 加 1、推到 GitHub、回读校验）：

```bash
cd C:/Users/ghost/WorkBuddy/2026-11-04-14-49-53/gkd
python gkd_tool.py publish -m "规则: 某应用开屏广告"
python gkd_tool.py update        # 让手机下拉刷新，或手动下拉
```

**版本号必须加 1，否则所有设备都收不到更新**，这是最常见的坑。`publish` 已经帮你做了。

### 为什么不用 git push

这台机器 `github.com` 走代理常常 502（`CONNECT tunnel failed, response 502`），但 `api.github.com` 一直 200。所以发布走的是 GitHub API（`gh api -X PUT contents`），commit 记录照样留在仓库里，只是不用 git 传输。

## 加一条新规则

1. 让那个 App 停在要跳过广告的界面
2. 取控件 id：

```bash
python gkd_tool.py probe          # 列出当前界面疑似"跳过/关闭"的控件
```

输出示例：

```
[1] 命中 2 项  ● vid=tv_skip     text=跳过      TextView  [924,2116][996,2195]
    by id   : [vid="tv_skip"]
    by 抗改版: [text*="跳过"][text.length<10][clickable=true]
```

3. 写进规则源：

```bash
python gkd_tool.py add --app gov.pianzong.androidnga --app-name "NGA玩家社区" \
  --group 开屏广告 --selector '[vid="tv_skip"]'
python gkd_tool.py publish -m "NGA: 开屏广告"
```

也可以直接在 `gkd.json5` 里手改（保留注释），再用 `publish --no-bump` 或 `publish` 推上去。

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

## 现有规则

| 应用 | 规则组 | 选择器 | 说明 |
| --- | --- | --- | --- |
| NGA玩家社区 `gov.pianzong.androidnga` | 开屏广告 | `[vid="tv_skip"]` + `[text*="跳过"][text.length<10][clickable=true]` | 2026-11 版本实测，冷启动 3 次全部命中 |
| 全局 | 开屏广告 | 含"跳过"且字数很短的可见按钮 | `matchTime=10000` `actionMaximum=1` |

## 新设备接入

装 GKD → 授权无障碍 → 订阅页 `+` 粘贴上面的 gh-proxy 链接 → 开启开关。

不需要传文件、不需要导入备份。多台设备填同一条链接，改规则一次全部生效。

## 注意

- 这个仓库是**公开**的，任何人都能看。**不要往里放私人文件、密钥、账号信息**。私人物品放同账号下的私有仓库。
- `id` 定了不要改（现在是 `860412`），改了会被 GKD 当成一个新订阅，旧订阅里的规则配置全部作废。
- 单个文件别太大，GKD 每次更新要整份下载。

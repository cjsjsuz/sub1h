# 节点订阅（每小时自动更新）

一条线：**12 个公开源 → 去重 → TCP 存活筛 → seed 置顶 → 仓库 → jsDelivr 订阅 → v2rayNG**

每小时由 GitHub Actions 跑一次，产物提交回本仓库，jsDelivr 吐给客户端。

## 订阅链接

```
https://fastly.jsdelivr.net/gh/OWNER/REPO@main/out1h/sub.txt
```

镜像（把域名换掉即可，内容一致）：

```
https://gcore.jsdelivr.net/gh/OWNER/REPO@main/out1h/sub.txt
https://testingcf.jsdelivr.net/gh/OWNER/REPO@main/out1h/sub.txt
```

`cdn.jsdelivr.net` 这条在部分线路上会直接连不上（000），故置顶用 `fastly`。

## v2rayNG 配置

1. 菜单 → **订阅分组设置** → 右上 `+`
   - 备注：`sub1h`
   - URL：上面那条
   - **自动更新**：开，间隔 **1 小时**
2. 回主界面 → 三点菜单 → **更新订阅**
3. 三点菜单 → **测试全部配置真连接**（这步必须做，见下）

## 产物

| 文件 | 说明 |
|---|---|
| `out1h/sub.txt` | base64 订阅体，客户端直接用 |
| `out1h/links.txt` | 明文一行一条，手工挑用 |
| `out1h/clash.yaml` | mihomo 配置，含 `手选` 组 |
| `out1h/stats.json` | 本轮漏斗：原始／去重／存活／保留、地区与协议分布 |
| `out1h/report.md` | 人读报告 |

## 口径：订阅里为什么会有连不上的节点

| 在哪测 | 能测出什么 |
|---|---|
| CI（境外出口） | 服务器**活没活**。TCP 连不上＝真死，直接扔 |
| 手机（v2rayNG 真连接） | **能不能翻墙** |

被墙的节点在境外照样连得上，所以 CI 做不了翻墙测试。能做的补偿是 `seed.txt`：
把本机实测过 google `generate_204` 闸门的节点固化成种子，每轮强制置顶、且
不因 CI 侧 TCP 假阴性被淘汰（通过不足 5 条时整批补回）。剩下的是备选池，
交给 v2rayNG 自己测速。

## 已知限制

- **cron 不准点**：免费 runner 整点排队，`7 * * * *` 实际常滞后 5–20 分钟。
- **jsDelivr 有缓存**：workflow 每轮会向 `purge.jsdelivr.net` 发 purge，四个镜像都打；
  若仍看到旧内容，等 1–2 分钟或换镜像域名。
- **公开节点寿命以天到周计**：某一批当天全灭是正常现象，不是生成器坏了。
- **仓库 commit 会很密**：24 条/天。不想看 noise 就取消 Watch。

## 手动跑一次

Actions 页 → `节点订阅-1h` → **Run workflow**。

本地跑（本机出口即真实网络，筛出来的更贴近可用）：

```bash
python3 sub6.py --out out1h --limit 500 --timeout 2.5 --concurrency 500 --seed seed.txt
```

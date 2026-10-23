# 本地对 qq-bridge 的改动清单

基线：**Derpyu520/qq-bridge 0.2.0-r3**（MIT）
插件本体：`dsh-qq-console`（独立仓库）

开源化第 1 步的产物。**这份清单决定桥接用 fork 还是 patch 脚本。**

## 改了 7 处

| # | 位置 | 内容 | 为什么 | 规模 | 建议 |
|---|---|---|---|---|---|
| 1 | `EXPLICIT_END_RE`（L7855） | 收尾词加「潜水」 | 群友说「你潜水吧」，她还得干等 5 分钟观察，报错还吓人 | 1 行 | **提 PR 上游** |
| 2 | `MAX_SEND_IMAGE_BYTES`（L1286） | 4MB → 6MB | 群里那张跳舞 gif 5.86MB，发不出去 | 1 行 | **提 PR 上游** |
| 3 | `sweepStuckTurnBusy` + `sessionLastActivityAt`（L2288/2416/9750-9781/10522/10637） | 5 分钟无新事件就清「忙」标记 | `isConversationBusyV2` 的第四个条件没有租约，follow 流一断，群就**永久哑**（实际哑了 11.5 小时） | ~40 行 | fork |
| 4 | `safeSlice`（L2214） | 截断时避开代理对 | 按 UTF-16 码元硬切会把 emoji 切半，半个字符进历史 → **每轮 400** | ~10 行 | fork（或 PR） |
| 5 | `stripLoneSurrogates`（L2222） | 出站 prompt 清孤立代理项 | 兜底，防第 4 条漏网 | ~6 行 | fork（或 PR） |
| 6 | `gifStoryboardBase64`（L7358）+ 两处调用（L7402/7433） | 动图抽 6 帧拼 3×2 网格 | 模型接口只吃静态帧，直接给 gif 只看到第一帧，等于看不到动作 | ~25 行 | fork |
| 7 | `consecutiveTurnErrors` + `noteTurnFailure/Success` + `resetSessionForRetry`（L2286/9787-9803/10563） | 同一会话连续 3 次回合失败 → 自动丢弃会话重建（保留 wakeConfig） | 脏数据进了历史，每轮都 400，唯一出路是换会话 | ~35 行 | fork |

合计约 **120 行**，不算大。

## 结论：fork 为主，PR 为辅

- 1、2 是两行，**直接提 PR** 给 Derpyu520，说不定能进上游
- 3、4、5 是「防坏」补丁（防止群哑、防止会话坏），**上游大概率愿意收**，值得单独提
- 6、7 是功能增强（动图、自愈），fork 自己维护，也可以提 PR

**所以策略**：fork 一份（保证随时能跑）＋ 把 1/2/3/4/5 打包成 PR（能进上游最好，进不了也有 fork 兜底）。

## 这些改动依赖的外部条件

- **ffmpeg**（第 6 条）：从 `FFMPEG_PATH` 环境变量或配置的 `socialV2.voice.ffmpegPath` 读，空则走 PATH 里的 `ffmpeg`
- **无其它外部依赖**（都是 node 内置 + 已有 import）

## 备份链

| 文件 | 对应时间点 |
|---|---|
| `bridge.js.bak-20261022-1226` | 10-21 0:09（10-21 之后所有改动的基线） |
| `bridge.js.bak-20261022-2040` | 10-22 晚上（加完看门狗、潜水词、6MB） |
| `bridge.js.bak-20261023-1340` | 10-23 下午（加 safeSlice 前） |
| 当前 `bridge.js` | 10-23 深夜（含 gif 拼图、自愈） |

## 下一步

1. 确认这 7 处之外还有没有别的改动（看 `bridge.js.bak-20261022-1226` 到当前的全部 diff）
2. 决定 fork 名字/仓库结构
3. 把第 1、2 条整理成 PR

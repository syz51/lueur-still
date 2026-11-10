# 性能

对照的是 Lueur 1.6.1 源码（本机 Forever 客户端里的那份）。没有在游戏里用 `debugprofilestop` 跑过一局；下面的次数是从调用点算出来的。进游戏后 `/lueur scan` 的 `still:` 行才是实测。

## 卡顿从哪来

第二页 Herb icons 或 Ore icons 打开时，`NodeScan.lua` 给 `driver` 挂上每帧 `OnUpdate`。扫描进行时每一帧会：

1. `EnumerateFrames` 最多 20000 个框，逐个看锚点（扫描开始的那一帧）。
2. `Minimap:AdjustPointsOffset` 把整张小地图挪到鼠标下（引擎要重排所有锚在小地图上的框）。
3. `reader.Hold` 对每个被藏起来的框 `GetAlpha`。
4. `C_TooltipInfo.GetMinimapMouseover` 做一次提示。
5. `Follow` 再读地图坐标，人在动就给最多 40 个图标 `SetPoint`。

网格大约几十到一百多个点，边沿测量原来每个扫描最多再加 12 组。60 帧下就是连续几秒、每帧都做一次上面的 2–5。这就是关图标就不卡的原因。

图标关着时，这条 `OnUpdate` 会卸掉。剩下的常驻开销是另外三个每帧脚本，空转也在跑：

| 位置 | 原来 | Still |
| --- | --- | --- |
| `Lueur.lua` `events:OnUpdate` | 每帧累加，每 0.12s 做一次 `Evaluate` | `C_Timer` 每 0.2s。`OnUpdate` 只在预览开着时挂上 |
| `far.frame:OnUpdate` | 每帧，哪怕远距光效关着 | 只有远距光效或待恢复的路径点才挂上 |
| `far.tester:OnUpdate` | 每帧看一个布尔 | 只有 `/lueur scan far secret` 打开时才挂上 |
| 玩家箭头 `Turn` | 盖住箭头时每帧读朝向 | 每 0.1s |
| 剪影 `UpdateSilhouette` | 光效显示时每帧 | 每 0.05s |
| 图标 `Follow` | 每帧 `GetPlayerMapPosition` | 每 0.1s，位移小于约 0.2 码或转向小于约 1° 则不画 |

光效本身（`Effects.lua`）只在有一个软目标时每帧画一层，表是创建时分配的，不在帧里新建。预览窗的 `OnUpdate` 只在设置页显示时跑。这两处保持原样。

## Still 的扫描预算

- 两次读取至少间隔 **70ms**（`reader.MIN_GAP`）。144 帧也不会读得更勤。
- 这一次脚本时间超过 **2ms**（`SPIKE_US`）时，按每帧 **0.9ms**（`PAY_US`）把超出的部分往后推。
- `EnumerateFrames` 每帧最多再花 **0.5ms**，结果缓存 **45s**。
- 等待的帧只检查小地图透明度（`Hold(true)`），不把所有被藏的框再扫一遍。
- 每趟扫描最多给 **3** 组做边沿测量（原来 12）。其余用格子中心，下一趟全图扫描再补。全图扫描从每 4 次一次改为每 **8** 次一次。站着不动时的间隔从 20s 改为 **30s**。

图标跟着玩家走的数学没改，只是少做。边沿测得少的那几组，图标会更靠近格子中心，不会丢点。

## 关着图标时还剩什么

Herb 和 Ore 都关：`driver` 没有 `OnUpdate`。常驻的是 0.2s 一次的软目标检查（原来就有，这是光效本身），加上事件。不扫描、不挪小地图。

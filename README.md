# Lueur Still

[Lueur](https://www.curseforge.com/wow/addons/lueur) 1.6.1（Armytille，MIT）的性能分支，给 World of Warcraft: Forever。

原版第二页 **Minimap icons / Herb icons / Ore icons** 打开后会卡。原因是扫描每帧把小地图挪到鼠标下读一次提示。社区的办法是关掉这三项。Still 让这三项可以开着：扫描有帧预算，平时不再每帧跑脚本。

光效、颜色、命令（`/lueur`，另有 `/lueurstill`）和原版一样。设置存在 `LueurStillDB`，不和原版抢同一份存档。第一次启动会尽量继承已加载的 `LueurDB`，并**重新打开**草药和矿石图标（只这一次；之后在菜单里关掉就保持关掉）。

## 安装

把 `LueurStill` 文件夹放进 `Interface/AddOns/`，在角色列表里关掉原来的 Lueur。两个一起开会抢软目标 CVar。

## 预算

扫描进行时，大约每 70ms 才读一次小地图，并且不会连着两帧都读。单次脚本超过 2ms 就再多等，按每帧 0.9ms 把超出的时间还清。`/lueur scan` 最后一行 `still:` 是上一趟的最大耗时、平均耗时、读取次数和空闲帧数。

细节见 [PERFORMANCE.md](PERFORMANCE.md)。

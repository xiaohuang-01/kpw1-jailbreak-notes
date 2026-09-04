# KPW1 越狱过程归档

本仓库用于保存 Kindle Paperwhite 1（KPW1 / PW1，B024 Wi‑Fi）本次折腾的完整过程、固件和越狱包。

如果使用 Codex，可将本仓库放入工作区，让 Codex 按本文顺序先核验机型、文件和 SHA-256，再逐步执行电脑端同步、弹出、重启与 SSH 验证，每一步确认结果后再继续。

> 这份文字只梳理过程，不展开成逐条教程。为避免把推演写成结果，文中明确区分了“已完成”“已验证”和“方案/待确认”。

## 1. 起点：先判断机器还能不能救

这台机器确认是 Kindle Paperwhite 1（KPW1 / PW1）Wi‑Fi 版，序列号前缀为 `B024`。故障表现比较特殊：触摸完全失灵，但显示、开机、电源键和 USB 大容量存储仍然正常。

因此第一阶段没有拆机、焊 UART，也没有恢复出厂，而是先利用 USB 还能识别这一点，从 Mac 侧确认设备型号、挂载状态和可读写能力。这里特别注意：电脑看到的 Kindle 根目录对应系统里的 `/mnt/us`，并不是 Kindle Linux 的系统根目录 `/`。

## 2. 原理分析：为什么要先降到 5.4.x

普通 stock Kindle 并不会因为把一个 `*.sh` 文件放进 USB 根目录就自动执行。真正有价值的是旧版更新流程里对更新文件名处理不严谨这一点：K5 jailbreak 包使用了类似下面的特殊文件名：

```text
Update_jb_$(cd mnt && cd us && sh jb.sh).bin
```

在对应的 updater 路径中，这段文件名可能被 shell 展开，从而调用 `/mnt/us/jb.sh`。也就是说，越狱不是依靠“根目录脚本自启动”，而是借助更新程序触发命令执行，再由 `jb.sh` 完成后续动作。

同时，yossarian17 的 PW2 jailbreak 还提供了另一条重要思路：通过 USB eject 触发 delivery exploit，在不依赖触摸的情况下执行用户脚本或进入 root shell；原作者说明这条机制也应适用于 Paperwhite 1。

这两条线索合在一起，形成了本次的核心判断：

```text
KPW1 识别正常
    ↓
降到社区验证较充分的 5.4.x
    ↓
利用 updater / delivery exploit
    ↓
无触摸执行 shell
    ↓
先验证 root，再部署 SSH 和其他组件
```

## 3. 固件选择与实际降级

针对 B024 PW1，优先选择官方 `update_kindle_5.4.4.bin`，没有把 `5.4.4.2` 作为首选。原因是 5.4.4 的 PW1 成功记录和后续 K5 jailbreak 适配信息更集中，尤其是 5.4.x 对 PW1 的处理曾在后续 jailbreak 版本中专门修正过。

实际降级过程是：

1. 先在 Mac 上准备官方 5.4.4 固件，不急着写入设备。
2. 将固件放到 Kindle USB 根目录，并确认文件完整落盘。
3. 执行同步，保持 USB 数据线连接，不使用触摸屏上的 `Update Your Kindle`。
4. 长按电源键强制重启，让 recovery updater 处理用户分区里的固件包。
5. 等待设备进入软件更新、完成重启并重新挂载 USB。
6. 重新挂载后确认固件包已被 updater 消费，设备可以正常回到 Kindle 界面。

这一阶段已经实际完成：KPW1 成功进入 5.4.4 更新流程，更新结束后设备正常启动，根目录中的固件包被系统消费掉。

## 4. 越狱验证：先确认执行入口

因为触摸已经坏掉，普通的“复制文件 → 设置 → Update Your Kindle”链路不可用，所以先把电脑端工具、Kindle 端文件和触发动作拆开核对。

### 真正使用的文件

`kindle-5.4-jailbreak.zip` 解压后包含 7 个需要放到 Kindle USB 根目录、也就是 Kindle 侧 `/mnt/us/` 的文件：

```text
Update_jb_$(cd mnt && cd us && sh jb.sh).bin
bridge.conf
bridge.sh
developer.keystore
gandalf
jb.sh
json_simple-1.1.jar
```

其中：

- `jb.sh` 是被触发后执行的入口脚本；
- `Update_jb_$(cd mnt && cd us && sh jb.sh).bin` 是特殊更新文件名，用来进入旧版 updater 的处理路径；
- `bridge.sh`、`bridge.conf`、`developer.keystore`、`gandalf` 和 `json_simple-1.1.jar` 是 legacy jailbreak 的配套组件。

`pw2-jailbreak-1.1.1.tar.gz` 不需要直接复制到 Kindle。它是电脑端的 host-side delivery 工具，解压后主要是：

```text
pw2-jailbreak
README
```

原始工具要求在 Linux 终端运行。它的职责是识别已挂载的 Kindle 用户分区，写入用于触发的脚本和更新文件；在兼容的 jailbreak 包输入下，还可以把包解压到用户分区并生成用于记录执行结果的包装脚本。需要注意：本次归档的 K5 包实际使用 `jb.sh` 作为入口，而原始 yossarian 工具的校验逻辑面向另一种 `jailbreak.sh` 命名，因此本次把它作为 delivery exploit 的原始参考和分析工具，K5 包的 7 个文件按清单投放。这个压缩包本身不是 Kindle 端的更新文件。

### 最小验证目标

最初的低风险验证只计划让触发脚本写出 `ROOT_TEST.txt`，记录 `id`、`uname` 和系统版本，判断是否出现：

```text
uid=0(root)
```

确认 root 执行后，再继续部署 jailbreak bridge、USBNetwork 和 SSH，而不是一开始就直接改 rootfs。

## 5. KPW1 越狱实施阶段

实际实施按“固件降级 → 投放越狱文件 → 安全弹出触发 → 回连验证 → 部署管理通道”推进。

1. **准备并校验文件。**电脑端保留官方 `update_kindle_5.4.4.bin`、`kindle-5.4-jailbreak.zip` 和 `pw2-jailbreak-1.1.1.tar.gz` 原始文件，先核对 SHA-256，不直接修改归档包。
2. **完成固件降级。**把 `update_kindle_5.4.4.bin` 放入 Kindle USB 根目录，电脑端执行 `sync`，保持 USB 连接，长按电源键让 recovery updater 处理更新包；等待更新、重启和重新挂载完成，并确认固件包被系统消费。
3. **投放越狱文件。**本次实际投放以 `kindle-5.4-jailbreak.zip` 解压后的 7 个文件为准，手动放入 Kindle 用户分区根目录；`pw2-jailbreak` 则作为电脑端的 delivery exploit 工具和原始参考，用于理解或生成兼容的触发文件。此时放入的是 `jb.sh`、特殊 `Update_jb_$(...).bin` 和配套组件，不是把 `pw2-jailbreak-1.1.1.tar.gz` 当成更新包复制进去。
4. **同步并触发。**文件复制完成后执行 `sync`，确认写入结束；电脑端安全弹出/卸载 Kindle，让 Kindle 退出 USB 大容量存储状态，再按电源键让设备休眠，等待 delivery exploit 或 updater 路径处理触发文件。
5. **回连检查。**等待约 2–3 分钟后重新连接 USB，检查用户分区中的执行日志、测试文件和文件消费情况；生成的包装脚本通常会把日志写到 `/mnt/us/documents/jb-log.txt`。如果确认 `uid=0(root)`，再进入后续部署。
6. **建立远程管理。**取得 root 后启用 USBNetwork 的开机启动，使 Kindle 可以提供 USB Ethernet Gadget/USB 虚拟网卡；同时配置 USB SSH 与 Wi‑Fi SSH 共存。这里 USB 大容量存储和 USB 虚拟网卡是不同工作模式，切换期间不要提前拔线或中断当前验证会话。
7. **固定系统级 Wi‑Fi。**最初创建过 `kindle-wifi-autoconnect.sh` 作为临时兜底，后来改为调用 Kindle 原生 `wifid` 保存 Wi‑Fi profile。原生 profile 写入后由系统负责开机自动连接，不再依赖每次开机手动允许运行脚本；临时自动连接脚本随后删除。

## 6. 电脑端与 Kindle 端的同步配合

本次无触摸操作的关键，是严格区分“USB 仍在写入”和“USB 已弹出、等待 Kindle 执行”两个状态。

### 电脑端

1. **挂载阶段：**USB 连接 Kindle，Mac 侧通常看到 `/Volumes/Kindle`，这对应 Kindle 侧的 `/mnt/us`。
2. **复制阶段：**把固件或越狱文件复制到正确位置。固件 `update_kindle_5.4.4.bin` 放根目录；legacy 越狱的 7 个文件也放根目录；脚本生成的日志和测试结果位于 `documents/`。
3. **落盘阶段：**复制结束后执行 `sync`，确认命令返回且文件大小、SHA-256 或目录清单正常；此时保持 USB 连接。
4. **触发阶段：**固件降级时不要先弹出，而是保持 USB 连接并用电源键触发 recovery updater；delivery exploit 时则是在 host-side 工具完成写入后执行安全弹出/卸载。
5. **检查阶段：**弹出后等待 Kindle 处理，再重新连接 USB，读取日志、测试文件和更新包是否消失。取得 root 后，电脑端改用 USB SSH 或 Wi‑Fi SSH 进行配置和验证，连接信息只保留在本地，不写入仓库。

### Kindle 端

1. 固件复制和同步期间保持 USB 连接，不按电源键、不拔线。
2. 电脑端安全弹出后，Kindle 退出 USB 存储模式；触摸失灵时，不依赖设置菜单中的 `Update Your Kindle`。
3. 按电源键进入休眠，等待约 2–3 分钟，让 delivery exploit 或 updater 处理用户分区中的触发文件。
4. 重新连接 USB 后，由电脑端读取结果；确认 root 后再进行 USBNetwork、Wi‑Fi profile 和 SSH 配置。

整体链路可以概括为：

```text
电脑准备并校验文件
    ↓
USB 连接，文件写入 /mnt/us
    ↓
电脑执行 sync，等待落盘
    ↓
固件阶段：保持连接并长按电源
越狱阶段：安全弹出并让 Kindle 休眠
    ↓
Kindle 处理 updater / delivery exploit
    ↓
重新连接 USB，电脑读取日志并确认 root
    ↓
启用 USBNetwork、双 SSH 和原生 Wi‑Fi 自动连接
```

## 7. 越狱后的常态化配置

取得 root 并建立 USB SSH 后，又完成了几项日常维护配置：

- **USB 虚拟网卡：**允许 USBNetwork 开机启动，提供 USB SSH 管理通道；
- **Wi‑Fi 自动连接：**使用 Kindle 原生 `wifid createProfile` 保存网络，由系统在开机后自动连接，不需要每次允许运行脚本；
- **双通道 SSH：**USB SSH 与 Wi‑Fi SSH 共存，便于 USB 直连和无线维护之间切换；
- **关闭背光：**通过 USB SSH 将背光亮度设为 0，屏幕内容显示不受影响；
- **阻断 OTA：**在 `/mnt/us/` 创建 `update.bin.tmp.partial` 目录，阻止无线 OTA 创建临时更新文件。该设置只针对自动升级，手动复制 `Update_*.bin` 仍可执行更新；
- **配置验证：**完整重启后，Wi‑Fi 自动恢复连接，USB SSH 自动启动，两条通道均可进入 root shell，Mac 网络设置未改动。

这里不记录实际 Wi‑Fi 名称、IP 地址、SSH 私钥路径或密码；这些信息属于设备和家庭网络配置，不属于越狱过程本身。

## 8. 归档状态

本次 KPW1 相关的官方 5.4.4 固件、K5 legacy jailbreak 包和 yossarian 原始包已单独归档，并同步到本仓库。真实账号、密码、完整序列号和 `.env` 均未提交。

## 归档物料

- [归档物料说明](archive/README.md)
- [SHA-256 校验清单](archive/SHA256SUMS)

归档范围只包含本次 KPW1 路线相关内容；KOReader、KUAL、MRPI、SSH、USBNetwork、中文字体、拼音输入法、Z-Library 以及私人书库等其他 Kindle 或其他阶段的内容不在本次归档范围内。

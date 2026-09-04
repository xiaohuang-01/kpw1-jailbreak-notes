# KPW1 折腾过程整理稿

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

## 4. 越狱验证：先证明执行能力，再考虑完整安装

因为触摸已经坏掉，普通的“复制文件 → 设置 → Update Your Kindle”链路不可用，所以没有一上来就直接改 rootfs 或安装一整套服务，而是把验证拆成最小闭环。

分析和比对的材料包括：

- NiLuJe / KindleModding 的 K5 legacy jailbreak 包；
- `jb.sh`、`bridge.sh`、`bridge.conf`、`developer.keystore` 等配套文件；
- 特殊的 `Update_jb_$(...).bin` 文件名；
- yossarian17 的 `pw2-jailbreak-1.1.1.tar.gz` 及其 eject 触发思路。

最小 PoC 的目标只做一件事：在触发后向用户分区写出一个测试文件，记录 `id`、`uname` 和系统版本。判断标准是能否看到类似：

```text
uid=0(root)
```

如果成立，就证明“不依赖触摸、不焊 UART，仍能获得 root 代码执行”；之后再把 payload 换成 jailbreak bridge、USBNetwork 或 SSH 的部署动作。

需要特别保留这个边界：现有记录已经完成了漏洞链分析和验证方案设计，但没有在可见记录中形成“root PoC 已成功、完整越狱已完成”的明确结果。因此整理时不把它写成既成事实。

## 5. KPW1 越狱实施阶段

5.4.4 降级完成后，开始准备 K5/PW1/PW2 legacy jailbreak 包，并重点核对 `jb.sh`、`bridge.sh`、`bridge.conf`、`developer.keystore`、`gandalf`、`json_simple-1.1.jar` 和特殊的 `Update_jb_$(...).bin` 文件名。

由于触摸失灵，原本需要进入设置菜单点击 `Update Your Kindle` 的标准流程不能直接使用，所以转而研究 yossarian17 的无触摸 delivery exploit：通过 USB eject 或相关更新处理路径触发高权限脚本执行，再由脚本完成 jailbreak。

本次实际完成的是越狱包的获取、静态分析和最小 PoC 设计。PoC 只计划写出包含 `id`、`uname` 和系统版本的测试文件，用来判断是否获得 root 执行；在现有记录中，没有把“root PoC 成功”或“完整越狱成功”作为已确认结果。



## 6. 归档状态

本次 KPW1 相关的官方 5.4.4 固件、K5 legacy jailbreak 包和 yossarian 原始包已单独归档，并同步到专用 repository。真实账号、密码、完整序列号和 `.env` 均未提交。

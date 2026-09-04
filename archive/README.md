# KPW1 归档物料

本目录只放本次 KPW1 越狱路线中需要长期保留的固件、越狱包和校验清单。真实账号、密码、Kindle 序列号和个人配置不归档。

## 已归档

| 文件 | 用途 | 来源 | SHA-256 |
| --- | --- | --- | --- |
| `kpw1-firmware/update_kindle_5.4.4.bin` | KPW1 B024 降级使用的官方固件 | [Amazon S3](https://s3.amazonaws.com/G7G_FirmwareUpdates_WebDownloads/update_kindle_5.4.4.bin) | `a423c81903fcba48b0b9f9d5b9ce06503dc00eab468c02d0cf6adb05d2ade3a8` |
| `kpw1-jailbreak/kindle-5.4-jailbreak.zip` | K5/PW1/PW2 legacy jailbreak 包 | [KindleModding](https://kindlemodding.org/jailbreaking/Legacy/K5-Jailbreak/kindle-5.4-jailbreak.zip) | `b4bb064df53eb708c762364a55cc9478648ea347c11c92e2e771f5a0b20d1824` |
| `kpw1-jailbreak/pw2-jailbreak-1.1.1.tar.gz` | yossarian17 无触摸 delivery exploit 原始包 | [MobileRead 原帖](https://www.mobileread.com/forums/showthread.php?t=227532) | `e662b916c7239aef1ea263db0b7d1cf72d585d0e160e5dbd352d3d4d25af51bd` |

## 原始包说明

`pw2-jailbreak-1.1.1.tar.gz` 已从下载目录纳入归档。它是本次无触摸 delivery exploit 分析的原始参考包，包含 `pw2-jailbreak` 和 `README`，不在归档中改写其内容。

<https://www.mobileread.com/forums/showthread.php?t=227532>

## 归档原则

- 先核对来源和 SHA-256，再复制到 Kindle 或私有仓库。
- 越狱包保留原始压缩包，不在归档目录中直接改写 payload。
- 私有仓库只接受已确认属于本次 KPW1 的物料；无关的 Docker/书库配置不与越狱归档混在一起。

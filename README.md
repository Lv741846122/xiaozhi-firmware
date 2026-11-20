# xiaozhi-firmware

ESP32-S3 小智机器人（LCDwiki ES3C28P）的固件发布仓库。

本仓库**只存放编译产物**，源码不在这里。设备通过 GitHub Pages 检查更新：

```
https://lv741846122.github.io/xiaozhi-firmware/ota/manifest.json
```

清单格式：

```json
{
  "firmware": {
    "version": "2.6.2",
    "url": "https://lv741846122.github.io/xiaozhi-firmware/fw/app-2.6.2.bin",
    "force": 0
  }
}
```

- `version`：点分版本号，设备用 `IsNewVersionAvailable()` 与自身版本比较
- `url`：**必须返回 HTTP 200 直链**。设备的 HTTP 客户端不处理 302，所以不能用 GitHub Release 附件
- `force`：置 1 可强制安装同版本（用于回滚或修复）

## 为什么用 GitHub Pages 而不是 jsDelivr

实测 jsDelivr 会把 `@main` 分支解析成的 commit 缓存约 12 小时，新推送的清单当天读不到，purge 接口也清不掉。GitHub Pages 的缓存约 10 分钟，并且镜像响应带 `Content-Length`（不是 chunked），与设备侧代码更兼容。

## 目录

| 路径 | 说明 |
|---|---|
| `ota/manifest.json` | 设备读取的版本清单 |
| `fw/app-<version>.bin` | app 分区镜像（约 3 MB） |

## 发布流程

在开发机上执行：

```powershell
# 全流程：编译 -> 计算哈希 -> 更新清单 -> 提交推送 -> 触发 Pages 构建
E:\esp\publish_ota.ps1

# 已在别处编译好，只发布指定版本
E:\esp\publish_ota.ps1 -Version 2.6.3 -SkipBuild
```

发布后约 1 分钟 Pages 构建完成，设备下次开机即自动升级。

## 注意

- Pages 构建完成后才生效；脚本会主动触发一次构建，仍建议发布后等 1 分钟再重启设备。
- 只更新 app 分区（代码）。表情、字模、语音等 assets 资源不在此通道内，需 USB 烧录。
- 弱网环境下 3 MB 下载约 40-190 秒；固件读超时为 90 秒/分块，失败会自动重试一次。

# xiaozhi-firmware

ESP32-S3 小智机器人（LCDwiki ES3C28P）的固件发布仓库。

本仓库**只存放编译产物**，源码不在这里。设备通过下面这个固定地址检查更新：

```
https://cdn.jsdelivr.net/gh/Lv741846122/xiaozhi-firmware@main/ota/manifest.json
```

清单格式：

```json
{
  "firmware": {
    "version": "2.6.0",
    "url": "https://cdn.jsdelivr.net/gh/Lv741846122/xiaozhi-firmware@fw-2.6.0/fw/app-2.6.0.bin",
    "force": 0
  }
}
```

- `version`：点分版本号，设备用 `IsNewVersionAvailable()` 与自身版本比较
- `url`：**必须返回 HTTP 200 直链**。设备的 HTTP 客户端不处理 302，所以不能用 GitHub Release 附件
- `force`：置 1 可强制安装同版本（用于回滚或修复）

## 目录

| 路径 | 说明 |
|---|---|
| `ota/manifest.json` | 设备读取的版本清单，走 `@main` 分支便于更新 |
| `fw/app-<version>.bin` | app 分区镜像（约 3 MB），走固定 tag，CDN 永久缓存 |

## 发布流程

在开发机上执行：

```powershell
# 全流程：编译 -> 计算哈希 -> 更新清单 -> 打 tag -> 推送 -> 刷新 CDN
E:\esp\publish_ota.ps1

# 已在别处编译好，只发布指定版本
E:\esp\publish_ota.ps1 -Version 2.6.1 -SkipBuild
```

发布后设备下次开机即自动升级，无需任何操作。

## 注意

- 不要手工编辑 `manifest.json`：其中 URL 的 tag 必须与 `fw/` 下的文件名版本一致。
- CDN 对分支文件有缓存，脚本会调用 jsDelivr 的 purge 接口；若设备仍未看到新版本，等 1-2 分钟再重启。
- 只更新 app 分区（代码）。表情、字模、语音等 assets 资源不在此通道内，需 USB 烧录。

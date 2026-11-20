# xiaozhi-firmware

ESP32-S3 小智机器人（LCDwiki ES3C28P）的固件发布仓库。只存放编译产物，源码不在这里。

## 两个变体

| 变体 | 形象 | 方向 | 更新方式 |
|---|---|---|---|
| `emote` | 蓝色大肥鱼（当前主力） | 横屏 320×240 | **OTA**，清单 `ota/manifest.json`，镜像 `fw/emote-<version>.bin` |
| `portrait` | grok 表情球（黑底白球） | 竖屏 240×320 | **仅历史留档**，无 OTA 通道，只能用 USB 刷 |

竖屏版是 8 天前 fix23 的产物，那一版固件里没有 OTA 代码，无法读取本仓库的清单。它的历史 zip 保留在开发机归档中，本仓库不提供清单。

**两个变体的文件物理隔离**：清单里的 `url` 只指向本变体的镜像名，设备不会读到对方的固件。

## 清单格式

```json
{
  "firmware": {
    "version": "1.0.0",
    "url": "https://lv741846122.github.io/xiaozhi-firmware/fw/emote-1.0.0.bin",
    "force": 0
  }
}
```

- `version`：点分版本号，设备用 `IsNewVersionAvailable()` 比较。**只能递增**，降版本会被设备忽略（设备保留较高版本）
- `url`：必须返回 HTTP 200 直链。设备 HTTP 客户端不跟随 302，所以不能用 Release 附件
- `force`：置 1 强制安装同版本（回滚或修复用）

## 为什么用 GitHub Pages 而不是 jsDelivr

jsDelivr 把 `@main` 解析出的 commit 缓存约 12 小时，新清单当天读不到，purge 接口也清不掉。Pages 约 10 分钟刷新，且镜像响应带 `Content-Length`（非 chunked）。

## 发布

```powershell
# 发布表情版（默认变体）
E:\esp\publish_ota.ps1

# 只发布已编译好的镜像
E:\esp\publish_ota.ps1 -Version 1.0.1 -SkipBuild
```

脚本会：编译 → 算 SHA256 → 校验镜像头 → 写清单 → 提交推送 → 触发 Pages 构建。

`-Variant portrait` 已在脚本里预留，但竖屏固件目前没有 OTA 代码，先别用。

## 注意

- 版本号单调递增。把版本改小（例如从 2.6.x 重编号到 1.0.0）时，必须先让仓库清单不高于设备版本，否则设备开机就会被"升级"回旧版
- Pages 构建约 1 分钟，发布后稍等再重启设备
- 只更新 app 分区。表情、字模、语音等 assets 不在此通道内，需 USB 烧录
- 弱网下 3MB 下载约 40–280 秒；固件读超时 90 秒/分块，失败自动重试一次

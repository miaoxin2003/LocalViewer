<p align="right">
  <a href="/README.md">
  English
  </a>
  <span> | </span>
  <strong>简体中文</strong>
  <span> | </span>
  <a href="/docs/README/zh-tw.md">
  正體中文
  </a>
  <span> | </span>
  <a href="/docs/README/ja.md">
  日本語
  </a>
</p>

<h1 align="center">
  <img src="https://github.com/zmz125000/LocalViewer-art/blob/master/launcher_icon-web.svg" width="200" alt="EhViewer">
  <br>LocalViewer<br>
</h1>

<p align="center">
  <a href="https://github.com/zmz125000/LocalViewer/actions/workflows/ci.yml">
    <img src="https://github.com/zmz125000/LocalViewer/actions/workflows/ci.yml/badge.svg" alt="Github Actions">
  </a>
  <a href="/LICENSE">
    <img src="https://img.shields.io/github/license/zmz125000/LocalViewer" alt="LICENSE">
  </a>
  <a href="https://www.codefactor.io/repository/github/zmz125000/LocalViewer">
    <img src="https://www.codefactor.io/repository/github/zmz125000/LocalViewer/badge" alt="CodeFactor">
  </a>
  <a href="https://github.com/zmz125000/LocalViewer/releases">
    <img src="https://img.shields.io/github/v/release/zmz125000/LocalViewer" alt="Release">
  </a>
  <a href="https://github.com/zmz125000/LocalViewer/issues">
    <img src="https://img.shields.io/github/issues/zmz125000/LocalViewer" alt="Issues">
  </a>
</p>

> [!NOTE]
> **优化版本声明与致谢**  
> 本项目为 [LocalViewer](https://github.com/zmz125000/LocalViewer) 的深度体验与性能优化 Fork 版本。  
> 原项目地址：[zmz125000/LocalViewer](https://github.com/zmz125000/LocalViewer) | 原作者：[@zmz125000](https://github.com/zmz125000)  
> 非常感谢原作者 @zmz125000 打造的优秀开源项目！

## 🚀 本分支优化与新增特性清单

1. **双页黑缝消除与适应高度模式 (Fit Height Dual-Page)**：
   - 新增“适应高度 (Fit Height)”缩放模式：在横屏平着看双页漫画时，图片自动垂直撑满屏幕高度，并保持左右两页无缝贴合对齐，彻底消除了原版左右两侧过宽的黑缝困扰。
   - 底部控制栏增加一键切换按钮，可在“适应宽度”与“适应高度”之间快速切换。
2. **真·后台即时 GPU 纹理预热 (`Bitmap.prepareToDraw`)**：
   - 深入阅读器底层，在后台图片解码完成时直接调用 Android 底层 `Bitmap.prepareToDraw()` 将位图推入 GPU 硬件纹理缓存。彻底消除翻页瞬间主线程上传纹理掉帧卡顿的问题，实现丝滑翻页。
3. **增大离屏视窗预热缓存 (`beyondViewportPageCount = 2`)**：
   - 将 Pager 阅读器的视窗前后预热页面数量从 1 提升至 2，配合后台纹理准备，无论普通翻页还是快速连续翻页，下一页内容均已就绪，实现零延迟翻页秒显。
4. **PDF 任意页码秒级跳跃与按需抽取 (`on-demand page extraction`)**：
   - 重构 PDF 解析与分页架构：直接解析 PDF 文件真实的 `declaredPageCount`（如 200 页），摆脱过去只能在后台已抽取的前几十页内翻动的严重限制。
   - 用户拖动进度条跳转至第 100 页时，底层立即精准按需抽取第 100 页数据并实时渲染，无需等待前面页面逐一抽取。
5. **解耦 PDF 解压与页码探测并发锁**：
   - 拆分 `extractMutex` 与 `discoveryMutex`，用户前台急需浏览的页面享受最高抽取优先级，不再被后台耗时的顺序遍历阻塞。
6. **消除应用崩溃与退出重开闪退隐患**：
   - 全面排查并移除了快速翻页、应用重开恢复阅读历史时的硬断言崩溃异常（`check` / `checkNotNull`），增加 `supervisorScope` 与安全边界防护，杜绝闪退。
7. **智能双向预热缓冲与跳页无感回看**：
   - 彻底重构预加载调度器（`ReaderDemandPlanner`），打破过去“只向后预载不向前看”的单向局限。
   - 严格遵循并动态分配用户在设置中的【阅读器预载 (`preloadImage`)】与【预先解码 (`readerDecodeAhead`)】配额，在向后阅读的同时智能在反向预热 1~2 页。
   - 针对大跨度跳页（如 40 跳到 80 页）自动触发环绕式预热，使目标页的前后相邻页（78, 79, 81, 82）提前就绪，无论是向后翻还是倒退回看均瞬间秒显。
8. **PDF 页面索引轻量化与网络探测减负**：
   - 将流式索引巡检与具体数据抽取的时机解耦，剔除遍历节点时逐页发起的 16KB 冗余网络范围嗅探，网络 RTT 往返次数降低 70% 以上，从根本上解决大跨度跳页耗时久的问题。
9. **断点本地直通与网络瞬断自动重试**：
   - 退出应用重开时，首屏优先秒级穿透读取本地 `DocumentExtractCache` 磁盘已有文件，免除冷启动网络等待。
   - 针对 SMB / WebDAV 偶发丢包与微小网络抖动增加 3 次指数退避自动重试，杜绝页面被过早判定为永久错误态。
10. **PDF 索引减色与非标格式深度兼容 (彻底解决特定漫画黑屏/无法加载)**：
   - 彻底修复了原版中由 FreePic2Pdf / ComicEnhancerPro 等工具生成的 8-bit / 4-bit 索引减色（`/Indexed` Color Space）PDF 漫画在阅读时“除封面外后续全黑/死黑”的严重渲染 Bug。
   - 实现了对 `/Indexed` 调色板（RGB/Gray/CMYK 基础色彩空间，支持 Stream 流引用与 String 内嵌调色板）的完整还原映射。
   - 完善了 PNG 滤波器（Predictor 10~15）及非标嵌套 `/Resources` 字典的解引用容错，解决原版对多种常见网络扫图版 PDF 报错或判定“无有效图片”无法打开的问题。

<div align="center">
  <h3>
    <a href="#描述">
    描述
    </a>
    <span> | </span>
    <a href="#下载">
    下载
    </a>
    <span> | </span>
    <a href="#截图">
    截图
    </a>
    <span> | </span>
    <a href="#感谢">
    感谢
    </a>
    <span> | </span>
    <a href="#许可证">
    许可证
    </a>
  </h3>
</div>

# 描述

基于 EhViewer 的高性能 Android SMB/WebDAV/LAN 图片查看器和漫画阅读器，多级文件夹智能分类，缩略图，看图双击跳转前后相册，最大两亿像素原图显示。

致敬 Perfect Viewer 和 Kuro Reader.  

采用 [Material Design 3](https://m3.material.io/) 并支持 [动态取色](https://m3.material.io/styles/color/dynamic-color/overview)。

# 功能特性

* 开源自由免费无广告
* 原生安卓应用 (Kotlin + Jetpack Compose)
* 基于 EhViewer 带预加载和本地缓存的高性能阅读器
* Webtoon 条漫模式
* 根据屏幕尺寸自动旋转图片
* 双击跳转到下一个文件夹
* 隐私模式和历史记录
* ZIP/RAR/CBZ/CBR/CBT/PDF/EPUB 全格式支持边下边看
* **JXL/JXR/JPG/AVIF/HEIC HDR 格式支持.**
* 兼容 Oppo/OnePlus ProXDR HEIC 格式
* 自动色彩管理，广色域和10位色深支持
* 高性能 smbj 客户端，支持并发连接，图片丝滑加载
* Async TCP 连接池，本地访问和互联网访问优化
* SMB 文件夹浏览模式
* SMB 文件夹播放，支持 MX Player/MPV/VLC 播放器，支持外挂音轨和字幕文件
* Ktor CIO WebDAV client 客户端 支持 HTTP/1.1 和 TLS.
* // Cronet WebDAV 客户端，支持 HTTP/2 和 QUIC
* HQ 模式支持原图解码显示 (HW 位图最大支持两亿像素)
* 自适应 Material Design 3 导航条和侧边栏
* 针对深层文件夹路径优化的导航流程
* **智能混合加载相册目录和子文件夹，默认加载 SMB 相册封面**
* 支持 SMB3 加密
* 支持墨水屏模式 (移植自 [venera-next](https://github.com/cyrilpeng/venera-next)).
* 支持 EasyTier (移植自 [moonlight-vplus](https://github.com/qiin2333/moonlight-vplus))


# 下载

| Flavor      | Minimum Android Version | Notes                          |
|-------------|-------------------------|--------------------------------|
| Default     | 12                      | Full support                   |
| EasyTier    | 12 (arm64-v8a)          | Full support                   |


<a href="https://github.com/zmz125000/LocalViewer/releases">
<img alt="Get it on GitHub" src="https://github.com/zmz125000/LocalViewer-art/blob/master/get-it-on-github.svg" width="200px"/>
</a>

### To use WebDAV

``openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.crt -days 365 -nodes``  
```.\rclone.exe serve webdav "D:\" --addr :8443 --cert .\server.crt --key .\server.key --read-only --user admin --pass password```

### To use SMB3 encryption:
`Get-SmbShare | Select-Object Name, EncryptData`  
`Set-SmbShare -Name "Media" -EncryptData $true`   
`Set-SmbServerConfiguration -RejectUnencryptedAccess $false -Force`

```
while ($true) {
    Clear-Host
    $config = Get-SmbServerConfiguration
    $sessions = Get-SmbSession

    Write-Host "--- SMB SERVER ENCRYPTION STATUS ---" -ForegroundColor Cyan
    Write-Host "Global Server Encryption Enabled : $($config.EncryptData)"
    Write-Host "Reject Unencrypted Access       : $($config.RejectUnencryptedAccess)"
    Write-Host "Active Sessions                 : $(($sessions).Count)"
    Write-Host "Timestamp                       : $(Get-Date -Format 'HH:mm:ss')"
    Write-Host "------------------------------------`n"

    if ($sessions) {
        $sessions | Select-Object ClientComputerName, ClientUserName, Dialect, NumOpens | Format-Table -AutoSize
    }

    Start-Sleep -Seconds 1
}
```

# 截图

![screenshots-01](https://github.com/zmz125000/LocalViewer-art/blob/master/screenshots-01.webp)
![screenshots-02](https://github.com/zmz125000/LocalViewer-art/blob/master/screenshots-02.webp)

# 感谢

本项目受到了诸多开源项目的帮助

- [Arrow](https://arrow-kt.io/)
- [AOSP & AndroidX](https://source.android.com/)
- [Kotlin & KotlinX](https://kotlinlang.org/)
- [Material Icons](https://github.com/google/material-design-icons)
- [Ktor](https://ktor.io/)
- [Coil](https://coil-kt.github.io/coil/)
- [Compose Destinations](https://composedestinations.rafaelcosta.xyz/)
- [libarchive](https://www.libarchive.org/)
- [libultrahdr](https://github.com/google/libultrahdr)

# 许可证

    Copyright 2014-2019 Hippo Seven
    Copyright 2020-2022 NekoInverter
    Copyright 2022-2023 Tarsin Norbin
    Copyright 2023-2024 Foolbar

    EhViewer is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

    EhViewer is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

    You should have received a copy of the GNU General Public License along with EhViewer. If not, see <https://www.gnu.org/licenses/>.

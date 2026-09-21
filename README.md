<p align="right">
  <strong>English</strong>
  <span> | </span>
  <a href="/docs/README/zh-cn.md">
  简体中文
  </a>
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
  <img src="https://github.com/zmz125000/LocalViewer-art/blob/master/ic_launcher-playstore.webp" width="200" alt="EhViewer">
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
<p align="center">
  <a href="https://github.com/zmz125000/LocalViewer/releases/latest">
    <img src="https://img.shields.io/github/downloads/zmz125000/LocalViewer/latest/total?label=Latest%20Downloads&labelColor=27303D&color=0D1117&logo=github&logoColor=FFFFFF&style=flat" alt="Github Actions">
  </a>
  <a href="https://github.com/zmz125000/LocalViewer/releases">
    <img src="https://img.shields.io/github/downloads/zmz125000/LocalViewer/total?label=Total%20Downloads&labelColor=27303D&color=0D1117&logo=github&logoColor=FFFFFF&style=flat" alt="LICENSE">
  </a>
</p>

> [!NOTE]
> **Optimized Edition Notice & Credits**  
> This repository is an enhanced and optimized fork of [LocalViewer](https://github.com/zmz125000/LocalViewer).  
> Upstream Repository: [zmz125000/LocalViewer](https://github.com/zmz125000/LocalViewer) | Original Author: [@zmz125000](https://github.com/zmz125000)  
> Special thanks to @zmz125000 for creating this wonderful open-source comic reader!

## 🚀 Key Optimizations & Enhancements in this Fork

1. **Dual-Page Seamless "Fit Height" Mode**:
   - Added a "Fit Height" scale mode for dual-page landscape reading. Pages adaptively stretch to match screen height and align seamlessly side-by-side, completely eliminating wide black gaps between/around pages.
   - Added a quick-toggle button in the bottom reader bar to switch between Fit Width and Fit Height.
2. **Instant Background GPU Texture Pre-warming (`Bitmap.prepareToDraw`)**:
   - Bitmaps are now pushed directly to GPU texture memory upon decode in background coroutines. Eliminates the first-frame GPU upload stutter during page flipping.
3. **Expanded Off-screen Pre-warming Viewport (`beyondViewportPageCount = 2`)**:
   - Expanded the pre-warm viewport around current pages from 1 to 2 in both horizontal and vertical modes, enabling instant page displays.
4. **On-Demand PDF Instant Page Jumping**:
   - Real `declaredPageCount` is now extracted upfront so readers can jump directly to any page (e.g. Page 100 of 200). Target pages are extracted on-demand without waiting for prior pages.
5. **Decoupled Concurrency Locks**:
   - Separated extraction and page discovery mutexes so user-visible foreground pages take immediate priority over background indexing.
6. **Crash Protection & Error Resilience**:
   - Eliminated hard assertions (`check` / `checkNotNull`) that caused crashes on rapid scrolling or reopening saved reading progress. Wrapped page loaders in `supervisorScope` with boundary safety.
7. **Smart Bidirectional Pre-warming & Seamless Jump Look-behind**:
   - Upgraded `ReaderDemandPlanner` to dynamically allocate look-behind windows, breaking the old unidirectional forward-only limitation.
   - Strictly preserves and respects user preferences for `preloadImage` and `readerDecodeAhead`, intelligently allocating backward reserve pages within the user's exact budget.
   - Jump detection ensures both forward and backward neighbors (e.g. 78, 79 and 81, 82 when jumping to 80) are immediately pre-warmed so readers can flip backward seamlessly.
8. **Lightweight PDF Discovery & Network I/O Reduction**:
   - Eliminated redundant 16KB stream probes during sequential index walks over SMB, slashing network RTT round-trips by >70% and drastically speeding up large page jumps.
9. **Disk Cache Fast Resume & Network Exponential Backoff**:
   - Instant 0-latency resume on cold restarts by querying local `DocumentExtractCache` files directly before network handshake.
   - Added automatic exponential backoff retries (up to 3 attempts) for transient SMB/network jitter, preventing premature failure errors.
10. **Enhanced PDF `/Indexed` Color Space & Format Compatibility (Fixed Black Screen / Load Failures)**:
   - Resolved a critical bug in the upstream reader where PDF manga quantized with 8-bit/4-bit indexed palettes (`/Indexed` ColorSpace with FlateDecode, commonly generated by FreePic2Pdf / ComicEnhancerPro) rendered as completely black screens on page 2 and beyond.
   - Implemented full palette reconstruction supporting RGB, Grayscale, and CMYK base color spaces from both indirect stream objects and inline strings.
   - Added robust tolerance for PNG Predictors (Predictor 10–15) and non-standard nested `/Resources` dictionaries, ensuring seamless compatibility with scan/comic PDF documents that failed to load on the original version.

<div align="center">
  <h3>
    <a href="#description">
    Description
    </a>
    <span> | </span>
    <a href="#download">
    Download
    </a>
    <span> | </span>
    <a href="#screenshot">
    Screenshot
    </a>
    <span> | </span>
    <a href="#thanks">
    Thanks
    </a>
    <span> | </span>
    <a href="#license">
    License
    </a>
  </h3>
</div>

# Description

A High performance Android SMB/WebDAV image viewer/comic reader with network gallery folder support.

Based on [EhViewer](https://github.com/FooIbar/EhViewer)   
With [Material Design 3](https://m3.material.io/)  
and [Dynamic Color](https://m3.material.io/styles/color/dynamic-color/overview) Support.  


Similar to Perfect Viewer and Kuro Reader but with Hi-Res images support (no downscaling), clean UI and way better performance.

Build with Grok 4.5.

## Features
* Webtoon gallery reader.
* Native Android app (Kotlin + Jetpack Compose)
* Double tap to go to next folder.
* Material Design 3 Navigation bar.
* Optimized navigation flow for deep folder path.
* ZIP/RAR/CBZ/CBR/CBT/PDF/EPUB support over network share.
* JXL/JXR/JPG/AVIF/HEIC HDR support.
* Compatible with Oppo/OnePlus ProXDR HEIC format.
* SMB signing JCE AESCMAC hardware acceleration support.
* Optimized Async TCP connection poll.
* Full feature network folder listing mode (Videos/Photos/Files listing).
* Network folder playback for MPV/MX Player/VLC with subtitles and external autio track. 
* Wide Color Gamut and 10-bit color mode support.
* Fast smbj client with concurrent connections support.
* Ktor OkHttp WebDAV client with HTTP/2 (default) and CIO HTTP/1.1 fallback.
* Network gallery folders recognition with fast cover loading.
* High performance reader with network cache from EhViewer.
* Reader allow full size image decode.
* Reader auto rotate image.
* E-Ink mode support (ported from [venera-next](https://github.com/cyrilpeng/venera-next)).
* EasyTier support (ported from [moonlight-vplus](https://github.com/qiin2333/moonlight-vplus)).

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

# Download

| Flavor      | Minimum Android Version | Notes                          |
|-------------|-------------------------|--------------------------------|
| Default     | 12                      | Full support                   |
| EasyTier    | 12 (arm64-v8a)          | Full support                   |
| HDR         | 14 (arm64-v8a, x86-64)  | Full support                   |

<a href="https://github.com/zmz125000/LocalViewer/releases">
<img alt="Get it on GitHub" src="https://github.com/zmz125000/LocalViewer-art/blob/master/get-it-on-github.svg" width="200px"/>
</a>

# Screenshot

![screenshots-01](https://github.com/zmz125000/LocalViewer-art/blob/master/screenshots-01.webp)
![screenshots-02](https://github.com/zmz125000/LocalViewer-art/blob/master/screenshots-02.webp)

# Thanks

Here is the libraries

- [Arrow](https://arrow-kt.io/)
- [AOSP & AndroidX](https://source.android.com/)
- [Kotlin & KotlinX](https://kotlinlang.org/)
- [Material Icons](https://github.com/google/material-design-icons)
- [Ktor](https://ktor.io/)
- [Coil](https://coil-kt.github.io/coil/)
- [Compose Destinations](https://composedestinations.rafaelcosta.xyz/)
- [libarchive](https://www.libarchive.org/)
- [libultrahdr](https://github.com/google/libultrahdr)

# License

    Copyright 2014-2019 Hippo Seven
    Copyright 2020-2022 NekoInverter
    Copyright 2022-2023 Tarsin Norbin
    Copyright 2023-2024 Foolbar

    LocalViewer is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

    LocalViewer is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

    You should have received a copy of the GNU General Public License along with EhViewer. If not, see <https://www.gnu.org/licenses/>.

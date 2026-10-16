# 下载中心第三方来源

| 内容 | 固定版本 / 来源 | 许可证与源代码 |
| --- | --- | --- |
| Android Python / yt-dlp 封装与 FFmpeg AAR | Maven Central `io.github.junkfood02.youtubedl-android:library:0.18.1`、`ffmpeg:0.18.1` | [项目源代码与 GPL-3.0](https://github.com/yausername/youtubedl-android/tree/0.18.1)；[Python 构建说明](https://github.com/yausername/youtubedl-android/blob/0.18.1/BUILD_PYTHON.md)；[FFmpeg 构建说明](https://github.com/yausername/youtubedl-android/blob/0.18.1/BUILD_FFMPEG.md) |
| yt-dlp 签名组件资源 | [官方 2026.08.19 发布](https://github.com/yt-dlp/yt-dlp/releases/tag/2026.08.19)，文件名 `yt-dlp` | [源代码及 Unlicense](https://github.com/yt-dlp/yt-dlp/tree/2026.08.19)，分发包内依赖另见 THIRD_PARTY_LICENSES |

yt-dlp 下载后核对官方 SHA2-256SUMS：`1fa6733c37ea6fb51c99ad8fe785e7b7e5f3246c9b980230329d4fb72ed8d4d6`，3,072,469 字节。本项目不运行绕过组件签名的解析器在线更新。版本与来源同时记录在 `modules/download/src/main/assets/parser/version.json`。

许可证原文随组件分发于 `assets/licenses/`。Android 封装、Python、FFmpeg 和各自依赖保留上游许可；此记录不是对整项目重新授权。工程中的修改源码、Gradle 版本锁定和打包脚本随本地交付工作区保留；公开再分发时应一并提供适用许可证要求的对应源码与构建材料，不上传私钥或私有配置。

设备验证素材为 Internet Archive 的 [Big Buck Bunny 页面](https://archive.org/details/BigBuckBunny_328)，仅在测试设备临时目录下载，未放入主 APK 或下载组件。

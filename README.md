# 工具匣按需组件

此仓库提供工具匣 Android 的组件文件（组件要求宿主 versionCode 12 及以上）。建议安装 **0.9.1 / versionCode 13**，修复 0.9.0 同步时“组件发布签名不匹配”的二进制传输问题，原有令牌可继续使用。主应用只安装一次，CLI、离线 ASR、视频剪辑、ComfyUI、文本统计和颜色转换均按需下载。

每个组件位于 `components/<组件ID>/<versionCode>/`，包含 `manifest.json`、`manifest.sig`、`classes.dex` 及必要资源。根目录 `catalog.json` / `catalog.sig` 列出可用版本。应用读取固定提交中的所选组件文件，不下载整个仓库，也不安装组件 APK。

| 组件 | 版本 | 目录 |
| --- | --- | --- |
| CLI | 2.1.0 | `components/cli/3` |
| 离线 ASR | 2.1.0 | `components/asr/3` |
| 视频剪辑 | 2.2.0 | `components/video/4` |
| ComfyUI | 1.0.0 | `components/comfy/2` |
| 文本统计 | 1.0.0 | `components/text/2` |
| 颜色转换 | 1.0.0 | `components/color/2` |

ASR 使用 `sherpa-onnx-paraformer-zh-2023-09-14` INT8 ONNX 中英模型，识别模型和词表从 [Hugging Face 固定版本](https://huggingface.co/csukuangfj/sherpa-onnx-paraformer-zh-2023-09-14/tree/def027084691107096b5ebba69785756d63de6c5) 下载，Silero VAD 从 [Sherpa 官方发布](https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/silero_vad.onnx) 下载。模型约 232.8 MiB，来源与 SHA-256 写入 ASR 签名清单。下载完成后离线识别，权重不存于此仓库。

在应用“全部模块 → 组件管理 → 配置仓库”填写 `guanhelujue/tools`。私有仓库需要手机保存对此仓库具有 Contents Read-only 权限的 fine-grained personal access token，公开仓库无需令牌。令牌不写入仓库，模型网站也不接收 GitHub 令牌。

所有发布清单均带数字签名，客户端验证签名、文件大小、SHA-256 和宿主兼容性后启用。组件不是隔离的第三方应用，与宿主共享权限；只使用既定发布者签名。宿主的原生引擎与共享运行库仍需随主 APK 更新。

组件授权与第三方声明位于各组件的 `assets/`。此目录为运行产物，不含主 APK、模型权重、签名私钥或用户凭据。模型文件与运行库授权详情见 ASR 资源说明。

0.9.1 先通过 Contents API 解析文件的 Git Blob SHA，再通过 Git Blob API 的 Base64 内容还原原始字节并校验，避免签名和 DEX 被文本转换。组件文件未改变，无须重新发布或重新签名。


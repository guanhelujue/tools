# 工具匣按需组件

此仓库提供工具匣 Android 的组件文件，各组件最低宿主版本见签名清单。下载中心需要 **0.10.2 / versionCode 18 或更新兼容宿主**，配套验证主 APK 为 **0.11.0 / versionCode 19**。历史 0.9.1 / versionCode 13 修复 0.9.0 同步时“组件发布签名不匹配”的二进制传输问题，原有令牌可继续使用。主应用只安装一次，CLI、离线 ASR、视频剪辑、ComfyUI、文本统计、颜色转换和下载中心均按需下载。

每个组件位于 `components/<组件ID>/<versionCode>/`，包含 `manifest.json`、`manifest.sig`、`classes.dex` 及必要资源。根目录 `catalog.json` / `catalog.sig` 列出可用版本。应用读取固定提交中的所选组件文件，不下载整个仓库，也不安装组件 APK。

| 组件 | 版本 | 目录 |
| --- | --- | --- |
| CLI | 3.0.0 | `components/cli/4` |
| 离线 ASR | 2.1.0 | `components/asr/3` |
| 视频剪辑 | 3.0.0 | `components/video/5` |
| ComfyUI | 1.2.0 | `components/comfy/5` |
| 文本统计 | 1.0.0 | `components/text/2` |
| 颜色转换 | 1.0.0 | `components/color/2` |
| 下载中心 | 1.0.0 | `components/download/2` |

视频剪辑 3.0.0 重构剪辑、导出与进度界面，增加分割和撤销 / 重做。无损直拼、H.264、H.265 先比较预估大小，确认后才合成；超出预估保留成片。支持切屏与锁屏合成，异常中断保留任务，恢复时从头重新合成。详见 [更新说明](docs/video-3.0.0.md)。

EasyCLIProxy 3.0.0 精简为“登录、连接、日志”三个页面：登录页管理账号池与授权；连接页控制代理服务、复制地址和密钥；日志页只展示模型请求明细与成功、失败或取消结果。采用浅灰绿与白色的原生界面，移除总览、额度、模型目录与统计图表；保留账号和连接配置，迁移最近 2,000 条请求记录。同步仓库后更新 CLI 组件即可。已通过编译、Lint 与 3 项模拟器集成测试；真实账号授权及模型请求尚未验证。

ComfyUI 1.2.0 将底部合并为“创作”和“任务”：在创作页的工作流选择面板中导入、选择和管理工作流；任务卡片同时显示进度、作品预览和保存入口。点击任务右上角“…”可删除任务，电脑服务需更新至 0.5.0。删除会移除工具匣任务及回传作品；手机已保存文件和 ComfyUI 原始输出保留，已开始的显卡计算不会被强制中断。另区分“电脑离线”和“ComfyUI 未启动”。沿用暖白与墨绿风格，保留登录修复和节点编辑；主应用仍为 0.9.1，同步仓库后仅更新 ComfyUI。

ASR 使用 `sherpa-onnx-paraformer-zh-2023-09-14` INT8 ONNX 中英模型，识别模型和词表从 [Hugging Face 固定版本](https://huggingface.co/csukuangfj/sherpa-onnx-paraformer-zh-2023-09-14/tree/def027084691107096b5ebba69785756d63de6c5) 下载，Silero VAD 从 [Sherpa 官方发布](https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/silero_vad.onnx) 下载。模型约 232.8 MiB，来源与 SHA-256 写入 ASR 签名清单。下载完成后离线识别，权重不存于此仓库。

在应用“全部模块 → 组件管理 → 配置仓库”填写 `guanhelujue/tools`。私有仓库需要手机保存对此仓库具有 Contents Read-only 权限的 fine-grained personal access token，公开仓库无需令牌。令牌不写入仓库，模型网站也不接收 GitHub 令牌。

所有发布清单均带数字签名，客户端验证签名、文件大小、SHA-256 和宿主兼容性后启用。组件不是隔离的第三方应用，与宿主共享权限；只使用既定发布者签名。宿主的原生引擎与共享运行库仍需随主 APK 更新。

组件授权与第三方声明位于各组件的 `assets/`。此目录为运行产物，不含主 APK、模型权重、签名私钥或用户凭据。模型文件与运行库授权详情见 ASR 资源说明。

0.9.1 先通过 Contents API 解析文件的 Git Blob SHA，再通过 Git Blob API 的 Base64 内容还原原始字节并校验，避免签名和 DEX 被文本转换。组件文件未改变，无须重新发布或重新签名。

## 下载中心 1.0.0

下载中心需要带下载系统入口和媒体运行库的工具匣 **0.10.2 / versionCode 18 或更新兼容宿主**；本次验证的配套主 APK 为 **0.11.0 / versionCode 19**，SHA-256 为 `2ebf806f1edf2f4e9b39f8f573fac15a3b258b2ee3c7a3f13c153c20a9670f76`。旧 0.9.1 主程序不能仅靠更新组件启用下载中心。主 APK 沿用独立安装包交付，不存入本组件仓库。

安装兼容主程序后，在组件管理同步仓库，选择「下载中心」安装。支持 HTTP/HTTPS、多连接、暂停续传、后台队列、SAF 目录和完成文件分享。网页媒体仅实际验证 Internet Archive 的 Big Buck Bunny 页面；其他网站未验证，当前不接入 HLS/DASH、直播、登录、DRM、BT、磁力或远程下载。

发布的 DEX、解析器与资源均取自同一份已验证交付，保留现有其他组件版本。详见 [使用和兼容说明](docs/DOWNLOAD_CENTER.md)、[验证结果](docs/DOWNLOAD_VALIDATION.md)和[第三方来源](docs/DOWNLOAD_THIRD_PARTY.md)。

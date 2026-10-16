# 下载中心 1.0.0

这是工具匣的按需 DEX 组件，需要工具匣 **0.10.2（版本码 18）或更新兼容宿主**，moduleApi 2。不是独立 App。构建使用 compileSdk 36、minSdk 29、targetSdk 35，包含 arm64-v8a / x86_64 共享原生运行库。

## 安装与使用

1. 同签名覆盖安装交付目录中的 `toolcase-0.11.0.apk`，保留原有应用数据。本次交付同步了其他 agent 的宿主版本推进，不回退已有宿主改动。
2. 在工具匣「组件管理」同步 `guanhelujue/tools` 仓库，选择「下载中心」安装。也可导入本地交付的 `offline-components/download.tcm`。
3. 打开「下载中心」，点击「新建下载」。粘贴一个或多个 HTTP/HTTPS 链接，点击「查看文件信息」，确认文件名、大小、类型和保存目录，再开始。也可以从浏览器的分享菜单选择「工具匣 · 下载中心」。剪贴板仅在点击粘贴时读取。
4. 普通文件使用默认「下载 / Toolcase」目录，或通过系统文件选择器授权目录。网页使用「网页视频 / 音频」模式，解析后选择格式。文件完成并导出校验成功后才允许打开、分享和模块导入。

离线导入组件仍验证宿主签名、组件签名、身份、版本、大小及 SHA-256。下载中心已按现有 GitHub 发布流程加入组件仓库，可以按需在线安装。其他组件的远端版本保持原样。不要把 `.tcm` 改名为 APK，也不要单独安装测试 APK。

## 使用范围

- 文件：HTTP/HTTPS、重定向、批量最多 100 个链接、RFC 5987 文件名、可编辑文件名/目录、可选请求头和可信 SHA-256。
- 任务：优先级、最多 3 个同时下载任务（默认 2）、批量暂停/继续/取消、搜索和状态筛选。删除记录与删除导出文件分开确认。
- 连接：自动最多 2 个；可选 1/2/4/8 个。全局上限 8、单主机上限 4；小文件、未知长度或无法确认资源版本时使用单连接。设置 8 不表示单个主机一定使用 8 个，也不承诺加速倍数。
- 文件冲突：自动改名、跳过、明确确认后覆盖。新文件先校验并发布，再删除旧文件；提供程序不支持最终改名或替换时保留完整文件并给出提示。
- 网页：使用 yt-dlp **2026.08.19**，当前接入 HTTP/HTTPS 点播格式。显示标题、封面、清晰度、格式、已知大小和仅音频选项。网站无独立音频时，提供下载后无损提取到 MKA 的选项；最终大小未知会明确显示。
- 分离的音视频使用 FFmpeg 无损封装为 MKV，显示「合并中」；发布完成前不进入「已完成」。

**实际验证的网站仅 Internet Archive（archive.org）的 Big Buck Bunny 页面。** 解析器可能支持其他公开页面，但本版本没有验证 YouTube、Bilibili 等网站，不承诺可用。暂不接入 HLS/DASH 分片协议、直播、浏览器登录状态、DRM、BT、磁力、远程电脑下载和自动拦截浏览器。需登录、受 DRM 保护、格式不支持、403/404 及网络故障分别提示。

## 可靠性与数据

`TransferEngine.java` 是 Android 与 JVM 验证共用的真实传输实现。先请求 `Range: bytes=0-0`，核对实际 206、Content-Range 和响应体；不会仅根据 Accept-Ranges 决定并发。

可续传资源必须有已知长度及有效的强 ETag 或 Last-Modified。分段请求带 If-Range，并检查每次 206 的范围、总长度和资源标识。每个分段先 fsync，再原子写入检查点；重启时截断未提交尾部，只请求缺失范围。任务文件锁与队列锁阻止同一任务重复执行。

Range 被忽略时，首次传输可安全清空临时分段后改为单连接；续传期间返回 200、416 或标识变化会要求明确重新下载。更新地址不能跳过一致性检查；跨路径/主机变更还要求可信 SHA-256 或重新下载。相同长度不是资源相同的证明。

失败最多重试 3 次，采用退避并尊重 Retry-After；超过 120 秒的服务端等待要求会停止本轮并提示稍后重试。未知大小不显示虚假百分比。完成时检查长度；填写可信散列时核对 SHA-256；每次导出均读回目标文件，验证长度和 SHA-256 与已下载文件一致。没有发布者散列时，读回校验验证的是本地保存一致性，不证明远端文件来源可信。

数据库、私有临时分段与用户导出文件分别管理。默认目录使用 MediaStore 的 IS_PENDING，SAF 使用 `.partial` 文档并在校验后改名。未完成文件不会交给其他模块。取消清除私有分段和能够访问的临时输出；授权被撤销导致无法清理时明确提示重新授权后清理。

临时区需要分段和组装结果两份空间，导出又需要目标目录空间。大型媒体还需保留原始流和合并输出，请预留余量；空间不足会停止并提示。卸载下载组件删除私有任务、分段和组件文件，保留已经导出的用户文件。

## Android 后台机制

- Android 14+：从可见页面提交用户发起数据传输 Job（UIDT），设置网络条件、持久化和任务通知，处理 JobService 停止原因。网络中止进入「等待网络」；系统中止与用户暂停分开记录。通知继续动作先打开可见页面，再调度任务。
- Android 10–13：使用声明在宿主中的 dataSync 前台服务和有上限的唤醒锁。服务不会在用户强停后擅自启动。
- Android 15+：优先仍为 UIDT；兼容服务实现 `onTimeout(startId, fgsType)`，停止传输、保留检查点并及时停止服务，不循环重启规避超时。API 29–33、35+ 和厂商后台策略尚未做设备实测。
- 用户暂停不会被系统恢复逻辑重新排队。手机重启保留数据库和有效分段，UIDT 持久任务由系统决定何时重调度；重新打开也可手动继续。强行停止 App 后不承诺继续运行。

依据：[UIDT 官方说明](https://developer.android.com/develop/background-work/background-tasks/uidt)、[前台服务超时](https://developer.android.com/develop/background-work/services/fgs/timeout)。宿主的后台保护不能替代下载任务的系统调度。

## 模块职责与选择依据

| 代码 | 职责 |
| --- | --- |
| `DownloadStore.kt` | SQLite 任务、状态、目标 URI；偏好设置 |
| `core/TransferEngine.java` | HTTP 协议、范围调度、验证、持久检查点和有限重试 |
| `DownloadQueue.kt` | 队列、网络条件、Job/Service、通知操作、进程停止协调 |
| `DownloadOutput.kt` | MediaStore / SAF 导出、冲突处理与读回校验 |
| `media/MediaResolver.kt` | 网页解析、格式选择、真实流下载和 FFmpeg 合并/提取 |
| `DownloadActivity.kt` / `DownloadForms.kt` | 手机 Compose 页面、分享接入、用户操作 |

评估过 [OkDownload](https://github.com/lingochamp/okdownload) 和系统 DownloadManager。二者不能直接提供本项目要求的可验证分段检查点、严格资源一致性策略、动态组件生命周期及可控的 UIDT 调度。通用引擎因此采用规模受控的独立实现，用同一份源码执行真实 HTTP 协议测试，而不包装模拟进度。

网页解析复用维护中的 [youtubedl-android 0.18.1](https://github.com/yausername/youtubedl-android)，使用其 Python/FFmpeg Android 运行库，避免自己维护网站解析规则。未加入其 aria2 下载器；所有可下载流都经过同一个可靠性引擎。解析器字节作为签名组件资源安装，可单独发新版组件更新；原生运行库或系统入口变化仍需升级宿主。支持 Android 系统配置的 HTTP 代理，不另建代理账户配置。

请求头只保存于应用私有数据，不写普通日志。跨源重定向不转发自定义请求头，HTTPS 到 HTTP 的重定向拒绝自动降级。宿主网络配置允许用户主动创建的 HTTP 下载；原有凭据接口的 HTTPS 校验保留。

## ASR / 剪辑 URI 协作接口

仅对 `COMPLETE` 的 content URI 发出明确 Intent：`ACTION_SEND`、正确 MIME、`EXTRA_STREAM`、`ClipData`、`FLAG_GRANT_READ_URI_PERMISSION`，目标为现有 `ComponentRuntime.entry(id)`。接收方需要在已签名组件 metadata 声明 `uriImportVersion: 1`，并负责将 URI 复制到自己的任务区或依法保存授权。

当前 ASR / 视频组件尚未声明该接口，下载中心会说明「打开模块后通过文件选择器导入」，并提供打开模块；未安装时引导组件管理。没有直接改写其他 agent 的模块内部逻辑，也不会无响应地发送不受支持的 Intent。

## 构建与交付

使用 JDK 17 和现有 Android SDK 36。为避开同期 agent 的构建目录，本模块使用隔离缓存与输出：

```powershell
.\gradlew.bat -I scripts/download-isolated.gradle --project-cache-dir .gradle-download '-PtoolcaseComponents=cli,asr,video,comfy,download' :app:assembleDebug :app:prepareComponents :app:testDebugUnitTest :modules:download-tests:test --console=plain --max-workers=2
.\scripts\package-download.ps1
```

产物位于 `dist/download-center-1.0.0/`，包括主 APK、下载中心 `.tcm`、所选五个组件的签名仓库树、说明和 SHA256SUMS。默认工程构建仍包含其他 agent 新接入的组件；`toolcaseComponents` 只限定本次交付集合。打包要求使用新目录，避免旧文件混入。`-UseVerifiedBuild` 可以封存已完成设备验证的隔离构建，避免并行开发的后续源码变更替换已验证产物。主 APK 使用现有本机调试签名；不要上传签名私钥。

第三方声明及许可证见组件 `assets/licenses/` 和 [DOWNLOAD_THIRD_PARTY.md](DOWNLOAD_THIRD_PARTY.md)。验证证据见 [DOWNLOAD_VALIDATION.md](DOWNLOAD_VALIDATION.md)。

# 下载中心验证记录

验证日期：2026-10-17（Asia/Shanghai）。本文件区分已执行的验证与仍需真实设备覆盖的场景。

## HTTP 协议与大文件

同一份 `TransferEngine.java` 同时进入 Android 组件和 JVM 测试。可控 HTTP 服务实际收发数据，验证生成文件 SHA-256；没有通过模拟进度代替网络。

16 项常规测试全部通过，另有 1 项实际超过 4 GiB 的完整传输通过：

| 场景 | 验证结果 |
| --- | --- |
| 正常 206 多连接 | 实际请求重叠，并发不超过单主机 4，结果散列一致 |
| 用户暂停 / 新引擎实例恢复 | 持久检查点有效，只请求缺失数据；总补传小于完整文件 |
| 忽略 Range 返回 200 | 安全单连接；不会把多个 200 正文拼接 |
| 先探测 206，实际分段忽略 Range | 清理首次分段并退回单连接，最终散列一致 |
| 探测拒绝 Range / 416 | 非空资源尝试普通 GET；真正传输返回 416 时要求重新下载 |
| Content-Range 错误 | 拒绝发布，不把损坏分段标成完成 |
| 连接中断 | 有限退避续传，最终散列一致 |
| ETag 变化 | 禁止混合不同版本，要求明确重下 |
| 未知长度 | 单连接真实 EOF，保持未知总量，无假百分比 |
| 重定向 / Content-Disposition | 正确还原 UTF-8、加号与文件名 |
| 429 / Retry-After | 实际等待至少指定时间，随后成功 |
| 403 | 不无限重试 |
| 错误可信散列 | 不导出为已完成 |
| 5 GiB 范围计算 | Long 算术完整覆盖，不发生 32 位溢出 |
| 自定义头 CRLF / 保留字段 | 拒绝不安全或引擎管理的字段 |
| 4 GiB 以上实际文件 | **4,295,032,853 字节**完整网络传输、组装和 SHA-256 校验，99.249 秒 |

大文件散列：`1916a2bfea313a8c87be5082ce759931cca1bea21a6026fcc3e6e55d800bd720`。

原始 JUnit 证据：[常规协议](download-evidence/protocol-tests.xml)、[大文件](download-evidence/large-transfer.xml)。默认构建跳过耗时大文件用例；先前执行的完整结果已单独保存，避免被默认测试覆盖。宿主单元测试 5 项通过。

```powershell
.\gradlew.bat -I scripts/download-isolated.gradle --project-cache-dir .gradle-download :modules:download-tests:test --rerun-tasks --console=plain
.\gradlew.bat -I scripts/download-isolated.gradle --project-cache-dir .gradle-download '-Ddownload.large=true' :modules:download-tests:test --rerun-tasks --console=plain
```

大文件用例需要约 9 GiB 可用空间；执行后临时测试目录由测试框架清理。

## Android 验收

专用 Android 14 / API 34 x86_64 模拟器，加载实际宿主与通过签名验证的下载中心 DEX。早期验收使用独立端口 5562；之后该实例被关闭、端口被其他 agent 复用，最终验收改用新的隔离实例 5564，没有操作其他 agent 的测试实例。

最终结果与每项证据保存在 [device-acceptance.txt](download-evidence/device-acceptance.txt) 和 [instrumentation.txt](download-evidence/instrumentation.txt)。可控 Android 文件大小 8 MiB，导出 SHA-256：`7d212b9c884f5c77896de960ae17cc341cda43b14d6a971f34ca29ebd4badf7f`。

验收覆盖真实 UIDT 后台传输、退到 HOME 后字节增加、通知暂停/继续、用户暂停意图、组件进程终止后恢复、Wi-Fi 关闭时停止并等待网络、Wi-Fi 恢复后续传、MediaStore 导出、可持久 SAF 授权、同名改名和显式覆盖、撤销目录权限、取消清理，以及组件卸载后导出文件的散列保持不变。

通知继续通过真实 PendingIntent 路径执行；测试进程赋予与用户点通知相符的后台 Activity 启动权限。普通后台代码直接发送 Activity PendingIntent 在 Android 14 上会被限制，不用该限制误判用户点击通知失效。

媒体使用真实 [Internet Archive Big Buck Bunny 页面](https://archive.org/details/BigBuckBunny_328)：解析标题及格式，实际下载 **43,315,070 字节** MP4，并调用生产 FFmpeg 合并函数。合并测试以同一真实 MP4 作为两个输入，分别选择其视频与音频轨，输出 MKV 后由 Android MediaMetadataRetriever 确认两种轨道。另执行无损仅音频提取，确认没有视频轨。**该测试不等价于已验证某网站的远程分离音视频格式**；分离格式调度已有实现，但没有宣称所有网站通过。

测试机访问 Internet Archive 使用电脑已有的系统 HTTP 代理。解析器接入 Android ProxySelector；测试只对专用模拟器转发本机代理端口。网络或网站规则变化可能使此在线用例失败，协议用例不依赖第三方网站。

## 复现设备测试

```powershell
# 先选择自己独占的设备序列号，不要复用其他 agent 的模拟器。
python scripts/download-test-server.py --port 19463
adb -s <serial> reverse tcp:19463 tcp:19463
.\gradlew.bat -I scripts/download-isolated.gradle --project-cache-dir .gradle-download '-PtoolcaseComponents=cli,asr,video,comfy,download' :app:assembleDebug :app:assembleDebugAndroidTest --console=plain
adb -s <serial> install -r .download-build/app/outputs/apk/debug/app-debug.apk
adb -s <serial> install -r -t .download-build/app/outputs/apk/androidTest/debug/app-debug-androidTest.apk
adb -s <serial> shell am instrument -w -e class app.toolcase.DownloadAcceptanceTest app.toolcase.test/androidx.test.runner.AndroidJUnitRunner
```

重启验证分两阶段：`DownloadRebootTest#prepare` 下载并暂停，实际重启测试设备并重新设置 adb reverse，再运行 `DownloadRebootTest#verify`。测试检查系统 boot_count 确实增加、用户暂停状态未被自动恢复，并在明确继续后验证导出散列。设备报告见 [reboot.txt](download-evidence/reboot.txt)。

## 手机界面检查

已在实际组件页面检查浅色竖屏、已完成卡片及深色 1.5 倍字体；长文件名保持两行边界，主要操作按钮可见。截图：[浅色](download-evidence/ui/portrait.png)、[已完成](download-evidence/ui/completed.png)、[深色大字体](download-evidence/ui/dark-large.png)。主列表和表单采用可滚动布局，横向筛选可以滚动；横屏专项截图的模拟器显示调整命令被执行策略拦截，因此未将横屏计为已实测。

## 限制与未覆盖项

- 设备验证为 API 34 x86_64 模拟器；Android 10–13、15+、arm64 真机、16 KiB 页大小设备及厂商省电策略尚未实测，不能据此承诺所有机型可靠后台运行。
- Android 15+ 兼容服务超时与系统停止回调有实现；没有使用 API 35 真机触发六小时 dataSync 限额。
- 可信散列由用户提供；无散列时验证长度、服务器版本标识和本地导出一致性。
- SAF 云目录实现差异很大。实际验证使用具有真实文件读写、改名、删除及授权撤销行为的 DocumentsProvider；不代表所有云盘均支持。
- 网页仅验证 archive.org 指定公开页面。HLS/DASH 分片、直播、登录、DRM、所有网站支持均不在本版承诺范围。
- ASR/视频当前版本未声明 URI 导入接口，因此提示用户进入模块手动选择已下载文件；不声称自动导入已验证。
- 同期宿主 AI/相册开发继续进行；本次签名仓库仅包含 CLI、ASR、视频、ComfyUI、下载中心，下载交付不打包尚在开发的 AI 相册组件。

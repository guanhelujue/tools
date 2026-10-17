# 相册助手 1.2.0 验证记录

日期：2026-10-17。宿主 0.12.2（22），album 1.2.0（6），独立 Android 14 模拟器 Toolcase_Album_20261017 / emulator-5576。

- 生产 APK、动态相册 TCM 构建通过，APK 签名有效。包内相册版本 6、最低宿主 22，TCM 与在线发布目录逐文件相等；生产 APK 不含测试 Service 或测试用例。
- 15 项 JVM 测试通过：12 项中文查询、时间/GPS、排除条件、错误匹配、结构校验、重复/相似测试，3 项稀疏向量稳定性、归一化、同义词召回与元数据退路测试。
- 16 项 Android 测试通过：4 项 SQLite 检索/迁移测试、11 项原图片与存储测试、1 项覆盖完整悬浮交互的端到端用例。最终日志为 verification/release-checks.log，OK (16 tests)。
- SQLite 测试涵盖 v2 → v3 保留描述与虚拟相册、进程重开后补建本地索引、内容变化/模型变化/删除与范围收缩清除索引；多条中文组合查询与全扫描结果一致，不填充错误结果。
- 1,004 条合成描述中，“猫”查询从倒排表召回 3 个候选，排除背景描述；本次模拟器查询约 4.6 ms（单次观测，不是所有设备或大图库的延迟保证）。没有图片解码、联网或模型请求。
- 既有 11 项图片/存储测试覆盖缓存复用、增量更新、图片删除、选定范围、取消与进程重开、模型失败、重试、图片方向与缩略图等。
- UI 用真实签名组件和四份测试猫照片，描述由测试夹具预置；验证不开启无障碍也可启用悬浮球、桌面显示、拖动吸边、半屏归档和中文搜索、保存智能相册、键盘、真正的横屏窗口、大字体与深色、悬浮权限撤回，以及相册组件进程被结束后重新显示悬浮球。
- 初次自动化中调整了候选数过严断言、界面重绘等待和 MediaStore 异步导入等待；最终完整套件已通过。系统设置会隐藏第三方悬浮窗，跨应用显示改在桌面验证。
- 界面截图均为测试环境。横屏背景为 Gallery TEST fixture（非小米相册），没有将测试 APK 安装到真实手机，没有读取或上传用户照片。

真实模型、语义 Embeddings 和 HyperOS 真机兼容性不在本次已通过范围。统一 AI 网关的旧/新版 CLI 连接另由主应用任务完成模拟上游验证；本轮保留其公共接口变更。稀疏词项向量可加速召回，但依赖描述、同义词和确定过滤规则，不等价于已验收的多模态语义理解。

复现：
`gradlew :app:assembleDebug :albumFloatingValidation:assembleDebug :modules:album:testDebugUnitTest -I scripts/album-floating-build.init.gradle -I scripts/album-floating-tests.init.gradle -PtoolcaseComponents=album`
随后仅在一次性模拟器安装生产宿主、测试 APK 和 Gallery TEST fixture，运行 AlbumRetrievalTest、AlbumStorageTest、AlbumFloatingUiTest，并带 `-e albumFloatingFixture true`。

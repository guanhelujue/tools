# 视频预览 3.0.1 验证

2026-10-17。视频组件版本码 6，沿用宿主共享 Media3 1.11.1；测试宿主为工具匣 0.10.1 / 17，Android 14 x86_64 模拟器。

## 可复现原因

3 秒、30 fps、90 帧的 H.264 片段每半秒换一种颜色，但整个片段只有开头一个关键帧。旧时间轴固定抽取 6 张 `OPTION_CLOSEST_SYNC` 图片，结果均为红色。真实 UI 测试 `VideoFramePreviewTest.filmstripShowsIntermediateFramesInsideLongGop` 连续两次失败；只将抽帧模式改为 `OPTION_CLOSEST`，相同测试即通过。

随后增加真实触摸拖动测试。未启用播放器拖动优化时，测试期间截图仅观察到红、绿、蓝三种画面，低于要求的四种；只启用 `setScrubbingModeEnabled` 后测试通过。最终实现仅在拖动期间启用，松手或取消后关闭并立即提交最后一次精确定位。保留 `SeekParameters.EXACT`，不以跳到关键帧换取速度。截图采样只能证明拖动过程中画面更新，不是设备解码帧率基准。

## 改动

- 从对应时间抽取真实画面；按 URI 和微秒时间缓存，避免不同缩放级别串用图片。
- 双指或 − / ＋ 缩放时间轴，缩放围绕当前播放位置；按显示宽度增加缩略图，最大密度受素材帧率限制。时间刻度随缩放细化。
- 单个后台任务解码；新视口替换待处理队列，复用当前源的读取器。位图缓存按字节限制为 4 MiB，队列完成或页面隐藏时释放闲置读取器，退出页面时释放全部资源。
- 主时间轴和裁剪滑条都接入拖动模式，松手保留最终精确定位。

## 结果与命令

构建与 9 项 `VideoPlanningTest` 通过。4 项 `VideoFramePreviewTest` 覆盖六种实际缩略图、拖动时画面更新与松手后最后一帧、按钮放大后的拖动精度、从裁剪手柄开始双指缩放时保留片段范围和播放位置。既有 `VideoEditorUiTest` 覆盖跨片段画面、裁剪、分割、撤销重做、导出确认、375 dp 小屏和 1.3 倍字体、667 × 375 dp 横向窗口。

```powershell
$env:JAVA_HOME='C:/Program Files/Eclipse Adoptium/jdk-17.0.20.8-hotspot'
.\gradlew.bat -I scripts/package-video-component.gradle :videoValidation:assembleDebug :modules:video:testDebugUnitTest --offline --console=plain --max-workers=2 '-Dorg.gradle.jvmargs=-Xmx1024m -XX:ActiveProcessorCount=4 -Dfile.encoding=UTF-8'
adb -s <test-device> install -r build/video-editor/harness/outputs/apk/debug/videoValidation-debug.apk
adb -s <test-device> shell am instrument -w -e class 'app.toolcase.VideoFramePreviewTest,app.toolcase.VideoEditorUiTest' app.toolcase.test/androidx.test.runner.AndroidJUnitRunner
```

红测、单变量试验与最终记录分别保存在 `build/video-preview-fix/red-test-2.log`、`probe-test.log`、`drag-baseline.log`、`drag-probe.log` 和 `final-tests.log`。双指测试曾使用跨度过小的注入手势，未达到测试要求的放大比例；改用完整的 40 → 320 dp 手势后单测通过，未降低放大断言。

只在测试设备运行：测试会安装正式签名的视频组件并替换草稿。交付包必须与测试 APK 中的 `assets/video.tcm` 字节一致。

## 边界

用户此前的格式报错已自行恢复，缺少当时素材与错误记录，未断言其原因，也未声称修复了格式兼容性。原片没有重新编码；预览改动不改变三种导出算法。

尚未用用户的原始 1.3 GB 视频、4K / HDR 素材或小米 HyperOS 真机验证流畅度。高分辨率或长 GOP 仍受设备解码能力限制。前后帧按钮继续按素材帧率/平均帧率作时间步进，可变帧率不能保证遍历每一个原始帧。

依据：[Android 抽帧选项](https://developer.android.com/reference/android/media/MediaMetadataRetriever)、[Media3 拖动模式](https://developer.android.com/reference/androidx/media3/exoplayer/ExoPlayer#setScrubbingModeEnabled(boolean))。

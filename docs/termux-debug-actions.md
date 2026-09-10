# Termux 修复版的手动构建

工作流：`.github/workflows/termux-debug-apk.yml`，显示名称为 **Termux fix - Debug APK**。
只配置 `workflow_dispatch`；提交代码、创建 PR 或添加工作流都不会自动运行。

## 手动启动

工作流保存在 `fix/termux-result-newline-20260910` 修复分支，
并已单独添加到默认分支 `master`，作为手动触发入口。仓库的 Actions 已启用。

GitHub 要求默认分支上存在工作流文件，才会接受手动触发事件。
入口注册不要求把修复代码合并到 `master`。
相关说明见 [GitHub 手动运行工作流文档](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/manually-run-a-workflow)。

在 Actions 中打开工作流并点击 **Run workflow**。
`source_ref` 默认是 `fix/termux-result-newline-20260910`，也可以填写需要验证的提交 SHA。
工作流会记录实际检出的提交和子模块版本，便于对应 APK 与源代码。

## 运行内容

1. 递归检出 `material3/material-color-utilities` 和 `llama-cpp/native/llama.cpp` 子模块。
2. 配置 JDK 17、Node.js 24、Bun 1.4.2 和 pnpm 12.3.4。
3. 安装 Android API 37 的 `platforms;android-37.0` 包、Build Tools 36.0.0、NDK 28.2.13676358 和 CMake 3.22.1。
4. 使用仓库自带的 Gradle 9.5.0 Wrapper，先执行 `:app:testDebugUnitTest`，只运行 `TermuxToolTest`。
5. 测试通过后执行 `:app:assembleDebug`，检查生成的 APK 签名并计算 SHA-256。

Build Tools 与 NDK 版本对应项目 AGP 9.3.1 的默认工具链，见
[AGP 9.3 兼容性说明](https://developer.android.com/build/releases/agp-9-3-0-release-notes)。
SDK 平台包名包含小版本号 `.0`，与应用的整数 `compileSdk = 37` 对应；
包名来自 [Google SDK 仓库索引](https://dl.google.com/android/repository/repository2-3.xml)。
首次运行因使用旧格式 `platforms;android-37` 而在安装 SDK 时失败，尚未进入代码编译。
Web UI 继续由 `:web:preBuild` 调用 Bun 安装锁定依赖，再通过 pnpm 构建。
工作流不会另外改写依赖版本或锁文件。

## 下载与安装

- `rikkahub-termux-debug-<提交>-<运行编号>-<重试编号>`：APK、`SHA256SUMS` 和 `build-info.txt`。
- `termux-build-diagnostics-<运行编号>-<重试编号>`：构建日志、JUnit XML 和 HTML 测试报告；失败时也尝试上传。
- 产物保留 14 天。项目当前配置生成 arm64-v8a、x86_64 和 universal APK。
- Debug 包名为 `excp.rikkahub.debug`，可与正式版并存；需要在 Debug 应用中重新配置 Termux 权限。
- 构建使用运行器生成的 Debug 签名，不使用上游发布密钥。不同运行的签名可能不同，不能保证覆盖安装前一次 Debug 包。

本工作流不创建 Release，不提交 Issue 或 PR，也不上传到应用商店。
实际依赖解析、完整 Android 编译和真机验证，要等首次运行后再确认。

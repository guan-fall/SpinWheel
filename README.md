# 转盘抽取（CHOISE / SpinWheel）

一个自用的 Android 转盘抽取 App。Kotlin + Jetpack Compose 实现，纯本地运行，无网络权限。

支持：默认模式（严格按权重、固定 7 秒）与仿真模式（力度条物理模拟、指针随手拨动互动）、多指针抽取、多转盘管理（10 个）、抽取历史（500 条）、音效震动、无障碍、浅色/深色主题与色盲友好/高对比度配色、中英双语。

## 下载安装

到 [Releases](../../releases) 页面下载最新的 APK（如 `CHOISE-v1.18.apk`），拷到手机直接安装（需允许安装未知来源应用）。

## 开源

本项目以 [MIT License](LICENSE) 开源，欢迎自由使用、修改与分发。

## 运行环境（本机已配好）

| 组件 | 位置/版本 |
|---|---|
| 项目 | `D:\appdevelopment\projects\SpinWheel` |
| Android Studio | `D:\appdevelopment\AndroidStudio` |
| Android SDK | `D:\appdevelopment\AndroidSdk`（compileSdk 34，minSdk 24，targetSdk 34） |
| JDK | `D:\appdevelopment\jdk-21.0.12.1+1` |
| Gradle | 8.13（`D:\appdevelopment\gradle-8.13`，Wrapper 已指向腾讯云镜像） |
| 依赖镜像 | 阿里云（google/public/gradle-plugin），国内全速下载 |

## 构建 & 安装（命令行，Git Bash）

```bash
cd /d/appdevelopment/projects/SpinWheel
export JAVA_HOME='D:\appdevelopment\jdk-21.0.12.1+1'
export GRADLE_USER_HOME='D:\appdevelopment\gradle-home'   # ASCII 路径，规避中文用户名导致的测试 worker 崩溃

# 构建 APK
/d/appdevelopment/gradle-8.13/bin/gradle.bat assembleDebug

# 安装到已连接的手机（需开启 USB 调试）
/d/appdevelopment/AndroidSdk/platform-tools/adb.exe install -r app/build/outputs/apk/debug/app-debug.apk

# 启动
/d/appdevelopment/AndroidSdk/platform-tools/adb.exe shell am start -n com.example.spinwheel/.MainActivity
```

APK 输出位置：`app/build/outputs/apk/debug/app-debug.apk`（可直接拷到手机安装）。

## Android Studio 打开

`File → Open` 选择 `D:\appdevelopment\projects\SpinWheel` 即可，Gradle Wrapper 与 SDK 路径均已配置好，同步可直接通过。

## 测试

```bash
# 单元测试（旋转引擎/批量解析/校验规则，24 个用例）
/d/appdevelopment/gradle-8.13/bin/gradle.bat testDebugUnitTest

# Compose UI 冒烟测试（需连接手机）
/d/appdevelopment/gradle-8.13/bin/gradle.bat connectedDebugAndroidTest
```

## 项目结构

```
app/src/main/java/com/example/spinwheel/
├── SpinWheelApp / MainActivity        # 入口（竖屏锁定）
├── data/
│   ├── model/     Models(转盘/选项)、Validators、BatchParser（纯 Kotlin 可测试）
│   ├── db/        Room：抽取历史
│   ├── store/     DataStore：设置 + 转盘 JSON（自动保存）
│   └── repo/      转盘仓库 / 历史仓库（全局 500 条自动裁剪）
├── engine/SpinEngine  扇区角度数学、默认模式落点、仿真物理（纯 Kotlin 可测试）
├── audio/SoundManager SoundPool 预加载，滴答/叮声音效为运行时合成 WAV
├── haptic/Haptics     Vibrator/VibratorManager 柔和震动
├── i18n/Strings       中文/English 双语
├── ui/theme/          iOS 风格配色（标准/色盲友好/高对比度）× 浅色/深色
├── ui/components/     自绘控件（按钮/弹窗/开关/滑杆/分段控件，无 Material 默认外观）
├── ui/wheellist/      转盘列表页（空状态加号、拖拽排序、菜单）
├── ui/wheelspin/      转盘抽取页（Canvas 转盘、力度条、结果弹窗、打乱撤销）
├── ui/wheeledit/      转盘编辑页（选项/权重/批量输入/删除）
├── ui/history/        历史记录页（筛选/删除/清空）
└── ui/settings/       设置页（音效/震动/主题/暗黑/语言/大字体等）
```

## 说明

- 依赖下载走阿里云镜像；Gradle Wrapper 下载走腾讯云镜像。
- `GRADLE_USER_HOME` 指到 `D:\appdevelopment\gradle-home` 是为了规避 Windows 中文用户名导致单元测试 worker 无法启动的问题。
- 音效文件（tick.wav / ding.wav）首次启动时由代码合成写入应用私有目录，无需外部素材。
- 每次冷启动（杀后台后再次进入）会自动打乱全部转盘的选项顺序并保存，
  目的是避免每次使用都从相同排列开始；权重、概率与历史不受影响。

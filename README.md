# 🎤 声域 (VocalScope)

一款基于 Android 的实时人声音域测试应用，帮助你快速、准确地检测个人声音的音高范围

## ✨ 功能特性

### 🎯 实时音高检测
- 基于 **TarsosDSP YIN 算法** 的专业级音高分析
- 实时显示当前发声的**音名**（如 C4、F#3）和**频率**（Hz）
- 采样率 44100Hz，缓冲区 2048 样本，确保低延迟高精度

### 🎹 钢琴键盘可视化
- 内置 F2 ~ D5 范围的虚拟钢琴键盘（覆盖绝大多数人声音域）
- 已检测到的**音域范围**以蓝色高亮显示在键盘上
- **实时黄色游标**随发声位置在键盘上移动
- 每个白键底部标注音名，直观易读

### 🛡️ 多重防误判机制
- **RMS 音量阈值过滤**：自动忽略背景噪音和微弱杂音
- **YIN 置信度校验**：仅在置信度 > 85% 时采信检测结果
- **频率范围限制**：将检测范围约束在 65Hz ~ 1100Hz 的人声区间
- **八度跳变防护**：通过中值滤波 + 跳变候选确认机制，防止瞬时误判拉坏音域极值
- **硬件级降噪**：自动启用 Android `NoiseSuppressor`（如设备支持）

### 📱 用户体验
- 采用 **Material Design 3** 设计语言，界面简洁美观
- 录音时**屏幕保持常亮**，避免测试中断
- 自适应弹性布局，适配不同屏幕尺寸
- 一键开始/停止测试，支持重置记录

## 📸 界面预览

![VocalScope 界面预览](assets/preview.png)

|    功能     |           说明            |
|:---------:|:-----------------------:|
|  中央圆形区域   |      实时显示当前检测到的音名       |
|   频率显示    |     当前音高对应的频率值（Hz）      |
| 最低音 / 最高音 |      本次测试中检测到的音域极值      |
|   钢琴键盘    | 蓝色区域为个人音域跨度，黄色圆点为实时音高位置 |

## 🏗️ 技术架构

```
com.ltx.vocalscope
├── MainActivity          # 主活动，权限管理与内容视图
├── VocalScopeScreen()    # 核心 UI 组件（Compose）
├── PianoRangeChart()     # 钢琴键盘可视化组件（Canvas 绘制）
├── startAudioCapture()   # 音频采集与 YIN 音高检测
├── getMidiNoteNumber()   # 频率 → MIDI 编号转换
├── getNoteNameFromMidi() # MIDI 编号 → 音名转换
├── medianInt/Float()     # 中值滤波工具函数
└── ui/theme/             # Material 3 主题配置
```

### 核心技术栈

|           技术            |     用途      |     版本      |
|:-----------------------:|:-----------:|:-----------:|
|       **Kotlin**        |    开发语言     |   2.3.10    |
|   **Jetpack Compose**   |  声明式 UI 框架  | BOM 2026.02 |
|     **Material 3**      |    设计系统     |     最新      |
|      **TarsosDSP**      | YIN 音高检测算法  |     2.5     |
| **Android AudioRecord** | 原始 PCM 音频采集 |      -      |
|  **Kotlin Coroutines**  |   异步音频处理    |      -      |
|   **Compose Canvas**    |  钢琴键盘自定义绘制  |      -      |

### 音高检测流程

```
麦克风原始 PCM 数据
    ↓
AudioRecord (44100Hz, Mono, 16-bit)
    ↓
Short → Float 转换 + RMS 计算
    ↓
TarsosDSP YIN PitchProcessor
    ↓
置信度 > 0.85 && RMS > 0.015 && 65~1100Hz
    ↓
MIDI 编号计算 (A4=440Hz 标准)
    ↓
跳变检测 + 中值滤波平滑
    ↓
UI 更新 (音名、频率、音域范围、钢琴键盘)
```

## 🚀 快速开始

### 环境要求

- **Android Studio** Meerkat 或更高版本
- **JDK** 11+
- **Android SDK** API 24+（最低支持 Android 7.0）
- 目标 SDK：API 36

### 构建与运行

```bash
# 克隆项目
git clone https://github.com/tianxing-ovo/VocalScope.git
cd VocalScope

# 构建调试版本
./gradlew assembleDebug

# 安装到已连接的设备
./gradlew installDebug
```

### 权限说明

应用需要以下权限：

|       权限       |       用途        |
|:--------------:|:---------------:|
| `RECORD_AUDIO` | 通过麦克风采集人声进行音高分析 |

> 首次启动时会弹出权限请求对话框，授予后即可正常使用。

## 🎵 使用指南

1. **打开应用** → 授予录音权限
2. **点击「开始测试」** → 中央圆圈变为紫色，表示正在录音
3. **对着手机唱出不同音高** → 观察音名和频率的实时变化
4. **查看钢琴键盘** → 蓝色区域即为你的音域跨度
5. **点击「停止测试」** → 结束本次录音
6. **点击「重置记录」** → 清除音域数据，准备下一次测试

### 使用技巧

- 🎙️ 尽量在**安静环境**中测试，以获得最准确的结果
- 🗣️ 从你能发出的**最低音**缓慢滑到**最高音**，让系统完整捕捉你的声域
- ⏱️ 每个音至少**保持 0.5 秒以上**，给算法足够的稳定时间
- 📏 钢琴键盘覆盖 F2 到 D5，基本涵盖了从低音 Bass 到高音 Soprano 的完整范围

## 📂 项目结构

```
VocalScope/
├── app/
│   ├── build.gradle.kts          # 应用级构建配置
│   └── src/main/
│       ├── AndroidManifest.xml    # 应用清单（权限声明）
│       ├── java/com/ltx/vocalscope/
│       │   ├── MainActivity.kt   # 核心逻辑（音频采集 + UI）
│       │   └── ui/theme/         # Material 3 主题
│       └── res/
│           ├── mipmap-*/          # 应用图标（多分辨率）
│           ├── values/            # 字符串、颜色、主题资源
│           └── xml/               # 备份规则
├── gradle/
│   └── libs.versions.toml        # 统一依赖版本管理
├── build.gradle.kts              # 项目级构建配置
├── settings.gradle.kts           # 项目设置
├── LICENSE                       # MIT 开源协议
└── README.md                     # 本文件
```

## 🔧 自定义配置

你可以在 `MainActivity.kt` 中调整以下参数：

|      参数       |         默认值          |       说明        |
|:-------------:|:--------------------:|:---------------:|
| `sampleRate`  |       `44100`        |     采样率（Hz）     |
| `bufferSize`  |        `2048`        | TarsosDSP 缓冲区大小 |
| `probability` |      `> 0.85f`       |    YIN 置信度阈值    |
| `currentRms`  |      `> 0.015f`      | 最低音量阈值（过滤背景噪音）  |
|     频率范围      |     `65f..1100f`     |    人声检测频率限制     |
|     大跳变阈值     |      `>= 7` 半音       |   触发跳变检测的音程差    |
|    中值滤波窗口     |        `3` 帧         |     平滑窗口大小      |
|    钢琴键盘范围     | F2 ~ D5 (MIDI 41~74) |    键盘显示的音域范围    |

## 📋 版本历史

### v1.0 (2026-03-06)
- ✅ 基于 TarsosDSP YIN 算法的实时音高检测
- ✅ 钢琴键盘音域可视化（蓝色高亮 + 黄色游标）
- ✅ 多重防误判机制（跳变检测、中值滤波、RMS 过滤）
- ✅ Material 3 界面设计
- ✅ 录音时屏幕常亮
- ✅ 自适应弹性布局

## 📄 开源协议

本项目基于 [MIT License](LICENSE) 开源

# HarmonyOS Dev Skill

> AI Agent 的 HarmonyOS 应用开发专家技能库，深度整合 ArkTS / ArkUI / HarmonyOS API 12+ 完整知识体系，让 AI 助手秒变鸿蒙开发专家。

[!["Star"](https://img.shields.io/github/stars/FIEforever/harmonyos-dev-skill?style=social)](https://github.com/FIEforever/harmonyos-dev-skill)
[!["Fork"](https://img.shields.io/github/forks/FIEforever/harmonyos-dev-skill?style=social)](https://github.com/FIEforever/harmonyos-dev-skill)
[!["License"](https://img.shields.io/github/license/FIEforever/harmonyos-dev-skill)](LICENSE)

---

## What is this?

本项目是一套面向 AI 编程助手（如 Trae、Cursor、Claude Code 等）的 **HarmonyOS 开发专家技能（Skill）**，以 Markdown 文件的形式提供 AI 运行时所需的全部开发规范、API 参考和最佳实践。

简单说：**让 AI 帮你写 HarmonyOS 代码时，AI 不再「一知半解」，而是真正懂鸿蒙、写得对、写得好的专家级助手。**

---

## Why?

开发 HarmonyOS 应用时，你是不是遇到过这些情况：

- AI 生成的代码用 `@ohos.xxx` 旧式 API，开发规范已过时
- AI 不了解 Stage 模型/UIAbility 的生命周期细节
- AI 不熟悉华为 Core Vision Kit、CameraKit 等高级 Kit 的用法
- AI 写的动画/手势代码不是最优解
- AI 不知道 NDK C++ 怎么调用、一次开发多端部署怎么做

**本 Skill 解决了这些问题。** 它源自华为官方文档，经过系统性整理、结构化输出，专为 AI 运行时优化。

---

## Features

| 模块 | 内容 | 亮点 |
|------|------|------|
| **ArkUI 组件** | 九大组件体系（布局/基础UI/输入/容器/手势/动画/高级/媒体/绘制/画布） | 7 种手势全覆盖 + 完整动画系统 |
| **ArkTS 语法** | 状态管理、生命周期、路由、装饰器、泛型、并发 | @State/@Link/@Prop/@Watch 最佳实践 |
| **HarmonyOS API** | UIAbility、Stage模型、网络、存储、AI、媒体、相机 | 含完整代码示例 |
| **图形系统** | Drawing NDK、ArkGraphics 3D、Canvas、XComponent、WebGL | 6 大图形 Kit 全覆盖 |
| **多端部署** | 响应式布局断点、四种布局方案、自由流转、跨设备迁移 | 一套代码多设备运行 |
| **NDK 开发** | C++ / N-API 桥接、CMake 构建、ArkTS 调用 | CPU 密集型计算必备 |
| **DevEco 工具** | Profiler 分析模板、Testing 框架、性能优化全流程 | 开发效率翻倍 |
| **uniapp 跨平台** | HBuilderX + Vue 3 + UTS 插件 + ArkUI 嵌入 | 一套代码多端运行（App+小程序+H5+鸿蒙）|

---

## File Structure

```
harmonyos-dev-skill/
├── SKILL.md                          # 主入口 - Skill 配置与触发词
├── references/
│   ├── arkts-patterns.md             # ArkTS 语法模式参考
│   ├── arkui-components.md           # ArkUI 九大组件体系
│   ├── harmonyos-apis.md             # HarmonyOS API 12 完整参考
│   ├── graphics.md                   # 图形系统专项（6大Kit）
│   └── deveco-tools.md               # DevEco 工具 + 测试 + 性能优化
├── LICENSE                           # MIT 开源协议
├── .gitignore                        # Git 忽略配置
└── README.md                         # 本文件
```

---

## Quick Start

### 1. Download

Clone 本仓库：

```bash
git clone https://github.com/FIEforever/harmonyos-dev-skill.git
cd harmonyos-dev-skill
```

或者直接下载 [ZIP](https://github.com/FIEforever/harmonyos-dev-skill/archive/refs/heads/main.zip)。

### 2. Import to Your AI Editor

#### Trae（推荐）

1. 打开 Trae → **设置** → **规则和技能**
2. 点击 **创建** → 选择 **上传 SKILL.md 或 .zip**
3. 上传本仓库压缩包，或直接将 `harmonyos-dev-skill` 文件夹导入
4. 选择类型（全局/项目）→ 确认

或者复制到 Trae 全局技能目录：

```bash
# Windows
C:\Users\<YourName>\.trae-cn\skills\harmonyos-dev\

# macOS / Linux
~/.trae-cn/skills/harmonyos-dev/
```

#### Cursor / Claude Code

将 `SKILL.md` 内容复制到对应的 Custom Instructions 或 Project Instructions 中。

### 3. Verify

在 AI 助手中发送：

> "帮我用 HarmonyOS 写一个带相机预览和骨架识别的页面"

如果 AI 使用 `@kit.CameraKit`、`@kit.CoreVisionKit` 等新版 API 写法回复，说明 Skill 已生效。

---

## 三种鸿蒙开发方案对比

| 对比项 | harmonyos-dev-skill（综合版）| harmonyos-native-skill（纯原生）| uniapp-harmony-skill（跨平台）|
|--------|------------------------------|--------------------------------|-------------------------------|
| **Skill 仓库** | [harmonyos-dev-skill](https://github.com/FIEforever/harmonyos-dev-skill) | [harmonyos-native-skill](https://github.com/FIEforever/harmonyos-native-skill) | [uniapp-harmony-skill](https://github.com/FIEforever/uniapp-harmony-skill) |
| **覆盖范围** | 综合，含部分跨平台说明 | 纯 ArkTS + ArkUI 原生 | Vue 3 + HBuilderX 跨平台 |
| **代码复用** | 仅鸿蒙平台 | 仅鸿蒙平台 | App + 小程序 + H5 + 鸿蒙 |
| **性能** | 最佳 | 最佳 | 接近原生 |
| **学习成本** | 需要学习 ArkTS | 需要学习 ArkTS | Vue 开发者友好 |
| **原生 API** | 全量直接调用 | 全量直接调用 | 通过 UTS 插件 |
| **适用场景** | 通用鸿蒙开发 | 极致性能、纯鸿蒙应用 | 多端应用快速开发 |

### 选择建议

- **harmonyos-dev-skill（本仓库）**：综合参考，覆盖面最广，适合日常鸿蒙开发
- **[harmonyos-native-skill](https://github.com/FIEforever/harmonyos-native-skill)**：只包含纯 ArkTS 原生内容，不含跨平台，更聚焦、更精简
- **[uniapp-harmony-skill](https://github.com/FIEforever/uniapp-harmony-skill)**：适合已有 Vue 经验、需同时支持多端的开发者

---

## Key References

### ArkUI 组件速查

```typescript
// 九大组件体系全覆盖
// 布局: Row, Column, Flex, Stack, Grid, List, Scroll, Swiper, Tabs
// 基础: Text, Image, Button, Badge, Marquee, Progress, Rating
// 输入: TextInput, Search, DatePicker, Checkbox, Radio, Toggle, Slider
// 容器: Navigation, TabBar, Dialog, Sheet, SideBar, Panel
// 手势: TapGesture, LongPressGesture, PanGesture, PinchGesture, RotationGesture
// 动画: animateTo, @AnimatableExtend, TransitionEffect
```

### Stage 模型核心

```typescript
// UIAbility 生命周期
// onCreate() → onWindowStageCreate() → onForeground() → onBackground() → onDestroy()

// 状态管理（按需选用）
@State      // 组件内私有状态
@Prop       // 父→子单向传值
@Link       // 父↔子双向绑定
@Provide    // 跨组件单向
@Consume    // 配合 @Provide
@ObjectLink // 对象深层绑定
```

### 媒体 + AI 能力

```typescript
// 相机预览
import { camera } from '@kit.CameraKit';

// AI 视觉
import { visionLib } from '@kit.CoreVisionKit';
// subjectSegmentation (骨架分割) / textRecognition (文字识别)

// 语音
import { speechRecognizer } from '@kit.CoreSpeechKit';
import { textToSpeech } from '@kit.CoreSpeechKit';
```

---

## Supported AI Editors

| Editor | Support Status | Import Method |
|--------|---------------|---------------|
| **Trae** | ✅ 完整支持 | 上传 zip / 目录导入 |
| **Cursor** | ✅ 兼容 | Custom Instructions |
| **Claude Code** | ✅ 兼容 | `--project` + SKILL.md |
| **Windsurf** | ✅ 兼容 | Project Rules |
| **其他 Claude-based** | ✅ 兼容 | 复制 SKILL.md 内容 |

---

## Documentation Versions

| Version | Date | Highlights |
|---------|------|-----------|
| v2.4 | 2026-04-25 | 新增 harmonyos-native-skill（纯原生）对比说明 |
| v2.3 | 2026-04-20 | 新增 uniapp 跨平台开发方案对比 |
| v2.2 | 2026-04-20 | 全量扩充：媒体/AI/分布式/NDK/多端部署/工具链 |
| v2.1 | 2026-04-19 | 新增图形系统（6大图形Kit） |
| v2.0 | 2026-04-18 | Stage模型 + ArkTS完整规范 |
| v1.0 | 2026-04-17 | 初始版本 |

---

## Contributing

欢迎提交 Issue 和 Pull Request！

- 发现 API 错误或有更新？提交 Issue 或直接 PR
- 有更好的代码示例？欢迎分享
- 想添加新模块？先提 Discussion 讨论

---

## References

本 Skill 内容整理自以下优质文档（均为国内可访问链接）：

- [初识 ArkTS 语言 - HarmonyOS Developer](https://developer.harmonyos.cool/docs/arkts/intro/)（HarmonyOS 开发者站 ArkTS 入门）
- [ArkUI 框架实战指南 - 阿里云开发者社区](https://developer.aliyun.com/article/1631022)（2024-10-29，ArkUI 声明式开发详解）
- [HarmonyOS ArkTS 应用入门实操 - 腾讯云](https://cloud.tencent.com/developer/article/2366971)（Stage 模型 + 第一个 ArkTS 应用）
- [ArkTS 开发全场景应用 - 华为云社区](https://bbs.huaweicloud.com/blogs/438338)（2024-10-31，DevEco 环境搭建 + 完整语法 + 分布式特性）
- [Core Vision Kit 骨骼点检测 - CSDN](https://harmonyosdev.csdn.net/6952a908bf6b0e4b285f4e11.html)（2025-12-30，骨架识别 + subjectSegmentation + 官方避坑指南）
- [HarmonyOS 相机开发 - 腾讯云](https://cloud.tencent.com/developer/article/2340875)（CameraKit 相机模块 + 权限 + 拍照/预览完整流程）
- [Core Vision Kit 示例代码 - Gitee](https://gitee.com/harmonyos_codelabs/core-vision-kit-codelab-ark-ts-image-segmentation-demo)（华为官方 Codelabs 仓库，图像主体分割示例）

### 纯原生开发参考

- [harmonyos-native-skill 仓库](https://github.com/FIEforever/harmonyos-native-skill)（纯 ArkTS + ArkUI 原生开发，不含跨平台）

### uniapp 鸿蒙开发参考

- [uniapp-harmony-skill 仓库](https://github.com/FIEforever/uniapp-harmony-skill)（AI Agent 的 uniapp 鸿蒙开发专家技能库）
- [uni-app 鸿蒙开发专题 - uni-app 官网](https://uniapp.dcloud.net.cn/tutorial/harmony/dev.html)
- [运行和发行 - uni-app 官网](https://uniapp.dcloud.net.cn/tutorial/harmony/runbuild.html)
- [调用鸿蒙原生API - uni-app 官网](https://uniapp.dcloud.net.cn/tutorial/harmony/native-api.html)
- [嵌入鸿蒙原生组件 - uni-app 官网](https://uniapp.dcloud.net.cn/tutorial/harmony/native-component.html)
- [HBuilderX 下载](https://www.dcloud.io/hbuilderx.html)
- [DevEco Studio 下载](https://developer.huawei.com/consumer/cn/download/deveco-studio)

---

## License

[MIT License](LICENSE) - 随意使用，Star 和 Fork 是最好的支持！

---

> Made with ❤️ for the HarmonyOS Developer Community

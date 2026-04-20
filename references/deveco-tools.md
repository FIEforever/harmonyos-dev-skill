# DevEco Studio 工具、测试与性能优化

> 基于华为官方文档，涵盖 DevEco Studio 工具链、DevEco Testing、DevEco Profiler、Cloud Testing 云测平台完整使用指南。

---

## 一、DevEco Studio 工具链总览

| 工具 | 功能定位 | 入口 |
|------|---------|------|
| **DevEco Profiler** | 一站式性能分析 | View → Tool Windows → Profiler |
| **DevEco Testing** | 专项测试（兼容性/稳定性/性能） | 独立应用 / DevEco Studio 内置 |
| **DevEco Inspector** | UI 布局检查 | View → Tool Windows → Inspector |
| **AGC（AppGallery Connect）** | 应用发布与云测 | https://developer.huawei.com/consumer/cn/service/josp/agc/index.html |
| **hvigor** | 高性能构建工具（替代 Gradle） | 内置，命令行 `hvigor` |

---

## 二、DevEco Profiler（性能分析）

### 2.1 六大分析模板

| 模板 | 解决什么问题 | 关键指标 |
|------|-------------|---------|
| **Realtime Monitor** | 快速筛查 CPU/内存/帧率/GPU 异常 | 实时曲线，10 分钟定界 |
| **Frame** | 卡顿丢帧分析 | Lost Frames、Hitch Time、Vsync |
| **ArkUI** | 组件级卡顿定位 | Component/State 泳道 |
| **Launch** | 启动速度优化 | 各阶段耗时、热点函数 |
| **Snapshot** | 内存泄漏定位 | ArkTS 对象持有关系 |
| **Allocation/CPU** | Native 层问题 | 内存分配热点、CPU 热点函数 |

### 2.2 卡顿分析流程（Frame 模板）

```
框选卡顿时段 → 判断 RS Frame（渲染服务）/ App Frame（应用侧）
    ├── App 侧卡顿 → 切换 ArkTS Callstack 定位耗时函数
    └── Render 侧卡顿 → 查看 GPU 使用率，排查硬件合成过载
```

**关键泳道说明：**
- `RS Frame`：Render Service 侧帧数据，绿色=正常，红色=卡顿帧
- `App Frame`：应用侧帧数据，绿色=正常，红色=卡顿帧
- `Lost Frames`：丢帧数量，直接量化卡顿严重程度
- `Anomaly`：异常检测（图片解码>8.3ms、序列化>8ms）

### 2.3 ArkUI 模板：组件卡顿定位

| 问题类型 | 表现 | 优化方案 |
|---------|------|---------|
| 布局嵌套过深 | 组件层级>5层，绘制链路冗长 | 简化层级，使用 `RelativeContainer` |
| 冗余刷新 | 大型 Object 更新触发全量刷新 | **拆分对象为独立 @State** |
| 装饰器误用 | @Prop 传递大对象导致深拷贝 | 改用 `@Link` 传递引用 |
| 状态绑定异常 | 多子组件绑定同一状态变量 | 按需绑定，避免重复刷新 |

```typescript
// ❌ 反面：大型对象导致全量刷新
@State info: { name: string, avatar: string, desc: string } = {...};
this.info.age = 29;  // 触发所有 info 相关组件刷新

// ✅ 正面：拆分状态，精准更新
@State name: string = "测试用户";
@State age: number = 28;
@State avatar: string = "/images/avatar.png";
this.age = 29;  // 仅触发 age 相关组件刷新
```

### 2.4 快捷操作

| 操作 | 快捷键 |
|------|--------|
| 时间轴放大/缩小 | `W` / `S` |
| 时间轴左右移动 | `A` / `D` |
| 添加单点标记 | `M` |
| 添加时间段标记 | `Shift + M` |
| 聚焦异常时段 | `Alt + 框选` |

> **版本要求**：ArkUI Component/State 泳道需 DevEco Studio 5.1.0+；已上架应用不支持录制 ArkUI 泳道。

---

## 三、DevEco Testing（专项测试）

### 3.1 测试能力矩阵

| 能力 | 说明 | 使用方式 |
|------|------|---------|
| **兼容性测试** | 多设备安装/启动/界面显示 | 独立应用一键测试 |
| **稳定性测试** | 崩溃、无响应、内存泄漏 | 猴子测试（压力随机操作） |
| **性能测试** | CPU/内存/耗电量/流量 | 内置 Profiler |
| **安全测试** | 隐私声明、权限使用合理性 | 自动化扫描 |
| **UX 测试** | 大屏适配、视觉风格、动效 | 专项测试卡片 |
| **功耗测试** | 后台耗电、传感器使用 | 长时间监测 |

### 3.2 本地单元测试（JsUnit）

```typescript
// src/test/ets/ 目录下创建测试文件
import { describe, it, expect } from '@ohos/hypium';

describe('Calculator', () => {
  it('should add correctly', 0, () => {
    const result = add(2, 3);
    expect(result).assertEqual(5);
  });

  it('should handle zero', 0, () => {
    const result = add(0, 5);
    expect(result).assertEqual(5);
  });
});
```

### 3.3 UI 自动化测试

```typescript
import { uiTestFramework } from '@ohos.uitest';

// 通过 ID、文本、类型查找组件
uiTestFramework.getElement(uiTestFramework.locator.by.id('loginBtn'));
uiTestFramework.getElement(uiTestFramework.locator.by.text('登录'));
uiTestFramework.getElement(uiTestFramework.locator.by.type('Button'));

// 交互操作
.click();
.inputText('admin');
.swipe(500, 1000, 500, 500);  // 从 y=1000 滑到 y=500
.assertExist();   // 断言存在
.assertText('预期文本');
```

> **限制**：仅 HarmonyOS 3.0 release+ 支持；不支持权限弹窗、SystemUI 控件交互。

---

## 四、Cloud Testing 云测平台

### 4.1 接入流程

```
AGC 平台 → 质量 → 云测试 → 创建测试任务
    → 上传 release 包（必须使用发布证书签名）
    → 选择目标设备（按厂商/系统版本筛选）
    → 提交测试 → 查看报告
```

### 4.2 免费额度

| 项目 | 额度 |
|------|------|
| 每日免费时长 | **300 分钟** |
| 每日最大上传次数 | 500 次 |
| 单包大小上限 | 4 GB |

### 4.3 测试报告内容

- 问题截图 + 操作步骤复现
- 崩溃日志（JS/Native 全量）
- 性能数据（CPU/内存/帧率曲线）
- 耗电量分析

---

## 五、性能优化全流程

### 5.1 启动速度优化

```
启动阶段分解：
├── 应用进程创建（系统层）
├── Application.onCreate → 延迟初始化、异步加载
├── UIAbility.onCreate → onWindowStageCreate → 懒加载 + 分包
└── 首帧渲染完成 → 减少首屏布局复杂度、异步图片加载
```

**关键优化手段：**
- 延迟加载：非首屏模块在 `aboutToAppear` 中懒加载
- 预加载：SplashScreen 展示期间预加载数据
- 布局优化：减少嵌套层级，使用 `RelativeContainer`

### 5.2 内存优化

| 优化方向 | 具体措施 |
|---------|---------|
| 内存泄漏检测 | DevEco Memory Profiler 分析对象引用链 |
| 大图内存优化 | `Image` 组件使用 `sourceSize` 限制解码尺寸 |
| 列表内存优化 | `List`/`Grid` 设置合理的 `cachedCount` |
| 资源及时释放 | 页面销毁时取消订阅、释放动画、关闭数据库连接 |
| ArkTS 注意点 | 避免闭包捕获大对象；及时清理 `Emitter` 事件监听 |

### 5.3 渲染优化

| 层级 | 优化策略 |
|------|---------|
| 布局层 | 减少布局计算，使用 `flex` 替代百分比布局 |
| 绘制层 | 避免频繁重绘，使用 `cache` 属性缓存静态内容 |
| 合成层 | 减少离屏渲染，控制 `opacity`/`transform` 使用 |

**关键 API：**
- `@Component` 的 `reuseId` 实现组件复用
- `LazyForEach` 替代 `ForEach` 处理大数据列表
- `renderGroup` 控制渲染分组

### 5.4 卡顿分析关键指标

| 指标 | 目标值 | 阈值 |
|------|--------|------|
| 流畅度 | ≥60fps | 单帧 ≤16.6ms |
| 冷启动 | ≤2秒 | - |
| 热启动 | ≤500ms | - |
| CPU 占用 | ≤30%（无高负载） | - |

---

## 六、Hvigor 构建工具

### 6.1 常用命令

```bash
# 清理并构建
hvigor clean && hvigor build

# 构建 HAP 包
hvigor assembleHap

# 构建 release 包
hvigor assembleRelease

# 查看任务依赖
hvigor tasks --mode module -p module=entry
```

### 6.2 build-profile.json5 配置

```json5
{
  "app": {
    "compileSdkVersion": 5,
    "compatibleSdkVersion": 5,
    "products": [
      {
        "name": "default",
        "signingConfig": "signature1",
        "depends": []
      }
    ]
  },
  "modules": [
    {
      "name": "entry",
      "srcPath": "./modules/entry",
      "targets": ["ohos-archive"]
    }
  ]
}
```

---

## 七、DevEco Studio 常用快捷键

| 功能 | 快捷键 |
|------|--------|
| 打开 Command Palette | `Ctrl + Shift + P` |
| 查找文件 | `Ctrl + P` |
| 全局搜索 | `Ctrl + Shift + F` |
| 格式化代码 | `Ctrl + Shift + F` |
| 重构/重命名 | `Shift + F6` |
| 快速修复 | `Alt + Enter` |
| 打开 Profiler | View → Tool Windows → Profiler |
| 打开 Inspector | View → Tool Windows → Inspector |
| 预览 ArkUI | XTS 预览器（右侧 Previewer 面板）|
| 调试运行 | `F5` |
| 无调试运行 | `Ctrl + F5` |

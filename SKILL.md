---
name: harmonyos-dev
version: "2.0.0"
description: >
  HarmonyOS (鸿蒙) native application development with ArkTS and ArkUI.
  Use when developing HarmonyOS / OpenHarmony apps with DevEco Studio,
  writing ArkTS code, building ArkUI components, using HarmonyOS APIs
  (AbilityKit, UIAbility, Want, Router, Navigation, NavPathStack, Preferences,
  RelationalStore, Distributed, Core Vision Kit OCR/SubjectSegmentation, etc.),
  implementing state management (@State @Prop @Link @Provide @Consume @ObjectLink
  @ComponentV2 @Local @Param @Event @Monitor @Computed @ObservedV2 @Trace),
  lifecycle hooks (onCreate onWindowStageCreate onForeground onBackground onDestroy),
  page routing, network requests (http/rcp), data persistence, component reuse,
  custom components, animations, layout debugging, HAP packaging, multi-device
  adaptation, and HarmonyOS NEXT / API 12 features.
  Trigger keywords: 鸿蒙, HarmonyOS, ArkTS, ArkUI, DevEco Studio, UIAbility,
  Ability, Want, AppStorage, LocalStorage, PersistentStorage, @Builder,
  @Component, @Entry, @State, @Prop, @Link, @Observe, @ObjectLink, harmony,
  openharmony, 华为, harmonyos next, stage model, FA model, HAP, HAR, HSP,
  NavPathStack, Navigation, NavDestination, LazyForEach, subjectSegmentation,
  ArkGraphics, Drawing, Canvas, 2D绘图, 3D渲染, AR Engine, Component3D, Scene,
  SceneNode, GLB, XEngine, Graphics Accelerate, Spatial Recon, 3DGS, 高斯泼溅,
  平面检测, 运动跟踪, AR物体摆放, OH_Drawing, XComponent, 图形, 渲染, 着色器,
  GPU加速, 超分, VRS, subpass, 空间重建, 三维重建, MediaKit, AVPlayer,
  AVRecorder, SoundPool, AVCodec, 音视频, 视频播放, 音频录制, CameraKit,
  相机预览, 拍照, 录像, Core Vision Kit, Core Speech Kit, OCR, 文字识别,
  语音识别, 语音合成, TTS, ASR, HiAI, 端侧AI, 模型推理, NDK开发, C++,
  native模块, napi, CMake, 响应式布局, BreakpointSystem, 多端部署, 一次开发,
  自由流转, 分布式流转, 跨设备迁移, KV数据库, relationalStore, Preferences,
  DevEco Profiler, DevEco Testing, 云测, 性能优化, 卡顿分析, 内存优化,
  启动优化, 单元测试, UI测试, Gesture, 手势, PanGesture, PinchGesture,
  SwipeGesture, animateTo, 动画, WaterFlow, 瀑布流, SideBarContainer,
  RelativeContainer, TabBar, Sheet, Badge, Marquee, Video组件, QRCode,
  WebSocket, 推送通知, NotificationKit, AppRecovery, 错误恢复, AppStorage同步.
read_when:
  - Developing any HarmonyOS / OpenHarmony application
  - Writing ArkTS / ArkUI code
  - Asking about HarmonyOS API 12+ features
  - Working with DevEco Studio projects
  - Implementing 2D/3D graphics, AR, GPU rendering
  - Using Canvas, Drawing NDK, ArkGraphics 3D
  - Media (AVPlayer, CameraKit, MediaKit, AVCodec)
  - AI (Core Vision Kit, Core Speech Kit, HiAI)
  - Performance optimization, testing, DevEco tools
references:
  - references/arkts-patterns.md
  - references/arkui-components.md
  - references/harmonyos-apis.md
  - references/graphics.md
  - references/deveco-tools.md
---

# HarmonyOS 开发 Skill（API 12 官方规范）

> 基于华为官方文档 API 12 / HarmonyOS NEXT 编写，所有示例均遵循最新官方规范。

---

## 一、ArkTS 核心语法约束

HarmonyOS NEXT 使用**严格类型的 ArkTS**，禁止：
- `any` 类型（必须明确声明类型）
- 动态属性访问（`obj[variable]`，除非使用 `Record<string, T>`）
- `===` 代替 `==` 检查类型
- 类继承多接口时需用 `implements` 而不是 `extends`

```typescript
// ✅ 正确示例
const name: string = 'HarmonyOS';
const params = router.getParams() as Record<string, Object>;
const id = params['id'] as string;

// ❌ 错误示例
const name = 'HarmonyOS';           // 缺少类型声明
const params: any = router.getParams(); // 禁用 any
```

---

## 二、状态管理 V1 装饰器（稳定版）

### 2.1 装饰器速查表

| 装饰器 | 数据流向 | 适用场景 | 关键限制 |
|--------|---------|---------|---------|
| `@State` | 组件内部 | 私有状态 | 对象属性修改需解构赋值 |
| `@Prop` | 父→子（单向） | 只读传参 | 子组件不可直接修改 |
| `@Link` | 父↔子（双向） | 双向绑定 | 禁止在 `@Entry` 组件使用 |
| `@Provide/@Consume` | 跨层级双向 | 全局共享 | 需相同标识符 |
| `@Observed + @ObjectLink` | 属性级观测 | 对象嵌套 | 必须搭配 `@Observed` |
| `@StorageProp/@StorageLink` | AppStorage↔组件 | 应用级共享 | — |
| `@LocalStorageProp/@Link` | LocalStorage↔组件 | 页面级共享 | — |

### 2.2 @State 注意事项

```typescript
@State user: { name: string; age: number } = { name: 'Alice', age: 20 };

// ✅ 正确：解构赋值触发更新
this.user = { ...this.user, name: 'Bob' };

// ❌ 错误：直接修改属性不会触发 UI 更新
this.user.name = 'Bob';
```

### 2.3 @Link 父子双向绑定

```typescript
// 父组件（直接传 @State 变量，无需 $ 符号）
@State count: number = 0;
ChildComp({ count: this.count })

// 子组件
@Link count: number;
// 子组件修改 this.count 会同步到父组件
```

### 2.4 @Observed + @ObjectLink

```typescript
@Observed
class Product {
  name: string = '';
  price: number = 0;
  constructor(name: string, price: number) {
    this.name = name;
    this.price = price;
  }
}

// 父组件
@State product: Product = new Product('iPhone', 6999);
ProductCard({ item: this.product })

// 子组件
@Component
struct ProductCard {
  @ObjectLink item: Product;
  build() {
    Text(this.item.name)
  }
}
```

---

## 三、状态管理 V2 装饰器（API 12+）

> V2 处于试用阶段，解决了 V1 的深度监听、精确更新等痛点。

### 3.1 V1 vs V2 对照

| V1 | V2 | 说明 |
|----|----|------|
| `@Component` | `@ComponentV2` | 组件标识 |
| `@State` | `@Local` | 组件内状态（更严格，不可从外部初始化） |
| `@Prop` | `@Param` | 外部输入属性 |
| 回调函数 | `@Event` | 子→父事件传递 |
| `@Watch` | `@Monitor` | 状态监听（支持深度路径） |
| `@Provide/@Consume` | `@Provider/@Consumer` | 跨层级同步 |
| — | `@Computed` | 计算属性（仅计算一次） |
| `@Observed` | `@ObservedV2` | class 深度监听 |
| — | `@Trace` | 属性级观测（配合 @ObservedV2） |

### 3.2 @ObservedV2 + @Trace

```typescript
@ObservedV2
class UserInfo {
  @Trace name: string = '';   // 属性变化才触发 UI 更新
  @Trace age: number = 0;
  phone: string = '';          // 未标记 @Trace，不触发更新
}

@Entry
@ComponentV2
struct UserPage {
  @Local user: UserInfo = new UserInfo();

  build() {
    Column() {
      Text(this.user.name)         // 跟踪 name
      Text(`${this.user.age}`)     // 跟踪 age
      Button('改名').onClick(() => {
        this.user.name = '小明';   // ✅ 触发更新（@Trace）
      })
    }
  }
}
```

### 3.3 @Param + @Event 父子通信

```typescript
@ComponentV2
struct Counter {
  @Param count: number = 0;
  @Event onCountChange: (newVal: number) => void = () => {};

  build() {
    Button(`+1`).onClick(() => {
      this.onCountChange(this.count + 1);  // 通知父组件
    })
  }
}

// 父组件使用
@Local total: number = 0;
Counter({
  count: this.total,
  onCountChange: (v: number) => { this.total = v; }
})
```

### 3.4 @Monitor 深度监听

```typescript
@ComponentV2
struct Page {
  @Local info: UserInfo = new UserInfo();

  @Monitor('info.name')
  onNameChange(monitor: IMonitor) {
    const { before, now } = monitor.value<string>();
    console.log(`name: ${before} → ${now}`);
  }

  // 同时监听多个属性
  @Monitor('info.name', 'info.age')
  onMultiChange(monitor: IMonitor) {
    console.log('changed:', monitor.dirty);  // ['info.name'] 或 ['info.age']
  }
}
```

### 3.5 @Computed 计算属性

```typescript
@ComponentV2
struct PricePage {
  @Local price: number = 100;
  @Local discount: number = 0.8;

  @Computed
  get finalPrice(): number {
    return this.price * this.discount;  // 仅在依赖变化时计算一次
  }

  build() {
    Text(`最终价格: ${this.finalPrice}`)
  }
}
```

---

## 四、标准文件结构（.ets 骨架）

```typescript
// 1. Kit 导入（推荐新式 @kit.XxxKit 写法）
import { AbilityKit } from '@kit.AbilityKit';
import { http } from '@kit.NetworkKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

// 2. 常量 & 类型
const TAG = 'MyPage';

// 3. 数据 Model
@ObservedV2
class Item {
  @Trace id: string = '';
  @Trace name: string = '';
}

// 4. 自定义子组件
@Component
struct ItemCard {
  @Prop item: Item;
  build() {
    // UI
  }
}

// 5. 页面入口组件
@Entry
@Component
struct IndexPage {
  @State items: Item[] = [];

  aboutToAppear(): void {
    // 初始化
  }

  aboutToDisappear(): void {
    // 清理资源
  }

  build() {
    Column() {
      // 页面 UI
    }
  }
}
```

---

## 五、UIAbility 生命周期（Stage 模型）

```
创建 → onCreate(want, launchParam)
      ↓
onWindowStageCreate(windowStage)  ← 加载主页面
      ↓
onForeground()   ← 进入前台（可触发数据刷新）
      ↓
onBackground()   ← 进入后台（暂停耗资源操作）
      ↓
onWindowStageDestroy()
      ↓
onDestroy()      ← 释放全局资源
```

```typescript
// entry/src/main/ets/entryability/EntryAbility.ets
import { UIAbility, AbilityConstant, Want } from '@kit.AbilityKit';
import { window } from '@kit.ArkUI';
import { hilog } from '@kit.PerformanceAnalysisKit';

const TAG = 'EntryAbility';

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    hilog.info(0x0000, TAG, 'onCreate');
    // 初始化全局数据、Preferences 等
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    hilog.info(0x0000, TAG, 'onWindowStageCreate');
    // 加载主页面（必须在此回调中进行）
    windowStage.loadContent('pages/Index', (err) => {
      if (err) {
        hilog.error(0x0000, TAG, `loadContent failed: ${err.message}`);
      }
    });
  }

  onForeground(): void {
    hilog.info(0x0000, TAG, 'onForeground');
  }

  onBackground(): void {
    hilog.info(0x0000, TAG, 'onBackground');
  }

  onDestroy(): void {
    hilog.info(0x0000, TAG, 'onDestroy');
  }
}
```

---

## 六、页面路由

### 6.1 Navigation（推荐，API 12 标准）

Navigation 是 API 12 推荐的路由方案，替代旧版 Router。

#### 配置路由表（route_map.json）

```json
// src/main/resources/base/profile/route_map.json
{
  "routerMap": [
    {
      "name": "DetailPage",
      "pageSourceFile": "src/main/ets/pages/DetailPage.ets",
      "buildFunction": "DetailPageBuilder"
    }
  ]
}
```

#### 根页面（使用 Navigation 作为容器）

```typescript
@Entry
@Component
struct Index {
  @Provide('pageStack') pathStack: NavPathStack = new NavPathStack();

  build() {
    Navigation(this.pathStack) {
      // 根页面内容
      Button('进入详情').onClick(() => {
        // 跳转并传参
        this.pathStack.pushDestinationByName('DetailPage', { id: '123', title: '商品详情' })
          .catch((err: Error) => {
            console.error(`路由失败: ${err.message}`);
          });
      })
    }
    .title('首页')
    .mode(NavigationMode.Stack)
  }
}
```

#### 子页面（使用 NavDestination 作为容器）

```typescript
// DetailPage.ets
@Builder
export function DetailPageBuilder() {
  DetailPage()
}

@Component
export struct DetailPage {
  @Consume('pageStack') pathStack: NavPathStack;
  private pageId: string = '';
  private pageTitle: string = '';

  build() {
    NavDestination() {
      Column() {
        Text(`ID: ${this.pageId}`)
        Button('返回').onClick(() => {
          this.pathStack.pop();
        })
      }
    }
    .title(this.pageTitle)
    .onReady((ctx: NavDestinationContext) => {
      // 在 onReady 中获取参数（不要在 aboutToAppear 里拿）
      const params = ctx.pathStack.getParamByName('DetailPage')[0] as Record<string, string>;
      this.pageId = params['id'] ?? '';
      this.pageTitle = params['title'] ?? '';
    })
  }
}
```

#### NavPathStack 常用方法

```typescript
// 跳转
this.pathStack.pushPath({ name: 'PageName' });
this.pathStack.pushDestinationByName('PageName', params);
this.pathStack.replacePath({ name: 'PageName' });   // 替换当前页

// 返回
this.pathStack.pop();
this.pathStack.popToName('PageName');
this.pathStack.popToIndex(0);
this.pathStack.clear();

// 查询
this.pathStack.size();
this.pathStack.getAllPathName();
this.pathStack.getParamByName('PageName');
this.pathStack.getParamByIndex(0);
```

### 6.2 Router（旧版，兼容使用）

```typescript
import router from '@ohos.router';

// 跳转（带参数）
router.pushUrl({
  url: 'pages/Detail',
  params: { id: '123' }
}).catch((err: Error) => console.error(err.message));

// 替换（不产生返回历史）
router.replaceUrl({ url: 'pages/Login' });

// 返回
router.back();
router.back({ url: 'pages/Index' });

// 接收参数（在 aboutToAppear 中）
aboutToAppear() {
  const params = router.getParams() as Record<string, Object>;
  this.id = params['id'] as string;
}
```

> **迁移建议**：新项目使用 Navigation，旧项目可继续用 Router，两者不可混用。

---

## 七、网络请求（@kit.NetworkKit）

```typescript
import { http } from '@kit.NetworkKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

const TAG = 'HttpUtil';

/**
 * 通用 HTTP 请求工具
 */
async function httpRequest<T>(
  url: string,
  method: http.RequestMethod = http.RequestMethod.GET,
  data?: object | string,
  headers?: Record<string, string>
): Promise<T> {
  const httpRequest = http.createHttp();

  try {
    const response = await httpRequest.request(url, {
      method,
      header: {
        'Content-Type': 'application/json',
        ...headers
      },
      extraData: data,
      connectTimeout: 10000,  // 连接超时 10s
      readTimeout: 30000,     // 读取超时 30s
    });

    if (response.responseCode !== http.ResponseCode.OK) {
      throw new Error(`HTTP ${response.responseCode}`);
    }

    return JSON.parse(response.result as string) as T;
  } catch (err) {
    hilog.error(0x0000, TAG, `请求失败: ${JSON.stringify(err)}`);
    throw err;
  } finally {
    httpRequest.destroy();  // 必须释放资源
  }
}

// 使用示例
interface UserResponse {
  id: string;
  name: string;
}

async function loadUser(userId: string): Promise<void> {
  const user = await httpRequest<UserResponse>(
    `https://api.example.com/users/${userId}`
  );
  // 处理 user 数据
}
```

---

## 八、数据持久化（@kit.ArkData）

### 8.1 Preferences（轻量级 KV 存储）

> 适用场景：配置项、用户偏好（不超过 1 万条）

```typescript
import { preferences } from '@kit.ArkData';

// API 12：第二个参数改为配置对象
const pref = preferences.getPreferencesSync(context, { name: 'app.db' });

// 写入（必须调用 flush 持久化）
pref.putSync('themeMode', 'dark');
pref.putSync('fontSize', 16);
await pref.flush();

// 读取
const theme = pref.getSync('themeMode', 'light') as string;
const size = pref.getSync('fontSize', 14) as number;

// 删除
pref.deleteSync('themeMode');
await pref.flush();
```

### 8.2 关系型数据库 RelationalStore（复杂结构化数据）

```typescript
import { relationalStore, ValuesBucket } from '@kit.ArkData';

// 初始化
const STORE_CONFIG: relationalStore.StoreConfig = {
  name: 'app.db',
  securityLevel: relationalStore.SecurityLevel.S1
};
const rdb = await relationalStore.getRdbStore(context, STORE_CONFIG);

// 建表
await rdb.executeSql(
  `CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    price REAL,
    category TEXT
  )`
);

// 增
rdb.insertSync('products', { name: '项链', price: 1999.0, category: 'necklace' } as ValuesBucket);

// 查
const predicates = new relationalStore.RdbPredicates('products');
predicates.equalTo('category', 'necklace').orderByAsc('price');
const result = rdb.querySync(predicates, ['id', 'name', 'price']);
const items: Array<{ id: number; name: string; price: number }> = [];
while (result.goToNextRow()) {
  items.push({
    id: result.getLong(result.getColumnIndex('id')),
    name: result.getString(result.getColumnIndex('name')),
    price: result.getDouble(result.getColumnIndex('price'))
  });
}
result.close();  // ⚠️ 必须关闭结果集

// 改
const updateData: ValuesBucket = { price: 1599.0 };
const updatePred = new relationalStore.RdbPredicates('products');
updatePred.equalTo('id', 1);
rdb.updateSync(updateData, updatePred);

// 删
const deletePred = new relationalStore.RdbPredicates('products');
deletePred.equalTo('id', 1);
rdb.deleteSync(deletePred);
```

### 8.3 KV 数据库（分布式同步）

> ⚠️ 必须在 `onWindowStageCreate` 中初始化，不能在 `onCreate` 中

```typescript
import { distributedKVStore } from '@kit.ArkData';

// 在 onWindowStageCreate 中初始化
const kvManager = distributedKVStore.createKVManager({
  context,
  bundleName: context.applicationInfo.name
});
const kvStore = await kvManager.getKVStore<distributedKVStore.SingleKVStore>('my-store', {
  createIfMissing: true,
  encrypt: false,
  kvStoreType: distributedKVStore.KVStoreType.SINGLE_VERSION,
  securityLevel: distributedKVStore.SecurityLevel.S1
});

// 增/改
await kvStore.put('userInfo', JSON.stringify({ name: 'Alice', age: 20 }));

// 查
const str = await kvStore.get('userInfo') as string;
const user = JSON.parse(str);

// 删
await kvStore.delete('userInfo');
```

---

## 九、权限声明与动态申请（API 12）

### 9.1 module.json5 静态声明

```json
// module.json5 → module → requestPermissions
"requestPermissions": [
  {
    "name": "ohos.permission.INTERNET",
    "reason": "$string:internet_reason"
  },
  {
    "name": "ohos.permission.LOCATION",
    "reason": "$string:location_reason",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inuse"
    }
  },
  {
    "name": "ohos.permission.CAMERA",
    "reason": "$string:camera_reason",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inuse"
    }
  }
]
```

> `user_grant` 级权限（位置、相机、麦克风等）必须填 `reason` 和 `usedScene`。

### 9.2 动态申请权限（API 12 标准流程）

```typescript
import abilityAccessCtrl, { Permissions } from '@ohos.abilityAccessCtrl';
import bundleManager from '@ohos.bundle.bundleManager';

/**
 * 检查权限状态
 */
function checkPermission(permission: Permissions): boolean {
  const atManager = abilityAccessCtrl.createAtManager();
  const bundleInfo = bundleManager.getBundleInfoForSelfSync(
    bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
  );
  const tokenId = bundleInfo.appInfo.accessTokenId;
  const grantStatus = atManager.checkAccessTokenSync(tokenId, permission);
  return grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED; // 0
}

/**
 * 首次申请权限
 */
async function requestPermission(
  context: Context,
  permissions: Array<Permissions>
): Promise<Array<Permissions>> {
  const atManager = abilityAccessCtrl.createAtManager();
  const result = await atManager.requestPermissionsFromUser(context, permissions);
  const denied: Array<Permissions> = [];

  result.authResults.forEach((status, index) => {
    const dialogShown = result.dialogShownResults?.[index] ?? false;
    if (status !== 0 && !dialogShown) {
      denied.push(permissions[index]);
    }
  });
  return denied;
}

/**
 * 二次申请（引导至系统设置，API 12 新增）
 */
async function requestPermissionOnSetting(
  context: Context,
  permissions: Array<Permissions>
): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager();
  const results = await atManager.requestPermissionOnSetting(context, permissions);
  return results.every(s => s === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED);
}

// 使用示例（完整两步流程）
async function ensureLocationPermission(context: Context): Promise<boolean> {
  const perms: Array<Permissions> = ['ohos.permission.APPROXIMATELY_LOCATION'];

  if (checkPermission(perms[0])) return true;  // 已授权

  const denied = await requestPermission(context, perms);  // 首次申请
  if (denied.length === 0) return true;

  return requestPermissionOnSetting(context, denied);  // 引导设置
}
```

---

## 十、日志（推荐用法）

```typescript
import { hilog } from '@kit.PerformanceAnalysisKit';

const DOMAIN = 0x0000;  // 业务域，16进制
const TAG = 'MyFeature';

hilog.debug(DOMAIN, TAG, '调试信息: %{public}s', someValue);
hilog.info(DOMAIN, TAG, '初始化完成');
hilog.warn(DOMAIN, TAG, '注意: %{public}d', warningCode);
hilog.error(DOMAIN, TAG, '错误: %{public}s', error.message);
```

---

## 十一、核心开发原则

1. **类型安全**：严格声明所有变量类型，禁用 `any`，用 `unknown` + 类型守卫代替。
2. **响应式驱动**：仅通过装饰器状态变量驱动 UI，不要直接操作 DOM（ArkUI 没有 DOM）。
3. **资源管理**：HTTP 请求在 `finally` 中 `destroy()`；RDB 结果集必须 `close()`；视觉服务在 `aboutToDisappear` 中 `release()`。
4. **生命周期边界**：KV 数据库在 `onWindowStageCreate` 初始化；Preferences 可在 `onCreate` 初始化。
5. **新 Kit 写法**：优先 `import { xx } from '@kit.XxxKit'`，而不是旧的 `@ohos.xxx` 导入。
6. **路由优先级**：新项目用 Navigation + NavPathStack；已有项目保持 Router 一致性。
7. **日志规范**：用 `hilog` 而不是 `console.log`，生产代码避免打印敏感信息。

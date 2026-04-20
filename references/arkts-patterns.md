# ArkTS 高级开发模式（API 12 官方规范）

---

## 一、LazyForEach + IDataSource（列表性能优化）

> **规范**：数据量 ≥ 50 项必须用 `LazyForEach`，禁止 `ForEach`。

### 1.1 IDataSource 接口规范（API 12+）

```typescript
// API 12+ 接口命名（注意与旧版的区别）
interface IDataSource {
  totalCount(): number;
  getData(index: number): Object;
  registerDataObserver(observer: DataObserver): void;      // ⚠️ 旧版叫 registerDataChangeListener
  unregisterDataObserver(observer: DataObserver): void;
}

interface DataObserver {
  onDataReload(): void;           // ⚠️ 旧版叫 onDataReloaded
  onDataAdd(index: number, count: number): void;
  onDataDelete(index: number, count: number): void;
  onDataChange(index: number, count: number): void;
  onDataMove(from: number, to: number): void;
}
```

### 1.2 企业级 BaseDataSource 模板

```typescript
// 通用 BaseDataSource 基类（必须继承，禁止自行实现 IDataSource）
class BaseDataSource<T> implements IDataSource {
  protected dataArray: T[] = [];
  private observers: DataObserver[] = [];

  // ===== IDataSource 必须实现的接口 =====
  totalCount(): number {
    return this.dataArray.length;
  }

  getData(index: number): T {
    return this.dataArray[index];
  }

  registerDataObserver(observer: DataObserver): void {
    if (this.observers.indexOf(observer) === -1) {
      this.observers.push(observer);
    }
  }

  unregisterDataObserver(observer: DataObserver): void {
    const i = this.observers.indexOf(observer);
    if (i !== -1) this.observers.splice(i, 1);
  }

  // ===== 数据操作方法（带通知） =====

  // 全量刷新（首次加载用，加载更多禁止用）
  refreshData(newData: T[]): void {
    this.dataArray = [...newData];
    this.observers.forEach(o => o.onDataReload());
  }

  // 增量追加（加载更多用）
  addData(newData: T[]): void {
    const startIndex = this.dataArray.length;
    this.dataArray.push(...newData);
    this.observers.forEach(o => o.onDataAdd(startIndex, newData.length));
  }

  // 删除单条
  deleteData(index: number): void {
    this.dataArray.splice(index, 1);
    this.observers.forEach(o => o.onDataDelete(index, 1));
  }

  // 更新单条
  updateData(index: number, item: T): void {
    this.dataArray[index] = item;
    this.observers.forEach(o => o.onDataChange(index, 1));
  }
}
```

### 1.3 完整业务使用示例

```typescript
interface Product {
  id: string;
  name: string;
  price: number;
}

class ProductDataSource extends BaseDataSource<Product> {}

@Entry
@Component
struct ProductListPage {
  private dataSource: ProductDataSource = new ProductDataSource();
  @State isLoading: boolean = false;
  private page: number = 1;
  private hasMore: boolean = true;

  aboutToAppear(): void {
    this.loadData(true);
  }

  async loadData(isFirst: boolean = false): Promise<void> {
    if (this.isLoading || (!isFirst && !this.hasMore)) return;
    this.isLoading = true;

    try {
      // 模拟接口请求
      const newItems: Product[] = await fetchProducts(this.page);
      if (isFirst) {
        this.dataSource.refreshData(newItems);   // 首次：全量刷新
      } else {
        this.dataSource.addData(newItems);       // 加载更多：增量追加
      }
      this.hasMore = newItems.length >= 20;
      this.page++;
    } finally {
      this.isLoading = false;
    }
  }

  build() {
    Column() {
      List({ space: 8 }) {
        LazyForEach(
          this.dataSource,
          (item: Product, _index: number) => {
            ListItem() {
              ProductCard({ product: item })
            }
            .reuseId('product-card')  // ⚡ 必须设置，性能提升 50%+
          },
          (item: Product) => item.id  // 唯一 key，避免用 index
        )
      }
      .cachedCount(3)               // 预加载 3 屏
      .layoutWeight(1)
      .onReachEnd(() => this.loadData())
    }
  }
}
```

### 1.4 性能规范

| 规则 | 要求 | 说明 |
|------|------|------|
| `reuseId` | **必须设置** | 布局相同的项用同一 `reuseId`，性能提升 50%+ |
| `cachedCount` | 普通列表 2-3 | 过大增加内存，过小会白屏 |
| 布局层级 | ≤ 5 层 | 超过则拆分子组件 |
| `build()` | 禁止复杂计算 | 数据预处理放 DataSource 层 |
| key 生成器 | 用唯一业务 ID | 禁止用 `index`，否则删除时错乱 |

### 1.5 组件复用生命周期

```typescript
@Reusable
@Component
struct ProductCard {
  @Prop product: Product;

  // 组件被复用前调用（重置状态，防止数据残留）
  aboutToReuse(params: Record<string, Object>): void {
    this.product = params['product'] as Product;
  }

  // 组件被回收时调用（释放大资源）
  aboutToRecycle(): void {
    // 释放图片缓存、取消动画等
  }

  build() {
    Column() {
      Text(this.product.name)
      Text(`¥${this.product.price}`)
    }
  }
}
```

---

## 二、@Builder 和 @BuilderParam（UI 插槽模式）

```typescript
// @Builder：构建函数，不持有状态
@Builder
function GlobalButton(label: string, onClick: () => void) {
  Button(label)
    .onClick(onClick)
    .width('100%')
    .height(44)
}

// 组件内 @Builder（可访问 this）
@Component
struct Card {
  title: string = '';

  @Builder
  buildHeader() {
    Row() {
      Text(this.title).fontSize(18).fontWeight(FontWeight.Bold)
      Blank()
    }
    .width('100%')
    .padding({ bottom: 8 })
  }

  build() {
    Column() {
      this.buildHeader()
      // ... 内容
    }
  }
}

// @BuilderParam：插槽（类似 slot）
@Component
struct Container {
  @BuilderParam content: () => void = this.defaultContent;

  @Builder
  defaultContent() {
    Text('默认内容')
  }

  build() {
    Column() {
      this.content()  // 渲染传入的内容
    }
    .padding(16)
    .borderRadius(8)
    .backgroundColor('#F5F5F5')
  }
}

// 使用插槽
Container() {
  Text('自定义内容')
  Button('点击')
}
```

---

## 三、自定义 Dialog

```typescript
// 自定义弹窗组件
@CustomDialog
struct ConfirmDialog {
  controller: CustomDialogController;
  title: string = '确认';
  message: string = '';
  onConfirm: () => void = () => {};

  build() {
    Column({ space: 20 }) {
      Text(this.title).fontSize(18).fontWeight(FontWeight.Bold)
      Text(this.message).fontSize(14).fontColor('#666666')
      Row({ space: 12 }) {
        Button('取消')
          .backgroundColor('#EEEEEE')
          .fontColor('#333333')
          .layoutWeight(1)
          .onClick(() => this.controller.close())
        Button('确认')
          .backgroundColor('#007DFF')
          .layoutWeight(1)
          .onClick(() => {
            this.onConfirm();
            this.controller.close();
          })
      }
    }
    .padding(24)
    .width(300)
  }
}

// 使用弹窗
@Entry
@Component
struct MyPage {
  dialogController: CustomDialogController = new CustomDialogController({
    builder: ConfirmDialog({
      title: '删除确认',
      message: '确定要删除这条记录吗？',
      onConfirm: () => this.handleDelete()
    }),
    alignment: DialogAlignment.Center,
    cornerRadius: 16,
    autoCancel: true
  });

  handleDelete(): void {
    // 删除逻辑
  }

  build() {
    Button('删除').onClick(() => this.dialogController.open())
  }
}
```

---

## 四、动画

### 4.1 属性动画（animateTo）

```typescript
@Component
struct AnimPage {
  @State scale: number = 1.0;
  @State opacity: number = 1.0;

  build() {
    Column() {
      Image($r('app.media.icon'))
        .scale({ x: this.scale, y: this.scale })
        .opacity(this.opacity)
        .animation({              // 属性动画（隐式）
          duration: 300,
          curve: Curve.EaseInOut,
          iterations: 1
        })

      Button('放大').onClick(() => {
        // animateTo：显式动画（精确控制）
        animateTo({ duration: 400, curve: Curve.Spring }, () => {
          this.scale = 1.5;
        });
      })
    }
  }
}
```

### 4.2 页面转场（pageTransition）

```typescript
@Entry
@Component
struct PageA {
  pageTransition() {
    // 进入动画
    PageTransitionEnter({ duration: 300, curve: Curve.EaseOut })
      .slide(SlideEffect.Right)

    // 退出动画
    PageTransitionExit({ duration: 300, curve: Curve.EaseIn })
      .slide(SlideEffect.Left)
  }

  build() {
    Column() { /* ... */ }
  }
}
```

### 4.3 组件转场（transition）

```typescript
@Component
struct FadeCard {
  @State show: boolean = false;

  build() {
    Column() {
      if (this.show) {
        Text('淡入内容')
          .transition(TransitionEffect.OPACITY   // 透明度
            .combine(TransitionEffect.scale({ x: 0.8, y: 0.8 }))  // 缩放
            .animation({ duration: 300, curve: Curve.EaseOut })
          )
      }
      Button('切换').onClick(() => {
        animateTo({ duration: 300 }, () => {
          this.show = !this.show;
        })
      })
    }
  }
}
```

---

## 五、手势识别

```typescript
@Component
struct GesturePage {
  @State offsetX: number = 0;
  @State offsetY: number = 0;
  @State scale: number = 1.0;

  build() {
    Column() {
      Image($r('app.media.photo'))
        .translate({ x: this.offsetX, y: this.offsetY })
        .scale({ x: this.scale, y: this.scale })
        .gesture(
          // 手势组合：拖动 + 捏合
          GestureGroup(
            GestureMode.Simultaneous,
            // 拖动手势
            PanGesture({ direction: PanDirection.All })
              .onActionUpdate((event: GestureEvent) => {
                this.offsetX += event.offsetX;
                this.offsetY += event.offsetY;
              }),
            // 捏合缩放
            PinchGesture()
              .onActionUpdate((event: GestureEvent) => {
                this.scale = event.scale;
              })
          )
        )
    }
  }
}
```

---

## 六、AppStorage + PersistentStorage

```typescript
// EntryAbility.onCreate 中初始化持久化
PersistentStorage.persistProp('language', 'zh-CN');
PersistentStorage.persistProp('themeMode', 'light');

// 组件中使用（自动与 AppStorage 双向同步）
@StorageLink('themeMode') themeMode: string = 'light';
@StorageProp('language') language: string = 'zh-CN';  // 单向读取

// 非组件代码中访问
AppStorage.set('userToken', 'Bearer xxx');
const token = AppStorage.get<string>('userToken');
```

---

## 七、常见错误与修复

| 错误现象 | 原因 | 修复 |
|---------|------|------|
| 修改对象属性 UI 不更新 | 直接修改 @State 对象属性 | 用解构赋值 `this.obj = {...this.obj, key: val}` |
| LazyForEach 内容错乱 | reuseId 设置不当 | 检查 reuseId 一致性，在 aboutToReuse 重置状态 |
| 列表滑动白屏 | cachedCount 过小 | 增大 cachedCount 到 3-5 |
| 路由跳转失败 | 页面未在 route_map.json 配置 | 检查 routerMap 配置，路径区分大小写 |
| @Link 不生效 | 在 @Entry 组件中使用 @Link | @Link 不能用于 @Entry 组件 |
| KV 数据库初始化失败 | 在 onCreate 中初始化 | 改到 onWindowStageCreate 中初始化 |
| 网络请求内存泄漏 | 未调用 httpRequest.destroy() | 在 finally 中调用 destroy() |
| 权限申请返回 [2,2,2] | 同时申请了不可动态申请的权限 | 分开申请，LOCATION_IN_BACKGROUND 不可动态申请 |
| @Prop 子组件修改无效 | 试图直接修改 @Prop 变量 | 通过 @Event 或回调通知父组件修改 |

# ArkUI 组件快速参考（API 12 官方规范）

> 基于华为官方文档 API 12 / HarmonyOS NEXT，涵盖 ArkUI 九大组件体系、完整手势系统、动画系统。

---

## 一、布局组件

### 1.1 Column / Row（线性布局）

```typescript
// Column：垂直排列
Column({ space: 12 }) {      // space：子组件间距
  Text('标题')
  Text('内容')
}
.width('100%')
.padding({ top: 16, left: 16, right: 16, bottom: 16 })
.alignItems(HorizontalAlign.Start)   // 水平对齐
.justifyContent(FlexAlign.Center)    // 垂直对齐

// Row：水平排列
Row({ space: 8 }) {
  Image($r('app.media.icon')).size({ width: 40, height: 40 })
  Text('标签').layoutWeight(1)       // 占剩余空间
  Button('操作')
}
.width('100%')
.alignItems(VerticalAlign.Center)    // 垂直对齐
.justifyContent(FlexAlign.SpaceBetween)  // 两端对齐
```

### 1.2 ArkUI 九大组件体系（完整分类）

ArkUI 组件按功能分为九大类，涵盖布局、基础UI、输入、容器、手势、动画、高级组件、状态管理、网络存储。

#### 分类速查表

| 分类 | 组件 | 说明 |
|------|------|------|
| **布局控件** | Row, Column, Flex, Stack, Grid, List, Scroll, Swiper, Tabs | 页面结构与排列 |
| **基础UI** | Text, Image, Button, Icon, Badge, Marquee, Progress, LoadingProgress, Rating | 界面基础元素 |
| **输入控件** | TextInput, Search, DatePicker, TimePicker, Checkbox, Radio, Toggle, Slider, Picker | 用户数据录入 |
| **容器控件** | Navigation, TabBar, Dialog, Popover, Sheet, SideBar, Panel | 页面导航与弹层 |
| **手势动画** | Gesture, Transition, AnimationController, MotionPath | 交互与动效 |
| **高级控件** | Web, Canvas, Video, Map, QRCode, XComponent | 复杂场景 |
| **媒体组件** | Video, Audio, Camera, XComponent | 音视频播放 |
| **绘制组件** | Shape, Circle, Ellipse, Rect, Path, Line, Polyline, Polygon | 矢量图形 |
| **画布组件** | Canvas, OffscreenCanvas, WebGL | 自定义绘制 |

---

### 1.2 Stack（层叠布局）

```typescript
Stack({ alignContent: Alignment.BottomEnd }) {
  Image($r('app.media.photo'))
    .width('100%').height(200)
  Text('角标')
    .backgroundColor('#FF0000')
    .padding({ left: 8, right: 8, top: 4, bottom: 4 })
    .borderRadius(4)
}
```

### 1.3 Flex（弹性布局）

```typescript
Flex({
  direction: FlexDirection.Row,
  wrap: FlexWrap.Wrap,               // 换行
  justifyContent: FlexAlign.SpaceAround,
  alignItems: ItemAlign.Center
}) {
  ForEach(tags, (tag: string) => {
    Text(tag)
      .padding({ left: 12, right: 12, top: 6, bottom: 6 })
      .backgroundColor('#EEF2FF')
      .borderRadius(20)
  })
}
```

### 1.4 Grid（网格布局）

```typescript
Grid() {
  ForEach(items, (item: ItemType) => {
    GridItem() {
      Column() {
        Image(item.imageUrl)
          .aspectRatio(1)        // 宽高比 1:1
        Text(item.name).fontSize(12)
      }
    }
  })
}
.columnsTemplate('1fr 1fr 1fr')    // 3 列等宽
.rowsGap(12)
.columnsGap(12)
.layoutDirection(GridDirection.Row)
```

### 1.5 List（滚动列表）

```typescript
List({ space: 8, initialIndex: 0 }) {
  // 使用 LazyForEach 替代 ForEach（数据量 ≥ 50 必须用）
  LazyForEach(this.dataSource, (item: ItemType) => {
    ListItem() {
      Row() {
        Text(item.name).layoutWeight(1)
        Text(item.value).fontColor('#999999')
      }
      .padding(16)
      .width('100%')
    }
    .reuseId('list-item')
  }, (item: ItemType) => item.id)
}
.listDirection(Axis.Vertical)
.edgeEffect(EdgeEffect.Fade)       // 边缘回弹效果
.cachedCount(3)
.onReachEnd(() => this.loadMore())
.divider({ strokeWidth: 0.5, color: '#EEEEEE' })  // 分割线
```

### 1.6 Scroll（可滚动容器）

```typescript
Scroll(this.scrollController) {
  Column({ space: 12 }) {
    // 内容...
  }
  .width('100%')
}
.scrollable(ScrollDirection.Vertical)
.scrollBar(BarState.Auto)
.scrollBarColor('#CCCCCC')
.onScroll((xOffset: number, yOffset: number) => {
  // 监听滚动
})
.onScrollEnd(() => {
  // 滚动停止
})
```

### 1.7 Tabs（标签页）

```typescript
@Entry
@Component
struct TabsPage {
  @State currentIndex: number = 0;
  private controller: TabsController = new TabsController();

  @Builder
  tabBar(title: string, index: number) {
    Column() {
      Text(title)
        .fontColor(this.currentIndex === index ? '#007DFF' : '#999999')
        .fontSize(14)
        .fontWeight(this.currentIndex === index ? FontWeight.Bold : FontWeight.Normal)
      Divider()
        .strokeWidth(2)
        .color('#007DFF')
        .opacity(this.currentIndex === index ? 1 : 0)
        .width(24)
        .margin({ top: 4 })
    }
    .padding({ top: 8, bottom: 8 })
  }

  build() {
    Tabs({ controller: this.controller, index: this.currentIndex }) {
      TabContent() {
        // 首页内容
      }
      .tabBar(this.tabBar('首页', 0))

      TabContent() {
        // 分类内容
      }
      .tabBar(this.tabBar('分类', 1))
    }
    .barPosition(BarPosition.Start)   // 标签栏在顶部
    .barMode(BarMode.Fixed)
    .onChange((index: number) => {
      this.currentIndex = index;
    })
  }
}
```

### 1.8 Navigation（导航容器，API 12 推荐）

> 参见 SKILL.md 第六节，这里仅列出组件属性速查

```typescript
Navigation(this.pathStack)
  .title('页面标题')
  .mode(NavigationMode.Stack)        // Stack / Split / Auto
  .hideTitleBar(false)
  .hideNavBar(false)
  .navBarWidth(240)                  // Split 模式侧栏宽度
  .toolbarConfiguration([
    { value: '首页', icon: $r('app.media.home') },
    { value: '我的', icon: $r('app.media.profile') }
  ])
```

---

## 二、基础组件

### 2.1 Text

```typescript
Text('普通文本')
  .fontSize(16)
  .fontColor('#333333')
  .fontWeight(FontWeight.Medium)      // Normal / Medium / Bold / Lighter / Bolder
  .fontStyle(FontStyle.Italic)
  .textAlign(TextAlign.Center)        // Start / Center / End / Justify
  .lineHeight(24)
  .letterSpacing(1)
  .maxLines(2)                        // 最多显示 2 行
  .textOverflow({ overflow: TextOverflow.Ellipsis })  // 超出省略
  .decoration({                       // 下划线/删除线
    type: TextDecorationType.Underline,
    color: '#007DFF'
  })
  .copyOption(CopyOptions.InApp)      // 允许复制
```

### 2.2 TextInput

```typescript
@State inputValue: string = '';

TextInput({ placeholder: '请输入内容', text: this.inputValue })
  .fontSize(16)
  .fontColor('#333333')
  .placeholderColor('#AAAAAA')
  .placeholderFont({ size: 16 })
  .maxLength(100)
  .type(InputType.Normal)             // Normal / Password / Email / PhoneNumber / Number / URL
  .enterKeyType(EnterKeyType.Done)    // Done / Send / Search / Next / Go
  .showPasswordIcon(true)             // 密码类型显示眼睛图标
  .onChange((value: string) => {
    this.inputValue = value;
  })
  .onSubmit((enterKey: EnterKeyType) => {
    // 确认键触发
  })
```

### 2.3 Button

```typescript
// 主要按钮
Button('确认', { type: ButtonType.Capsule })
  .width(200)
  .height(48)
  .fontSize(16)
  .fontColor(Color.White)
  .backgroundColor('#007DFF')
  .onClick(() => { /* ... */ })

// 图标按钮
Button({ type: ButtonType.Circle }) {
  Image($r('app.media.add'))
    .size({ width: 24, height: 24 })
}
.size({ width: 48, height: 48 })
.backgroundColor('#007DFF')

// 加载状态按钮
Button({ type: ButtonType.Capsule }) {
  Row({ space: 8 }) {
    if (this.loading) {
      LoadingProgress().size({ width: 20, height: 20 }).color(Color.White)
    }
    Text(this.loading ? '加载中...' : '提交').fontColor(Color.White)
  }
}
.enabled(!this.loading)
.backgroundColor(this.loading ? '#AAAAAA' : '#007DFF')
```

### 2.4 Image

```typescript
// 网络图片
Image('https://example.com/image.jpg')
  .width(200)
  .height(150)
  .objectFit(ImageFit.Cover)    // Contain / Cover / Fill / ScaleDown / None / Auto
  .borderRadius(8)
  .interpolation(ImageInterpolation.High)   // 插值质量
  .onComplete((event) => {
    console.log(`图片尺寸: ${event.width}x${event.height}`);
  })
  .onError(() => {
    console.error('图片加载失败');
  })

// 本地资源
Image($r('app.media.logo'))
  .width(80)
  .height(80)
  .objectFit(ImageFit.Contain)
```

### 2.5 Toggle / Checkbox / Radio

```typescript
// Toggle 开关
Toggle({ type: ToggleType.Switch, isOn: this.isEnabled })
  .selectedColor('#007DFF')
  .onChange((isOn: boolean) => {
    this.isEnabled = isOn;
  })

// Checkbox 复选框
Checkbox({ name: 'item1', group: 'checkboxGroup' })
  .select(this.checked)
  .selectedColor('#007DFF')
  .onChange((checked: boolean) => {
    this.checked = checked;
  })

// Radio 单选
Radio({ value: 'option1', group: 'radioGroup' })
  .checked(this.selected === 'option1')
  .radioStyle({ checkedBackgroundColor: '#007DFF' })
  .onChange((isChecked: boolean) => {
    if (isChecked) this.selected = 'option1';
  })
```

### 2.6 Slider

```typescript
Slider({
  value: this.sliderValue,
  min: 0,
  max: 100,
  step: 1,
  style: SliderStyle.OutSet    // OutSet / InSet
})
.trackColor('#EEEEEE')
.selectedColor('#007DFF')
.showTips(true)               // 滑动时显示数值
.onChange((value: number, mode: SliderChangeMode) => {
  this.sliderValue = value;
  if (mode === SliderChangeMode.End) {
    // 释放时才触发最终逻辑
  }
})
```

### 2.7 Progress

```typescript
// 线形进度条
Progress({ value: this.progress, total: 100, type: ProgressType.Linear })
  .width('100%')
  .height(4)
  .color('#007DFF')
  .backgroundColor('#EEEEEE')

// 环形进度条
Progress({ value: 75, total: 100, type: ProgressType.Ring })
  .size({ width: 80, height: 80 })
  .color('#007DFF')
  .style({ strokeWidth: 8, scaleCount: 20, scaleWidth: 3 })
```

### 2.8 LoadingProgress（加载动画）

```typescript
LoadingProgress()
  .color('#007DFF')
  .size({ width: 40, height: 40 })
```

---

## 三、视觉修饰符速查

```typescript
// 尺寸
.width('100%')         // 百分比宽度
.height(200)           // 固定高度 vp
.size({ width: 100, height: 100 })
.aspectRatio(16 / 9)   // 宽高比
.layoutWeight(1)       // 弹性占比（等同 flex: 1）
.constraintSize({ minWidth: 100, maxWidth: 300 })

// 外观
.backgroundColor('#FFFFFF')
.borderRadius(12)
.borderRadius({ topLeft: 12, topRight: 12, bottomLeft: 0, bottomRight: 0 })
.border({ width: 1, color: '#EEEEEE', style: BorderStyle.Solid })
.shadow({ radius: 8, color: 'rgba(0,0,0,0.1)', offsetX: 0, offsetY: 2 })
.opacity(0.8)

// 间距
.padding(16)
.padding({ top: 8, right: 16, bottom: 8, left: 16 })
.margin({ bottom: 12 })

// 定位
.position({ x: 10, y: 20 })        // 绝对定位（相对父容器）
.offset({ x: 10, y: 0 })           // 相对偏移
.zIndex(10)                         // 层级

// 变换
.rotate({ angle: 45 })
.scale({ x: 1.5, y: 1.5 })
.translate({ x: 10, y: 0 })

// 裁剪
.clip(true)                         // 裁剪溢出内容
.clip(new Circle({ width: 100, height: 100 }))  // 圆形裁剪

// 响应式（针对折叠屏/平板）
.onAreaChange((oldArea: Area, newArea: Area) => {
  // 尺寸变化监听
})
```

---

## 四、手势与事件

```typescript
Column()
  .onClick((event: ClickEvent) => {
    console.log(`点击位置: ${event.x}, ${event.y}`);
  })
  .onLongPress(() => {
    // 长按
  })
  .gesture(
    TapGesture({ count: 2 }).onAction(() => {
      // 双击
    })
  )
  .gesture(
    SwipeGesture({ direction: SwipeDirection.Horizontal })
      .onAction((event: GestureEvent) => {
        if (event.angle > 0) { /* 向右滑 */ }
        else { /* 向左滑 */ }
      })
  )
```

---

## 五、自定义组件生命周期

```typescript
@Component
struct MyComponent {
  // 组件创建时（可初始化数据）
  aboutToAppear(): void {
    // 订阅事件、加载数据
  }

  // 组件销毁前（清理资源）
  aboutToDisappear(): void {
    // 取消定时器、取消网络请求、释放 PixelMap
  }

  // 组件即将被复用（@Reusable 组件）
  aboutToReuse(params: Record<string, Object>): void {
    // 用新 params 重置状态
  }

  // 组件即将被回收
  aboutToRecycle(): void {
    // 释放大资源（图片、视频等）
  }

  // 页面显示（Navigation 子页面，onReady 中获取参数）
  // onPageShow / onPageHide（Router 模式页面）

  build() {
    // UI 声明
  }
}
```

---

## 六、Swiper（轮播）

```typescript
@Entry
@Component
struct BannerPage {
  private swiperController: SwiperController = new SwiperController();
  @State currentBanner: number = 0;

  build() {
    Column() {
      Swiper(this.swiperController) {
        ForEach([1, 2, 3], (i: number) => {
          Image($r(`app.media.banner${i}`))
            .width('100%')
            .height(200)
            .objectFit(ImageFit.Cover)
        })
      }
      .loop(true)                      // 循环播放
      .autoPlay(true)
      .interval(3000)
      .indicator(                      // 指示器样式
        Indicator.dot()
          .selectedItemWidth(16)
          .selectedItemHeight(6)
          .selectedColor('#007DFF')
          .itemWidth(6)
          .itemHeight(6)
          .color('#CCCCCC')
      )
      .onChange((index: number) => {
        this.currentBanner = index;
      })
    }
  }
}
```

---

## 七、Sheet（底部弹出面板，API 12 新增）

```typescript
@Entry
@Component
struct SheetPage {
  @State showSheet: boolean = false;

  build() {
    Column() {
      Button('打开面板').onClick(() => {
        this.showSheet = true;
      })
    }
    .bindSheet(
      $$this.showSheet,           // 双向绑定显示状态
      this.sheetContent,          // 面板内容 Builder
      {
        height: SheetSize.MEDIUM,   // MEDIUM(50%) / LARGE(90%) / 具体 vp 值
        dragBar: true,             // 拖拽把手
        onDisappear: () => {
          this.showSheet = false;
        }
      }
    )
  }

  @Builder
  sheetContent() {
    Column({ space: 16 }) {
      Text('底部面板标题').fontSize(18).fontWeight(FontWeight.Bold)
      // 内容...
    }
    .padding(24)
  }
}
```

---

## 八、完整手势系统（Gesture API）

HarmonyOS ArkUI 提供7种基础手势识别器，支持单手、组合手势、并行手势组。

### 8.1 七种基础手势

| 手势类型 | 类名 | 触发条件 | 典型场景 |
|----------|------|----------|----------|
| **点击** | `TapGesture` | 手指轻点 | 按钮点击、选择 |
| **长按** | `LongPressGesture` | 按住指定时长 | 右键菜单、拖拽开始 |
| **滑动手势** | `PanGesture` | 滑动距离≥设定值 | 翻页、左滑删除 |
| **捏合手势** | `PinchGesture` | 双指缩放 | 图片缩放 |
| **旋转手势** | `RotationGesture` | 双指旋转 | 图片旋转 |
| **方向手势** | `SwipeGesture` | 快速滑动方向 | 侧滑切换 Tab |
| **手势组** | `GestureGroup` | 组合多种手势 | 复杂交互 |

### 8.2 完整手势代码示例

```typescript
@Entry
@Component
struct GestureDemo {
  @State scale: number = 1;
  @State angle: number = 0;
  @State offsetX: number = 0;
  @State offsetY: number = 0;

  build() {
    Column() {
      Image($r('app.media.sample'))
        .width(200).height(200)
        .scale({ x: this.scale, y: this.scale })
        .rotate({ angle: this.angle })
        .translate({ x: this.offsetX, y: this.offsetY })
        // 单击手势
        .onClick(() => {
          console.info('单击');
        })
        // 长按手势（500ms）
        .gesture(
          LongPressGesture({ duration: 500 })
            .onAction(() => {
              console.info('长按触发');
            })
        )
        // 滑动手势（PanGesture）
        .gesture(
          PanGesture()
            .onActionUpdate((event: GestureEvent) => {
              this.offsetX += event.offsetX;
              this.offsetY += event.offsetY;
            })
            .onActionEnd(() => {
              console.info(`滑动结束: x=${this.offsetX}, y=${this.offsetY}`);
            })
        )
        // 捏合手势（双指缩放）
        .gesture(
          PinchGesture()
            .onActionUpdate((event: GestureEvent) => {
              this.scale = this.scale * event.scale;
            })
            .onActionEnd(() => {
              // 限制缩放范围
              if (this.scale < 0.5) this.scale = 0.5;
              if (this.scale > 3.0) this.scale = 3.0;
            })
        )
        // 旋转手势
        .gesture(
          RotationGesture()
            .onActionUpdate((event: GestureEvent) => {
              this.angle += event.angle;
            })
        )
        // 手势组：同时支持捏合+旋转
        .gesture(
          GestureGroup(GestureMode.Parallel,
            PinchGesture(),
            RotationGesture()
          )
        )
    }
  }
}
```

### 8.3 滑动方向手势

```typescript
// SwipeGesture：识别快速滑动手势方向
Swiper() {
  // ...
}
.gesture(
  SwipeGesture(SwipeDirection.Horizontal)  // 水平滑动
    .onAction((event: GestureEvent) => {
      if (event.velocityX > 0) {
        console.info('向右快速滑动');
      } else {
        console.info('向左快速滑动');
      }
    })
)

// SwipeDirection: Horizontal / Vertical / All
```

### 8.4 手势冲突解决策略

| 策略 | 说明 | 适用场景 |
|------|------|----------|
| `GestureMode.Parallel` | 并行执行，所有手势同时识别 | 缩放+旋转同时 |
| `GestureMode.Sequence` | 顺序执行，失败才下一个 | Tap 优先于 Pan |
| `GestureMode.Exclusive` | 互斥执行，第一个成功阻止后续 | 点击 vs 滑动冲突 |

---

## 九、动画系统

### 9.1 animateTo 显式动画

```typescript
// 基础动画：宽高、颜色、位置
animateTo({
  duration: 300,        // 持续时间 ms
  curve: Curve.EaseOut, // 缓动曲线
  iterations: 1,       // 播放次数
  playMode: PlayMode.Normal
}, () => {
  this.targetWidth = 200;
  this.targetHeight = 200;
  this.bgColor = '#007DFF';
})

// Curve 缓动曲线速查
// EaseIn / EaseOut / EaseInOut / Linear
// FastOutSlowIn / SlowOutFastIn / FastOutLinear
// 弹簧曲线：Spring(响应时间, 阻尼)
```

### 9.2 @AnimatableExtend 扩展动画属性

```typescript
// 为自定义组件添加动画属性
@AnimatableExtend(Column)
function animWidth(width: number) {
  .width(width)
}

@State animW: number = 100;

Column() {
  // ...
}
.animWidth(this.animW)

Button('展开').onClick(() => {
  animateTo({ duration: 500 }, () => {
    this.animW = 300;
  });
})
```

### 9.3 路由转场动画

```typescript
// Navigation 页面转场动画
Navigation(this.pathStack) {
  // ...
}
.navDestination(this.RouterPageMap)  // 必须指定路由映射

// routerMap 中配置转场动画
{
  "name": "DetailPage",
  "pageSourceFile": "src/main/ets/pages/DetailPage.ets",
  "buildFunction": "DetailPageBuilder",
  "window": {
    "transitionInEffect": TransitionEffect.OPACITY.animation({ duration: 300 })
  }
}
```

### 9.4 组件转场 animation

```typescript
// 列表项进入动画
List() {
  LazyForEach(this.dataSource, (item: string, index: number) => {
    ListItem() {
      Text(item)
        .opacity(0)
        .translate({ x: 50 })
    }
    .transition(
      TransitionEffect.OPACITY
        .combine(TransitionEffect.translate({ type: TransitionType.Enter, x: 50 }))
    )
    .animation({
      duration: 300,
      delay: index * 50,  // 交错进入
      curve: Curve.EaseOut
    })
  })
}

// 删除动画
ListItem()
  .transition(TransitionEffect.asDelete(TransitionEffect.OPACITY))
```

---

## 十、TextInput / Search 组件

```typescript
// 单行输入框
TextInput({ placeholder: '请输入手机号', text: this.phone })
  .type(InputType.PhoneNumber)     // PhoneNumber / Email / Number / Password / NORMAL
  .enterKeyType(EnterKeyType.Done)
  .onChange((value: string) => {
    this.phone = value;
  })
  .onSubmit((enterKey: EnterKeyType) => {
    console.info('提交');
  })

// 多行输入框
TextInput({ placeholder: '请输入内容' })
  .type(InputType.Normal)
  .maxLines(5)                     // 最大行数
  .height(120)

// Search 搜索框
Search({ placeholder: '搜索商品', value: this.searchText })
  .searchButton('搜索')
  .onSubmit((value: string) => {
    this.doSearch(value);
  })
  .onChange((value: string) => {
    this.searchText = value;
    // 实时搜索建议
  })
```

---

## 十一、Picker 系列组件

```typescript
// 日期选择器
DatePicker({
  start: new Date('2020-01-01'),
  end: new Date('2030-12-31'),
  selected: this.selectedDate
})
  .lunar()                          // 显示农历
  .onDateChange((value: Date) => {
    this.selectedDate = value;
  })

// 时间选择器
TimePicker({ selected: this.selectedTime })
  .useMilitaryTime(true)           // 24小时制
  .onTimeChange((value: TimePickerResult) => {
    this.selectedTime = value;
  })

// 文本选择器
Picker({ range: ['选项1', '选项2', '选项3'], selected: this.pickerIndex })
  .onChange((value: string, index: number) => {
    console.info(`选择: ${value}, 索引: ${index}`);
  })
```

---

## 十二、相对布局 RelativeContainer

```typescript
// RelativeContainer：解决复杂相对定位问题
RelativeContainer() {
  Text('主标题')
    .id('title')
    .alignRules({
      top: { anchor: '__container__', align: VerticalAlign.Top },
      start: { anchor: '__container__', align: HorizontalAlign.Start }
    })
    .padding(16)

  Image($r('app.media.avatar'))
    .id('avatar')
    .alignRules({
      top: { anchor: 'title', align: BottomAlign.Bottom },
      start: { anchor: 'title', align: HorizontalAlign.Start },
      bottom: { anchor: '__container__', align: VerticalAlign.Bottom }
    })
    .width(60).height(60)
    .margin({ top: 8, left: 16 })

  Text('副标题')
    .id('subtitle')
    .alignRules({
      top: { anchor: 'title', align: BottomAlign.Bottom },
      start: { anchor: 'avatar', align: HorizontalAlign.End }
    })
    .margin({ left: 12, top: 8 })
}
.height('100%')
.width('100%')
```

---

## 十三、List 进阶用法

```typescript
// 带分组头部的列表
List({ scroller: this.listScroller, space: 0 }) {
  // 分组头部
  ListItemGroup({ header: this.groupHeader('热门推荐') }) {
    ForEach(this.hotItems, (item: Item) => {
      ListItem() { this.listItemBuilder(item) }
    })
  }

  // 普通项
  ForEach(this.items, (item: Item) => {
    ListItem() { this.listItemBuilder(item) }
  })
}
.sticky(Sticky.Header)              // 分组吸顶
.onScrollIndex((startIndex: number, endIndex: number) => {
  // 可见区域变化回调
})
.onReachStart(() => {
  // 触达顶部
})
.onReachEnd(() => {
  // 触达底部，加载更多
  this.loadMore();
})
```

---

## 十四、WaterFlow 瀑布流

```typescript
WaterFlow() {
  LazyForEach(this.flowDataSource, (item: FlowItem) => {
    FlowItem() {
      Column() {
        Image(item.imageUrl).width('100%')
          .aspectRatio(item.imageRatio)
        Text(item.title).fontSize(12)
          .padding(8)
      }
      .backgroundColor(Color.White)
      .borderRadius(8)
    }
  })
}
.columnsTemplate('1fr 1fr')        // 2列瀑布流
.columnsGap(8)
.rowsGap(8)
.itemConstraintSize({
  minHeight: 100,
  maxHeight: 300
})
.onReachStart(() => {
  this.loadData();
})
```

---

## 十五、SideBarContainer 侧边栏

```typescript
SideBarContainer(SideBarContainerType.Embed) {
  // 侧边内容
  Column() {
    Text('导航菜单')
    List() {
      ForEach(this.menuItems, (item: MenuItem) => {
        ListItem() { Text(item.name) }
      })
    }
  }
  .width('70%')
  .backgroundColor('#F5F5F5')

  // 主内容
  Column() {
    Text('主内容区域')
  }
}
.sideBarWidth(200)                  // 侧边栏宽度
.showSideBar(this.showSidebar)      // 控制显示
.minSideBarWidth(150)
.maxSideBarWidth(300)
.onChange((value: boolean) => {
  // 显示状态变化
})
```

---

## 十六、Badge 徽章 & Marquee 跑马灯

```typescript
// Badge 徽章（数字/圆点/自定义）
Badge({
  count: 99,                        // 99+ 溢出显示
  style: { badgeSize: 16, badgeColor: '#FF4D4F' },
  position: BadgePosition.RightTop  // RightTop / LeftTop / Right / Left
}) {
  Button('消息')
    .size({ width: 80, height: 40 })
}

// 圆点徽章
Badge({
  count: 0,
  style: { badgeSize: 8 },
  position: BadgePosition.RightTop
}) {
  Image($r('app.media.icon'))
}

// Marquee 跑马灯
Marquee({
  start: true,
  src: '欢迎来到鸿蒙世界！这是一条超长的滚动文本内容'
})
.fontSize(16)
.fontColor(Color.Blue)
.scrollAmount(50)                  // 滚动速度


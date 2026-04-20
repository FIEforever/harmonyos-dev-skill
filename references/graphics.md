# HarmonyOS 图形体系专项参考文档

> 覆盖 ArkGraphics 2D、ArkGraphics 3D、AR Engine Kit、Graphics Accelerate Kit、Spatial Recon Kit、XEngine Kit
> 基于官方文档 API 12/HarmonyOS 6 规范，2025-2026 年整理

---

## 一、ArkGraphics 2D（方舟2D图形服务）

### 1.1 两套 API 对比

| 维度 | CanvasRenderingContext2D（ArkTS） | Drawing NDK（C/C++ Native） |
|------|-----------------------------------|-----------------------------|
| 语言 | ArkTS | C/C++ |
| 场景 | UI 内嵌自绘 | 高性能游戏/相机/自定义渲染 |
| 上下文 | Canvas 组件 | XComponent + NativeWindow |
| 特点 | 声明式，简单 | 双缓冲，高性能 |
| 导入 | 内置 | `#include <native_drawing/...>` |

---

### 1.2 CanvasRenderingContext2D（ArkTS 层）

#### 初始化
```typescript
// 声明上下文（建议在 build 外部）
private settings: RenderingContextSettings = new RenderingContextSettings(true);
private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings);

Canvas(this.context)
  .width('100%')
  .height(300)
  .onReady(() => this.draw())
```

#### 路径绘制 API
| API | 说明 |
|-----|------|
| `beginPath()` | 开始新路径 |
| `moveTo(x, y)` | 移动画笔（不绘制） |
| `lineTo(x, y)` | 绘制直线 |
| `arc(x, y, r, sAngle, eAngle, antiClockwise?)` | 绘制圆弧 |
| `arcTo(x1, y1, x2, y2, radius)` | 通过两切线绘制圆弧 |
| `bezierCurveTo(cp1x,cp1y,cp2x,cp2y,x,y)` | 三次贝塞尔曲线 |
| `quadraticCurveTo(cpx, cpy, x, y)` | 二次贝塞尔曲线 |
| `closePath()` | 闭合路径 |
| `stroke()` | 描边绘制 |
| `fill(fillRule?)` | 填充绘制，可选 `"nonzero"/"evenodd"` |
| `clip()` | 裁剪路径 |

#### 矩形 API
```typescript
this.context.fillRect(x, y, width, height);    // 填充矩形
this.context.strokeRect(x, y, width, height);  // 描边矩形
this.context.clearRect(x, y, width, height);   // 清除区域
this.context.roundRect(x, y, w, h, radii);     // 圆角矩形（API 11+）
```

#### 样式设置
```typescript
this.context.fillStyle = '#FF0000';                          // 填充色
this.context.strokeStyle = Color.Blue;                        // 描边色
this.context.lineWidth = 5;                                   // 线宽
this.context.lineCap = 'round';                               // 线帽：butt/round/square
this.context.lineJoin = 'bevel';                              // 连接：miter/round/bevel
this.context.setLineDash([5, 3]);                             // 虚线 [线段长, 间隔长]
this.context.globalAlpha = 0.8;                               // 全局透明度
this.context.globalCompositeOperation = 'source-over';        // 合成模式
this.context.shadowBlur = 10;                                 // 阴影模糊
this.context.shadowColor = 'rgba(0,0,0,0.5)';                // 阴影颜色
this.context.shadowOffsetX = 5; this.context.shadowOffsetY = 5;
```

#### 渐变
```typescript
// 线性渐变
const grad = this.context.createLinearGradient(x0, y0, x1, y1);
grad.addColorStop(0, '#FF0000');
grad.addColorStop(1, '#0000FF');
this.context.fillStyle = grad;

// 径向渐变
const radGrad = this.context.createRadialGradient(x0,y0,r0, x1,y1,r1);
radGrad.addColorStop(0, 'white');
radGrad.addColorStop(1, 'black');
```

#### 图像操作
```typescript
// 异步加载图片
const img = await createImageBitmap($r('app.media.photo'));
this.context.drawImage(img, dx, dy);                      // 基础绘制
this.context.drawImage(img, dx, dy, dw, dh);              // 缩放绘制
this.context.drawImage(img, sx, sy, sw, sh, dx, dy, dw, dh); // 裁剪+缩放

// PixelMap 直接绘制
this.context.drawImage(pixelMap, 0, 0);

// 创建图案
const pattern = this.context.createPattern(img, 'repeat'); // repeat/repeat-x/repeat-y/no-repeat
this.context.fillStyle = pattern!;
```

#### 变换操作
```typescript
this.context.save();             // 保存当前状态（变换+样式）
this.context.restore();          // 恢复上一次保存的状态
this.context.translate(x, y);   // 平移坐标原点
this.context.rotate(angle);      // 旋转（弧度）
this.context.scale(x, y);       // 缩放
this.context.transform(a,b,c,d,e,f); // 自定义变换矩阵
this.context.setTransform(a,b,c,d,e,f); // 重置并设置变换矩阵
```

#### 文本绘制
```typescript
this.context.font = '20px sans-serif';    // 字体设置
this.context.textAlign = 'center';         // 对齐：left/right/center/start/end
this.context.textBaseline = 'middle';      // 基线：top/middle/bottom/alphabetic
this.context.fillText('Hello', x, y, maxWidth?);    // 填充文字
this.context.strokeText('Hello', x, y, maxWidth?);  // 描边文字
const metrics = this.context.measureText('Hello');   // 测量文字宽度
```

#### OffscreenCanvas（离屏画布，API 11+）
```typescript
// 用于预渲染复杂图形，避免主线程阻塞
const offscreen = new OffscreenCanvas(800, 600);
const offCtx = offscreen.getContext('2d') as OffscreenCanvasRenderingContext2D;
offCtx.fillRect(0, 0, 100, 100);

// 将离屏内容渲染到主画布
const bitmap = offscreen.transferToImageBitmap();
this.context.drawImage(bitmap, 0, 0);
```

#### 像素操作
```typescript
const imageData = this.context.getImageData(x, y, w, h); // 获取像素数据
this.context.putImageData(imageData, dx, dy);              // 写回像素数据
const newData = this.context.createImageData(w, h);        // 创建空白像素数据
// imageData.data 是 Uint8ClampedArray，每4字节为 RGBA
```

---

### 1.3 Drawing NDK（C/C++ Native 层）

#### 头文件
```cpp
#include <native_drawing/drawing_canvas.h>
#include <native_drawing/drawing_pen.h>
#include <native_drawing/drawing_brush.h>
#include <native_drawing/drawing_path.h>
#include <native_drawing/drawing_bitmap.h>
#include <native_drawing/drawing_color.h>
#include <native_drawing/drawing_font.h>
#include <native_drawing/drawing_text_typography.h>
#include <native_window/external_window.h>
```

#### 核心对象 API
```cpp
// ===== Bitmap（位图）=====
OH_Drawing_Bitmap* bm = OH_Drawing_BitmapCreate();
OH_Drawing_BitmapFormat fmt {COLOR_FORMAT_RGBA_8888, ALPHA_FORMAT_OPAQUE};
OH_Drawing_BitmapBuild(bm, width, height, &fmt);
void* pixels = OH_Drawing_BitmapGetPixels(bm);         // 获取像素地址

// ===== Canvas（画布）=====
OH_Drawing_Canvas* canvas = OH_Drawing_CanvasCreate();
OH_Drawing_CanvasBind(canvas, bm);                      // 绑定位图
OH_Drawing_CanvasClear(canvas, OH_Drawing_ColorSetArgb(0xFF,0xFF,0xFF,0xFF));
OH_Drawing_CanvasAttachPen(canvas, pen);                // 附画笔（描边）
OH_Drawing_CanvasAttachBrush(canvas, brush);            // 附画刷（填充）
OH_Drawing_CanvasDetachPen(canvas);                     // 取消画笔
OH_Drawing_CanvasDetachBrush(canvas);                   // 取消画刷
OH_Drawing_CanvasDrawPath(canvas, path);                // 绘制路径
OH_Drawing_CanvasDrawRect(canvas, rect);                // 绘制矩形
OH_Drawing_CanvasDrawCircle(canvas, cx, cy, radius);    // 绘制圆
OH_Drawing_CanvasSave(canvas);                          // 保存状态
OH_Drawing_CanvasRestore(canvas);                       // 恢复状态
OH_Drawing_CanvasRotate(canvas, degrees, px, py);       // 旋转
OH_Drawing_CanvasTranslate(canvas, dx, dy);             // 平移
OH_Drawing_CanvasScale(canvas, sx, sy);                 // 缩放

// ===== Pen（画笔 = 描边）=====
OH_Drawing_Pen* pen = OH_Drawing_PenCreate();
OH_Drawing_PenSetColor(pen, OH_Drawing_ColorSetArgb(A,R,G,B));
OH_Drawing_PenSetWidth(pen, 5.0f);
OH_Drawing_PenSetAntiAlias(pen, true);
OH_Drawing_PenSetCap(pen, LINE_ROUND_CAP);      // FLAT/SQUARE/ROUND
OH_Drawing_PenSetJoin(pen, LINE_ROUND_JOIN);    // MITER/ROUND/BEVEL

// ===== Brush（画刷 = 填充）=====
OH_Drawing_Brush* brush = OH_Drawing_BrushCreate();
OH_Drawing_BrushSetColor(brush, OH_Drawing_ColorSetArgb(A,R,G,B));
OH_Drawing_BrushSetAntiAlias(brush, true);

// ===== Path（路径）=====
OH_Drawing_Path* path = OH_Drawing_PathCreate();
OH_Drawing_PathMoveTo(path, x, y);
OH_Drawing_PathLineTo(path, x, y);
OH_Drawing_PathArcTo(path, x1,y1,x2,y2, startDeg, sweepDeg);
OH_Drawing_PathCubicTo(path, cp1x,cp1y, cp2x,cp2y, endx,endy);  // 三次贝塞尔
OH_Drawing_PathClose(path);
OH_Drawing_PathReset(path);

// ===== 销毁（必须手动释放）=====
OH_Drawing_CanvasDestroy(canvas);
OH_Drawing_BitmapDestroy(bm);
OH_Drawing_PenDestroy(pen);
OH_Drawing_BrushDestroy(brush);
OH_Drawing_PathDestroy(path);
```

#### 文本绘制（Typography 方案）
```cpp
// 1. 创建排版风格
OH_Drawing_TypographyStyle* typoStyle = OH_Drawing_CreateTypographyStyle();
OH_Drawing_SetTypographyTextDirection(typoStyle, TEXT_DIRECTION_LTR);
OH_Drawing_SetTypographyTextAlign(typoStyle, TEXT_ALIGN_LEFT);

// 2. 创建文本风格
OH_Drawing_TextStyle* textStyle = OH_Drawing_CreateTextStyle();
OH_Drawing_SetTextStyleColor(textStyle, OH_Drawing_ColorSetArgb(0xFF,0,0,0));
OH_Drawing_SetTextStyleFontSize(textStyle, 24);
OH_Drawing_SetTextStyleFontWeight(textStyle, FONT_WEIGHT_400);
const char* families[] = {"HarmonyOS Sans"};
OH_Drawing_SetTextStyleFontFamilies(textStyle, 1, families);

// 3. 构建并绘制
OH_Drawing_FontCollection* fc = OH_Drawing_CreateFontCollection();
OH_Drawing_TypographyCreate* handler = OH_Drawing_CreateTypographyHandler(typoStyle, fc);
OH_Drawing_TypographyHandlerPushTextStyle(handler, textStyle);
OH_Drawing_TypographyHandlerAddText(handler, "Hello HarmonyOS");
OH_Drawing_TypographyHandlerPopTextStyle(handler);
OH_Drawing_Typography* typo = OH_Drawing_CreateTypography(handler);
OH_Drawing_TypographyLayout(typo, 800.0f);
OH_Drawing_TypographyPaint(typo, canvas, 10.0, 50.0);

// 4. 清理
OH_Drawing_DestroyTypography(typo);
OH_Drawing_DestroyTypographyHandler(handler);
OH_Drawing_DestroyTextStyle(textStyle);
OH_Drawing_DestroyTypographyStyle(typoStyle);
OH_Drawing_DestroyFontCollection(fc);
```

#### 双缓冲送显流程（XComponent 场景）
```cpp
// 回调：OnSurfaceCreated 中获取 nativeWindow
OH_NativeWindow* nativeWindow = ...; // 从 XComponent surface 获取

// 每帧渲染流程
OHNativeWindowBuffer* buffer = nullptr;
int fenceFd = -1;
OH_NativeWindow_NativeWindowRequestBuffer(nativeWindow, &buffer, &fenceFd);
BufferHandle* handle = OH_NativeWindow_GetBufferHandleFromNative(buffer);
void* mapped = mmap(handle->virAddr, handle->size, PROT_READ|PROT_WRITE, MAP_SHARED, handle->fd, 0);

// 将 Bitmap 像素拷贝到 buffer
void* bitmapPixels = OH_Drawing_BitmapGetPixels(bitmap);
memcpy(mapped, bitmapPixels, handle->size);
munmap(mapped, handle->size);

Region region{nullptr, 0};
OH_NativeWindow_NativeWindowFlushBuffer(nativeWindow, buffer, fenceFd, region);
```

---

## 二、ArkGraphics 3D（方舟3D图形）

### 2.1 简介

基于轻量级3D引擎和渲染管线，提供基础3D场景绘制能力。支持**自定义场景模式**（开发者完全控制相机/光源/节点）和**自动场景模式**（框架自动创建基础相机和光照）。

**导入方式：**
```typescript
import { Component3D, Scene, SceneNode } from '@kit.ArkGraphics3D';
// 或旧式：import { Component3D, Scene } from '@ohos.arkgraphics_3d';
```

### 2.2 核心概念

| 概念 | 说明 |
|------|------|
| **Scene** | 3D 场景容器，顶级管理对象 |
| **SceneNode** | 场景节点，所有3D对象的基础单元，构成树状层级 |
| **Mesh** | 网格，定义物体几何形状 |
| **Material** | 材质，定义物体光学属性（颜色/纹理/粗糙度/金属度） |
| **Camera** | 虚拟相机，决定观察视角和投影方式 |
| **Light** | 光源（平行光/点光源/聚光灯） |
| **Environment** | 场景环境背景（基于图片的环境贴图） |
| **Animation** | 关键帧/骨骼驱动动画 |
| **Component3D** | ArkUI 中的3D组件，将3D场景嵌入2D界面 |

### 2.3 Component3D 组件（ArkUI 嵌入3D）

```typescript
@Entry
@Component
struct My3DPage {
  scene: Scene | null = null;
  
  build() {
    Column() {
      Component3D({ scene: this.scene })
        .width('100%')
        .height(400)
        .onReady((scene: Scene) => {
          this.scene = scene;
          this.initScene(scene);
        })
    }
  }
  
  async initScene(scene: Scene) {
    // 加载 GLB 模型（自动场景模式）
    await scene.load('/resources/models/wukong.glb');
    
    // 或手动配置（自定义场景模式）
    const cameraNode = scene.createNode('camera');
    const camera = cameraNode.addComponent('Camera');
    camera.fov = 60; // 视角
    camera.nearPlane = 0.1;
    camera.farPlane = 1000;
    cameraNode.position = { x: 0, y: 2, z: 5 };
    scene.addNode(cameraNode);
  }
}
```

### 2.4 场景节点管理

```typescript
// 创建节点
const node = scene.createNode('myNode');
node.position = { x: 1, y: 0, z: 0 };    // 位置
node.rotation = { x: 0, y: 45, z: 0 };    // 旋转（欧拉角）
node.scale = { x: 1, y: 1, z: 1 };        // 缩放

// 添加子节点（层级结构）
parentNode.addChild(childNode);

// 添加组件
const meshRenderer = node.addComponent('MeshRenderer');
meshRenderer.mesh = mesh;
meshRenderer.material = material;
```

### 2.5 模型加载

```typescript
// 方式1：整体加载 GLB 文件（推荐，含模型+材质+动画）
await scene.load('resources/rawfile/model.glb');

// 方式2：资源管理器加载
const resourceManager = getContext(this).resourceManager;
const data = resourceManager.getRawFileDescriptor('models/product.glb');
await scene.loadFromFd(data.fd, data.offset, data.length);

// 支持格式：GLB（推荐）、GLTF
// 注意：模型格式推荐转换为 .glb 获得最佳性能
```

### 2.6 相机配置

```typescript
// 创建透视相机
const camNode = scene.createNode('mainCamera');
const cam = camNode.addComponent('Camera') as CameraComponent;
cam.fov = 60;              // 视场角（度）
cam.nearPlane = 0.1;       // 近裁剪面
cam.farPlane = 500;        // 远裁剪面
cam.projectionType = 'perspective'; // 透视投影（默认）
// cam.projectionType = 'orthographic'; // 正交投影

// 相机位姿
camNode.position = { x: 0, y: 1.5, z: 5 };
camNode.lookAt({ x: 0, y: 0, z: 0 }); // 朝向目标点
```

### 2.7 光照配置

```typescript
// 平行光（DirectionalLight）
const dirLightNode = scene.createNode('dirLight');
const dirLight = dirLightNode.addComponent('DirectionalLight') as LightComponent;
dirLight.color = { r: 1, g: 1, b: 1 };   // 白光
dirLight.intensity = 1.0;
dirLightNode.rotation = { x: -45, y: 45, z: 0 };

// 点光源（PointLight）
const pointLightNode = scene.createNode('pointLight');
const pointLight = pointLightNode.addComponent('PointLight') as LightComponent;
pointLight.color = { r: 1, g: 0.8, b: 0.6 };
pointLight.intensity = 2.0;
pointLight.range = 10;                      // 影响范围
pointLightNode.position = { x: 2, y: 3, z: 2 };
```

### 2.8 材质配置

```typescript
// PBR 材质（基于物理的渲染）
const material = scene.createMaterial('myMaterial');
material.baseColorFactor = { r: 0.9, g: 0.7, b: 0.2, a: 1.0 }; // 基础色（金色）
material.metallicFactor = 0.9;     // 金属度
material.roughnessFactor = 0.1;    // 粗糙度（越小越光滑）
material.emissiveFactor = { r: 0, g: 0, b: 0 }; // 自发光（默认关闭）

// 设置纹理贴图
const texture = await scene.loadTexture('resources/rawfile/texture.png');
material.baseColorTexture = texture;
material.normalTexture = normalTexture; // 法线贴图
material.metallicRoughnessTexture = mrTexture;
```

### 2.9 动画控制

```typescript
// 加载含动画的 GLB 后自动获取动画列表
const animations = scene.getAnimations(); // AnimationClip[]
const anim = animations[0];

// 播放动画
anim.play();
anim.stop();
anim.pause();
anim.setTime(0.5);       // 跳转到指定时间（秒）
anim.setSpeed(2.0);      // 播放速度倍率
anim.setLoop(true);      // 是否循环

// 混合权重
anim.weight = 1.0;       // 0.0~1.0
```

### 2.10 性能优化

```typescript
// LOD（细节层次）：根据距离切换不同精度模型
node.setLOD(lodLevel); // 0=最高精度

// 实例化渲染（大量相同模型）
node.setInstanced(true);

// 帧率控制
scene.setTargetFrameRate(30); // 平衡性能与功耗

// 渲染模式
scene.setRenderMode('onDemand'); // 按需渲染（无动画时节省CPU）
```

---

## 三、AR Engine Kit（AR 引擎服务）

### 3.1 核心能力

| 能力 | 说明 |
|------|------|
| 运动跟踪（6DoF） | 实时追踪设备在三维空间中的位置和姿态 |
| 平面检测 | 检测水平/垂直平面（桌面/墙面） |
| 平面语义识别 | 识别平面类型（地板/桌面/墙/天花板） |
| 深度估计 | 生成场景深度图 |
| 环境 Mesh | 生成真实环境的3D网格（用于遮挡计算） |
| 图像跟踪 | 识别并跟踪预设图像目标 |
| AR 物体摆放 | 命中检测 + 虚拟物体放置 |
| 高精几何重建 | 稠密点云 + 立方体数据 |

**导入：**
```typescript
import arEngine from '@kit.AREngineKit';
// 旧式：import arEngine from '@ohos.arEngine';
```

### 3.2 权限声明（module.json5）
```json
{
  "requestPermissions": [
    { "name": "ohos.permission.CAMERA" }
  ]
}
```

### 3.3 核心开发流程（ArkTS）

```typescript
// 1. 创建 AR 会话
const session = arEngine.createARSession();

// 2. 配置会话（世界跟踪模式）
const config = arEngine.createARWorldTrackingConfig(session);
config.setEnablePlaneDetection(true);   // 启用平面检测
config.setEnableDepth(true);            // 启用深度估计
session.configure(config);

// 3. 注册 XComponent Surface（渲染输出）
// 在 XComponent 的 onSurfaceCreated 回调中：
session.setSurface(surface); // 绑定相机预览 Surface

// 4. 启动会话
session.resume();

// 5. 每帧更新（在渲染循环中）
const frame = session.update();
if (!frame) return;

// 6. 获取相机位姿
const cameraPose = frame.getCamera().getPose();
const position = cameraPose.getTranslation(); // [x, y, z]
const rotation = cameraPose.getRotationQuaternion(); // [qx, qy, qz, qw]

// 7. 获取检测到的平面
const planes = frame.getUpdatedTrackables(arEngine.ARTrackableType.PLANE);
for (const trackable of planes) {
  const plane = trackable as arEngine.ARPlane;
  if (plane.getTrackingState() === arEngine.TrackingState.TRACKING) {
    const center = plane.getCenterPose().getTranslation();
    const extent = plane.getExtent(); // [半宽, 半高]
    const type = plane.getPlaneType(); // HORIZONTAL_DOWNWARD/HORIZONTAL_UPWARD/VERTICAL
  }
}

// 8. 命中检测（AR 物体摆放）
function onScreenTap(screenX: number, screenY: number) {
  const hitResults = frame.hitTest(screenX, screenY);
  if (hitResults.length > 0) {
    const hitPose = hitResults[0].getHitPose();
    const hitPosition = hitPose.getTranslation();
    // 在 hitPosition 处放置虚拟模型
    placeVirtualObject(hitPosition);
  }
}

// 9. 暂停/恢复/销毁
session.pause();
session.resume();
session.close(); // 释放资源
```

### 3.4 深度估计

```typescript
const depthImage = frame.acquireDepthImage16Bits(); // 获取16位深度图
// 每个像素值为 毫米级深度值（uint16）
const width = depthImage.width;
const height = depthImage.height;
const buffer = depthImage.buffer; // ArrayBuffer
depthImage.release(); // 用完必须释放
```

### 3.5 图像跟踪

```typescript
// 添加图像追踪数据库
const imageDatabase = arEngine.createARImageDatabase(session);
imageDatabase.addImageWithPhysicalSize('poster', $r('app.media.target_image'), 0.3); // 宽度0.3米

const config = arEngine.createARImageTrackingConfig(session);
config.setImageDatabase(imageDatabase);
session.configure(config);

// 每帧获取跟踪到的图像
const images = frame.getUpdatedTrackables(arEngine.ARTrackableType.AUGMENTED_IMAGE);
for (const img of images) {
  const augImg = img as arEngine.ARAugmentedImage;
  if (augImg.getTrackingState() === arEngine.TrackingState.TRACKING) {
    const name = augImg.getName();    // 对应注册时的名称
    const pose = augImg.getCenterPose();
  }
}
```

### 3.6 环境 Mesh（防穿模）

```typescript
// 启用环境 Mesh
const config = arEngine.createARWorldTrackingConfig(session);
config.setEnvMeshEnabled(true);
session.configure(config);

// 每帧获取 Mesh 数据
const meshes = frame.getUpdatedTrackables(arEngine.ARTrackableType.ENVIRONMENT_MESH);
for (const mesh of meshes) {
  const envMesh = mesh as arEngine.AREnvironmentMesh;
  const vertices = envMesh.getVertices();  // Float32Array [x,y,z,x,y,z,...]
  const indices = envMesh.getTriangleIndices(); // Int32Array
  // 用顶点+索引构建遮挡 Mesh，在渲染时遮挡被真实环境挡住的虚拟物体
}
```

---

## 四、Graphics Accelerate Kit（图形加速服务）

### 4.1 三大核心服务

| 服务 | 功能 | 适用场景 |
|------|------|----------|
| **游戏渲染加速** | 超帧（MEMC插帧）、ABR自适应稳态渲染、OpenGTX帧率/SOC调度 | 重度游戏/AR/VR |
| **游戏资源加速** | 资源包后台静默下载、预加载 | 游戏关卡资源管理 |
| **游戏启动加速** | 内存镜像精准恢复，秒级冷启动 | 游戏/重型应用 |

> **限制**：
> - 仅支持中国境内（不含港澳台）
> - 仅支持 Phone / Tablet
> - 不支持模拟器
> - 硬件要求：马良 910 GPU 及以上

### 4.2 渲染加速技术详解

| 技术 | 原理 | 效果 |
|------|------|------|
| **超帧** | MEMC 运动估计+补偿，在真实帧间插入预测帧 | 提升帧率（如30→60fps），降低功耗 |
| **ABR** | 感知设备状态，自适应调整 FrameBuffer 分辨率 | 更稳定帧率，降低功耗 |
| **OpenGTX** | 感知渲染场景，自适应调整帧率和 SOC/DDR 频率 | 降低发热，延长续航 |

### 4.3 API 集成（ArkTS）

```typescript
import graphicsAccelerate from '@ohos.graphicsAccelerate';

// 1. 创建加速上下文
const accelerator = graphicsAccelerate.createContext({
  type: 'OPENGL_ES3',        // 后端类型：OPENGL_ES3 / VULKAN
  enableProfiling: false      // 是否开启性能分析
});

// 2. 能力检测（重要！不同设备能力不同）
const caps = accelerator.getCapabilities();
if (!caps.supportFloatTextures) {
  pipeline.setPrecision('mediump'); // 降级到16位精度
}

// 3. 创建纹理
const texture = accelerator.createTexture({
  width: 1920,
  height: 1080,
  format: 'RGBA8'  // 支持 RGBA8/NV21/NV12
});

// 4. 构建计算管线
const pipeline = accelerator.createComputePipeline({
  shaders: {
    vertex: 'shaders/base.vert',
    fragment: 'shaders/effect.frag'
  }
});
pipeline.setUniform('params', {
  smoothStrength: 0.8,
  whitenFactor: 0.6
});

// 5. 执行渲染（GPU 计算）
accelerator.dispatchCompute(pipeline, {
  inputTextures: [inputTexture],
  outputTexture: outputTexture
});
```

### 4.4 纹理池优化（高频帧场景）

```typescript
// 创建纹理池，复用纹理对象
const texturePool = new graphicsAccelerate.TexturePool(accelerator, {
  maxRetainTime: 3000,          // 最大保留时间（ms）
  defaultSize: [1920, 1080]
});

// 异步上传（相机帧场景）
cameraFrame.on('newFrame', (frame) => {
  texturePool.uploadAsync(frame.pixels, { format: 'NV21' });
});
```

### 4.5 动态降级（防过热）

```typescript
// 温度监控 + 自动降级
import thermal from '@ohos.thermal';

thermal.subscribeThermalLevel((level) => {
  if (level >= thermal.ThermalLevel.HOT) {
    // 降低 workgroup 密度，减少 GPU 负载
    pipeline.setWorkgroupSize([16, 16]);
    // 或降级 ABR 分辨率缩放因子
  }
});
```

### 4.6 Worker 线程离屏渲染

```typescript
// main thread
const worker = new worker.ThreadWorker('workers/renderWorker.ets');
worker.postMessage({ sharedGL: accelerator.getSharedContext() });

// renderWorker.ets
workerPort.onMessage = (e: MessageEvents) => {
  const workerCtx = graphicsAccelerate.createWorkerContext(e.data.sharedGL);
  // 在 Worker 中执行重型 GPU 计算，不阻塞主线程
};
```

---

## 五、XEngine Kit（GPU 加速引擎服务）

### 5.1 定位与能力

XEngine Kit 是专为鸿蒙优化的跨平台渲染引擎，基于**马良 GPU** 提供：

| 能力 | 说明 |
|------|------|
| **空域 GPU 超分** | 单帧图像超采样，开销最低 |
| **空域 AI 超分** | GPU/NPU 协同超采样，效果更佳 |
| **自适应 VRS** | 动态调整不同区域着色率，降低带宽 |
| **Subpass Shading** | 适用 TBDR/Forward+ 管线，降低带宽 |
| **多后端自动适配** | 支持 OpenGL ES 3.x / Vulkan / Metal |
| **跨平台** | Android / iOS / HarmonyOS 一套代码 |

> **硬件要求**：马良 910 GPU 及以上

### 5.2 初始化

```typescript
import { XEngine, BackendType } from '@kit.XEngineKit';

// 创建引擎（自动选择最优后端）
const engine = new XEngine({
  backendPreference: [BackendType.VULKAN, BackendType.GLES3, BackendType.GLES2],
  debug: false,            // 生产环境关闭 debug
  enableVRS: true          // 启用自适应 VRS
});
```

### 5.3 渲染管线（着色器）

```typescript
// 创建着色器程序
const vertexShader = `
  precision mediump float;
  attribute vec4 aPosition;
  attribute vec2 aTexCoord;
  varying vec2 vTexCoord;
  void main() {
    gl_Position = aPosition;
    vTexCoord = aTexCoord;
  }
`;

const fragmentShader = `
  precision mediump float;
  varying vec2 vTexCoord;
  uniform sampler2D uTexture;
  uniform float smoothLevel; // 磨皮参数
  void main() {
    vec4 color = texture2D(uTexture, vTexCoord);
    // 磨皮算法
    float mask = smoothstep(0.3, 0.7, color.r);
    gl_FragColor = mix(color, vec4(1.0), mask * smoothLevel);
  }
`;

const program = engine.createProgram(vertexShader, fragmentShader);
program.setUniform('smoothLevel', 0.5);
```

### 5.4 纹理处理

```typescript
// 创建输入/输出纹理
const inputTex = engine.createTexture({
  width: 1920,
  height: 1080,
  format: engine.TextureFormat.RGBA8,
  wrapS: 'clamp_to_edge',
  wrapT: 'clamp_to_edge',
  minFilter: 'linear',
  magFilter: 'linear'
});

// ALIGN4 对齐（NV21/NV12 格式注意）
const alignedWidth = Math.ceil(width / 4) * 4;

// 上传数据
inputTex.upload(pixelData);
const outputTex = engine.createTexture({ width, height, format: engine.TextureFormat.RGBA8 });
```

### 5.5 执行渲染

```typescript
// 帧缓冲对象
const fbo = engine.createFramebuffer();
fbo.attach(outputTex);

engine.render({
  program,
  inputTextures: { uTexture: inputTex },
  framebuffer: fbo,
  viewport: { x: 0, y: 0, width, height }
});

// 读取结果
const resultPixels = outputTex.readPixels(); // Uint8Array
```

### 5.6 Worker 多线程支持

```typescript
// 支持在 Worker 线程初始化 XEngine（防主线程阻塞）
const renderWorker = new worker.ThreadWorker('workers/renderWorker.ets');

// 传递共享上下文
renderWorker.postMessage({
  sharedContext: engine.getSharedContext(),
  width: 1920,
  height: 1080
});
```

### 5.7 温度自动降级

```typescript
import thermal from '@ohos.thermal';

thermal.subscribeThermalLevel((level) => {
  if (level >= thermal.ThermalLevel.HOT) {
    // 超过温度阈值，降级到 GLES2
    engine.switchBackend(BackendType.GLES2);
    // 或降低渲染分辨率
    renderScale = 0.75;
  }
});
```

---

## 六、Spatial Recon Kit（空间建模服务）

### 6.1 定位

提供 **3DGS（3D Gaussian Splatting）** 模型的端侧渲染与运算能力，用于将现实场景转换为高质量3D模型。

### 6.2 支持的模型格式

| 格式 | 说明 |
|------|------|
| **MP4** | 视频格式的3DGS模型 |
| **PLY** | 点云/高斯泼溅原始格式 |
| **GLB** | 打包的3D模型（与 ArkGraphics 3D 通用） |

### 6.3 使用限制

- **必须与 ArkGraphics 3D 联合使用**（作为其扩展模块）
- 支持设备：Phone / Tablet / PC / TV
- 仅支持中国境内（不含港澳台）

### 6.4 3DGS 模型加载

```typescript
import spatialRecon from '@kit.SpatialReconKit';
import { Scene } from '@kit.ArkGraphics3D';

// 在 ArkGraphics 3D 场景中加载 3DGS 模型
const scene: Scene = ...; // 已初始化的 Scene

// 加载 PLY 格式
const gaussianModel = await spatialRecon.loadGaussianModel({
  scene: scene,
  uri: 'resources/rawfile/scene.ply',
  format: spatialRecon.ModelFormat.PLY
});

// 加载 GLB 格式
const gaussianModel2 = await spatialRecon.loadGaussianModel({
  scene: scene,
  uri: 'resources/rawfile/scene.glb',
  format: spatialRecon.ModelFormat.GLB
});

// 加载 MP4 格式（视频序列）
const gaussianModel3 = await spatialRecon.loadGaussianModel({
  scene: scene,
  uri: 'resources/rawfile/scene.mp4',
  format: spatialRecon.ModelFormat.MP4
});
```

### 6.5 滤镜处理

```typescript
// 对3DGS渲染图像应用风格化滤镜
const filter = spatialRecon.createFilter({
  type: spatialRecon.FilterType.STYLE_TRANSFER,
  strength: 0.8
});
gaussianModel.applyFilter(filter);
```

### 6.6 典型应用场景

- **商品3D展示**：用手机扫描商品生成3D模型，在购物应用中360°展示
- **空间记忆**：用视频重建室内3D场景（如 Remy App）
- **AR 叠加**：将3DGS模型与 AR Engine 结合，实现真实环境中的3D内容
- **虚拟展览**：博物馆/展厅的3D数字化

---

## 七、图形能力对比速查

| Kit | 核心用途 | ArkTS API | Native API | 硬件要求 |
|-----|---------|-----------|------------|----------|
| **ArkGraphics 2D** | 2D矢量绘图、文字、图片 | CanvasRenderingContext2D | OH_Drawing_* | 所有设备 |
| **ArkGraphics 3D** | 3D模型渲染、场景搭建 | Component3D / Scene | 无 | 所有设备 |
| **AR Engine** | AR增强现实、平面检测、运动跟踪 | arEngine.* | HMS_AREngine_* (C) | 支持AR的设备 |
| **Graphics Accelerate** | 游戏/相机GPU加速、超帧、启动加速 | graphicsAccelerate.* | 无 | 马良910+ |
| **XEngine** | 跨平台高性能GPU渲染、着色器 | XEngine | OpenGL ES/Vulkan | 马良910+ |
| **Spatial Recon** | 3DGS三维重建模型加载渲染 | spatialRecon.* | 无 | Phone/Tablet |

---

## 八、图形开发常见问题

### Q1：Canvas 2D vs ArkGraphics 2D Drawing，如何选择？
- Canvas 2D（CanvasRenderingContext2D）：适合 UI 内嵌自绘，简单图形、图表、游戏画布，ArkTS 直接调用
- Drawing NDK：适合高性能场景（相机滤镜、游戏渲染），通过 XComponent 接入，C/C++ 实现，支持双缓冲

### Q2：ArkGraphics 3D 支持哪些3D模型格式？
目前主要支持 **GLB/GLTF**（glTF 2.0规范），推荐使用 .glb（二进制格式，性能最优）。

### Q3：AR Engine 需要哪些权限？
最少需要 `ohos.permission.CAMERA`，部分功能（深度图）可能需要额外传感器权限。

### Q4：Graphics Accelerate Kit 和 XEngine Kit 区别？
- **Graphics Accelerate Kit**：系统级加速方案，聚焦游戏渲染加速（超帧/ABR/OpenGTX）和游戏资源/启动加速，偏向系统调度
- **XEngine Kit**：开发者层跨平台渲染引擎，偏向着色器编程和多后端（OpenGL ES/Vulkan），适合自定义渲染管线

### Q5：Spatial Recon Kit 与 ArkGraphics 3D 是什么关系？
Spatial Recon Kit 是 ArkGraphics 3D 的扩展，专门处理 3DGS 格式模型的加载和渲染，**必须与 ArkGraphics 3D 联合使用**。

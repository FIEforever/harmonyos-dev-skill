# HarmonyOS Kit API 参考（API 12 官方规范）

> 所有导入均使用新式 `@kit.XxxKit` 写法（API 12+ 标准）

---

## 一、AbilityKit（Ability 管理）

```typescript
import { common, Want, UIAbility, AbilityConstant } from '@kit.AbilityKit';

// 启动另一个 Ability（同模块）
const want: Want = {
  bundleName: 'com.example.myapp',
  abilityName: 'SecondAbility',
  parameters: {
    userId: '123',
    source: 'MainPage'
  }
};
context.startAbility(want).catch((err: Error) => {
  console.error(`启动失败: ${err.message}`);
});

// 启动隐式 Want（跳转系统页面）
context.startAbility({
  action: 'ohos.want.action.SEND',
  type: 'text/plain',
  parameters: { 'ohos.extra.TEXT': '分享内容' }
});

// 结束当前 Ability
context.terminateSelf();

// 结束并返回结果（被 startAbilityForResult 拉起时）
context.terminateSelfWithResult({
  resultCode: 0,
  want: { parameters: { result: '操作成功' } }
});
```

---

## 二、NetworkKit（HTTP 网络请求）

### 2.1 基础用法

```typescript
import { http } from '@kit.NetworkKit';

// RequestMethod 枚举
// http.RequestMethod.GET / POST / PUT / DELETE / HEAD / OPTIONS / TRACE / CONNECT

const httpReq = http.createHttp();

try {
  const response = await httpReq.request('https://api.example.com/data', {
    method: http.RequestMethod.GET,
    header: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer token123'
    },
    connectTimeout: 10000,
    readTimeout: 30000,
  });

  if (response.responseCode === http.ResponseCode.OK) {     // 200
    const data = JSON.parse(response.result as string);
    // 处理数据
  }
} catch (err) {
  console.error(`请求失败: ${JSON.stringify(err)}`);
} finally {
  httpReq.destroy();  // ⚠️ 必须释放，否则内存泄漏
}
```

### 2.2 POST 请求（JSON Body）

```typescript
const response = await httpReq.request('https://api.example.com/users', {
  method: http.RequestMethod.POST,
  header: { 'Content-Type': 'application/json' },
  extraData: JSON.stringify({    // ⚠️ POST body 用 extraData，需手动 JSON.stringify
    name: '张三',
    age: 25
  }),
  connectTimeout: 10000,
  readTimeout: 30000
});
```

### 2.3 封装工具类

```typescript
import { http } from '@kit.NetworkKit';
import { hilog } from '@kit.PerformanceAnalysisKit';

const BASE_URL = 'https://api.example.com';
const TAG = 'HttpUtil';

async function request<T>(
  path: string,
  method: http.RequestMethod = http.RequestMethod.GET,
  body?: object,
  extraHeaders?: Record<string, string>
): Promise<T> {
  const req = http.createHttp();
  try {
    const res = await req.request(`${BASE_URL}${path}`, {
      method,
      header: {
        'Content-Type': 'application/json',
        ...extraHeaders
      },
      extraData: body ? JSON.stringify(body) : undefined,
      connectTimeout: 10000,
      readTimeout: 30000
    });

    if (res.responseCode < 200 || res.responseCode >= 300) {
      throw new Error(`HTTP ${res.responseCode}`);
    }

    return JSON.parse(res.result as string) as T;
  } catch (err) {
    hilog.error(0x0000, TAG, `[${method}] ${path} 失败: ${JSON.stringify(err)}`);
    throw err;
  } finally {
    req.destroy();
  }
}

// 使用
const users = await request<User[]>('/users');
const newUser = await request<User>('/users', http.RequestMethod.POST, { name: '李四' });
```

---

## 三、ArkData（数据持久化）

### 3.1 Preferences（用户首选项）

```typescript
import { preferences } from '@kit.ArkData';

// API 12：第二个参数是配置对象（旧版是字符串）
const pref = preferences.getPreferencesSync(context, { name: 'settings.db' });

// 写入（必须 flush 刷盘）
pref.putSync('theme', 'dark');
pref.putSync('fontSize', 16);
pref.putSync('notifications', true);
await pref.flush();

// 读取（第二个参数为默认值）
const theme = pref.getSync('theme', 'light') as string;
const fontSize = pref.getSync('fontSize', 14) as number;
const notify = pref.getSync('notifications', true) as boolean;

// 删除
pref.deleteSync('theme');
await pref.flush();

// 检查是否存在
const exists = pref.hasSync('theme');
```

### 3.2 RelationalStore（关系型数据库）

```typescript
import { relationalStore, ValuesBucket } from '@kit.ArkData';

// 1. 初始化（建议封装单例）
const store = await relationalStore.getRdbStore(context, {
  name: 'app.db',
  securityLevel: relationalStore.SecurityLevel.S1
});

// 2. 建表
await store.executeSql(`
  CREATE TABLE IF NOT EXISTS jewelry (
    id        INTEGER PRIMARY KEY AUTOINCREMENT,
    name      TEXT NOT NULL,
    category  TEXT NOT NULL,
    price     REAL NOT NULL DEFAULT 0,
    image_url TEXT,
    created_at INTEGER
  )
`);

// 3. 插入
store.insertSync('jewelry', {
  name: '18K 钻石项链',
  category: 'necklace',
  price: 3999.00,
  image_url: 'https://cdn.example.com/img/001.jpg',
  created_at: Date.now()
} as ValuesBucket);

// 4. 查询（使用 RdbPredicates）
const pred = new relationalStore.RdbPredicates('jewelry');
pred.equalTo('category', 'necklace')
  .greaterThan('price', 1000)
  .orderByAsc('price')
  .limitAs(20)
  .offsetAs(0);

const result = store.querySync(pred, ['id', 'name', 'price', 'image_url']);
const items: JewelryItem[] = [];
while (result.goToNextRow()) {
  items.push({
    id: result.getLong(result.getColumnIndex('id')),
    name: result.getString(result.getColumnIndex('name')),
    price: result.getDouble(result.getColumnIndex('price')),
    imageUrl: result.getString(result.getColumnIndex('image_url'))
  });
}
result.close();  // ⚠️ 必须关闭，否则资源泄漏

// 5. 更新
const updatePred = new relationalStore.RdbPredicates('jewelry');
updatePred.equalTo('id', 1);
store.updateSync({ price: 3599.00 } as ValuesBucket, updatePred);

// 6. 删除
const deletePred = new relationalStore.RdbPredicates('jewelry');
deletePred.equalTo('id', 1);
store.deleteSync(deletePred);

// 7. 事务
store.beginTransaction();
try {
  store.insertSync('jewelry', { /* ... */ } as ValuesBucket);
  store.insertSync('jewelry', { /* ... */ } as ValuesBucket);
  store.commit();
} catch (err) {
  store.rollBack();
  throw err;
}
```

---

## 四、ImageKit（图像处理）

```typescript
import { image } from '@kit.ImageKit';
import { fileIo } from '@kit.CoreFileKit';

// 从文件路径加载 PixelMap
async function loadPixelMap(uri: string): Promise<image.PixelMap> {
  const file = await fileIo.open(uri, fileIo.OpenMode.READ_ONLY);
  const imageSource = image.createImageSource(file.fd);
  const pixelMap = await imageSource.createPixelMap({
    editable: true,
    desiredPixelFormat: image.PixelMapFormat.RGBA_8888
  });
  await fileIo.close(file);
  imageSource.release();
  return pixelMap;
}

// 创建纯色 PixelMap（常用于白底图生成）
async function createSolidPixelMap(
  width: number,
  height: number,
  r: number = 255,
  g: number = 255,
  b: number = 255,
  a: number = 255
): Promise<image.PixelMap> {
  const pixelMap = await image.createPixelMap(
    new ArrayBuffer(width * height * 4),
    {
      editable: true,
      pixelFormat: image.PixelMapFormat.RGBA_8888,
      size: { width, height }
    }
  );
  // 填充颜色
  const buffer = new ArrayBuffer(width * height * 4);
  const data = new Uint8Array(buffer);
  for (let i = 0; i < data.length; i += 4) {
    data[i] = r; data[i + 1] = g; data[i + 2] = b; data[i + 3] = a;
  }
  await pixelMap.writeBufferToPixels(buffer);
  return pixelMap;
}

// 获取 PixelMap 信息
const info = await pixelMap.getImageInfo();
console.log(`宽: ${info.size.width}, 高: ${info.size.height}`);

// 读取像素数据
const buffer = await pixelMap.readPixelsToBuffer();
const data = new Uint8Array(buffer);
// RGBA 格式：data[0]=R, data[1]=G, data[2]=B, data[3]=A

// 打包保存为 JPEG
const packer = image.createImagePacker();
const packOptions: image.PackingOption = { format: 'image/jpeg', quality: 95 };
const packedData = await packer.packToData(pixelMap, packOptions);
packer.release();

// 释放 PixelMap（重要！）
pixelMap.release();
```

---

## 五、Core Vision Kit（基础视觉服务）

### 5.1 OCR 文字识别

```typescript
import { textRecognition } from '@kit.CoreVisionKit';
import { image } from '@kit.ImageKit';

// 在 aboutToDisappear 中调用 release()
async function recognizeText(pixelMap: image.PixelMap): Promise<string> {
  const visionInfo: textRecognition.VisionInfo = { pixelMap };
  const config: textRecognition.TextRecognitionConfiguration = {
    isDirectionDetectionSupported: false  // 是否检测文本方向
  };

  const result = await textRecognition.recognizeText(visionInfo, config);
  return result.value;  // 识别出的文字字符串
}

// 示例：选择图片后识别
async function ocrFromPicker(context: Context): Promise<string> {
  // 选择图片
  const picker = new photoAccessHelper.PhotoViewPicker();
  const res = await picker.select({
    MIMEType: photoAccessHelper.PhotoViewMIMETypes.IMAGE_TYPE,
    maxSelectNumber: 1
  });
  if (!res.photoUris.length) return '';

  // 加载为 PixelMap
  const pixelMap = await loadPixelMap(res.photoUris[0]);

  try {
    return await recognizeText(pixelMap);
  } finally {
    pixelMap.release();
    await textRecognition.release();  // 必须释放
  }
}
```

### 5.2 主体分割（背景移除 / 白底图生成）

> **约束**：
> - 不支持模拟器，必须真机测试
> - 主体大小 ≥ 原图 0.5%（1000×1000 图片中主体至少 50×50 px）
> - 建议分辨率 720p+，宽高比 ≤ 3:1

```typescript
import { subjectSegmentation } from '@kit.CoreVisionKit';
import { image } from '@kit.ImageKit';

// 完整流程
@Component
struct SegmentationPage {
  private pixelMap?: image.PixelMap;

  async aboutToAppear(): Promise<void> {
    await subjectSegmentation.init();   // 初始化引擎
  }

  async aboutToDisappear(): Promise<void> {
    await subjectSegmentation.release(); // 释放引擎
    this.pixelMap?.release();
  }

  /**
   * 执行主体分割，返回透明背景前景图
   */
  async doSegmentation(pixelMap: image.PixelMap): Promise<image.PixelMap | undefined> {
    const visionInfo: subjectSegmentation.VisionInfo = { pixelMap };
    const config: subjectSegmentation.SegmentationConfig = {
      maxCount: 20,
      enableSubjectDetails: true,
      enableSubjectForegroundImage: true,   // 必须为 true 才能获取前景图
    };

    try {
      const result: subjectSegmentation.SegmentationResult =
        await subjectSegmentation.doSegmentation(visionInfo, config);

      if (result.fullSubject?.foregroundImage) {
        return result.fullSubject.foregroundImage;  // 透明背景前景 PixelMap
      }
      return undefined;
    } catch (err) {
      const error = err as BusinessError;
      switch (error.code) {
        case 401:    console.error('参数错误'); break;
        case 146001: console.error('设备不支持主体分割'); break;
        case 146002: console.error('AI引擎初始化失败'); break;
        default:     console.error(`分割失败: ${error.message}`);
      }
      return undefined;
    }
  }

  /**
   * 生成白底图（前景图 + 白色背景合成）
   * @param foreground 透明背景前景图（来自 doSegmentation）
   * @param bgWidth 输出宽度
   * @param bgHeight 输出高度
   */
  async createWhiteBackgroundImage(
    foreground: image.PixelMap,
    bgWidth: number,
    bgHeight: number
  ): Promise<image.PixelMap> {
    const fgInfo = await foreground.getImageInfo();
    const fgW = fgInfo.size.width;
    const fgH = fgInfo.size.height;

    // 1. 读取前景像素
    const fgBuffer = await foreground.readPixelsToBuffer();
    const fgData = new Uint8Array(fgBuffer);

    // 2. 创建白色背景 buffer
    const bgBuffer = new ArrayBuffer(bgWidth * bgHeight * 4);
    const bgData = new Uint8Array(bgBuffer);
    bgData.fill(255);   // 全白（RGBA: 255,255,255,255）

    // 3. 将前景居中合成到背景
    const offsetX = Math.floor((bgWidth - fgW) / 2);
    const offsetY = Math.floor((bgHeight - fgH) / 2);

    for (let y = 0; y < fgH; y++) {
      for (let x = 0; x < fgW; x++) {
        const fgIdx = (y * fgW + x) * 4;
        const bgIdx = ((offsetY + y) * bgWidth + (offsetX + x)) * 4;
        const alpha = fgData[fgIdx + 3];   // 前景 Alpha 通道

        if (alpha > 128) {  // 不透明像素
          bgData[bgIdx] = fgData[fgIdx];
          bgData[bgIdx + 1] = fgData[fgIdx + 1];
          bgData[bgIdx + 2] = fgData[fgIdx + 2];
          bgData[bgIdx + 3] = 255;
        }
        // Alpha <= 128：保持白色背景
      }
    }

    // 4. 创建输出 PixelMap
    const result = await image.createPixelMap(bgBuffer, {
      editable: false,
      pixelFormat: image.PixelMapFormat.RGBA_8888,
      size: { width: bgWidth, height: bgHeight }
    });

    return result;
  }
}
```

---

## 六、LocationKit（定位服务）

```typescript
import { geoLocationManager } from '@kit.LocationKit';

// 权限：module.json5 声明 ohos.permission.APPROXIMATELY_LOCATION

// 获取当前位置（一次性）
const location = await geoLocationManager.getCurrentLocation({
  accuracy: geoLocationManager.LocationRequestType.PRIORITY_ACCURACY,
  timeoutMs: 10000
});
console.log(`纬度: ${location.latitude}, 经度: ${location.longitude}`);

// 持续监听位置
const requestInfo: geoLocationManager.LocationRequest = {
  priority: geoLocationManager.LocationRequestPriority.FIRST_FIX,
  scenario: geoLocationManager.LocationRequestScenario.UNSET,
  timeInterval: 5,     // 5 秒一次
  distanceInterval: 0,
  maxAccuracy: 100
};

const locationCallback = (location: geoLocationManager.Location) => {
  console.log(`更新位置: ${location.latitude}, ${location.longitude}`);
};

geoLocationManager.on('locationChange', requestInfo, locationCallback);

// 停止监听（在 aboutToDisappear 中调用）
geoLocationManager.off('locationChange', locationCallback);
```

---

## 七、NotificationKit（通知）

```typescript
import { notificationManager } from '@kit.NotificationKit';

// 发送本地通知
await notificationManager.publish({
  id: 1,
  content: {
    notificationContentType: notificationManager.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT,
    normal: {
      title: '通知标题',
      text: '通知内容详情',
      additionalText: '附加文字'
    }
  },
  showDeliveryTime: true,
  groupName: 'default'
});

// 取消通知
notificationManager.cancel(1);

// 取消所有通知
notificationManager.cancelAll();
```

---

## 八、CameraKit（相机）

```typescript
import { camera, cameraPicker } from '@kit.CameraKit';

// 推荐：使用系统相机 Picker（最简单，无需额外权限）
const pickerProfile: cameraPicker.PickerProfile = {
  cameraPosition: camera.CameraPosition.CAMERA_POSITION_BACK  // 后置摄像头
};
const result = await cameraPicker.pick(
  context,
  [cameraPicker.PickerMediaType.PHOTO],  // PHOTO 或 VIDEO
  pickerProfile
);

if (result.resultCode === 0) {
  const photoUri = result.resultUri;  // 照片 URI
  // 用 loadPixelMap(photoUri) 加载图片
}
```

---

## 九、MediaLibraryKit（媒体访问）

```typescript
import { photoAccessHelper } from '@kit.MediaLibraryKit';

// 图片选择器
const picker = new photoAccessHelper.PhotoViewPicker();
const result = await picker.select({
  MIMEType: photoAccessHelper.PhotoViewMIMETypes.IMAGE_TYPE,
  maxSelectNumber: 9  // 最多选 9 张
});
const uris: string[] = result.photoUris;

// 保存图片到相册
async function saveToAlbum(context: Context, pixelMap: image.PixelMap): Promise<string> {
  const helper = photoAccessHelper.createPhotoAccessHelper(context);
  const uri = await helper.createAsset(photoAccessHelper.PhotoType.IMAGE, 'jpg');

  const file = await fileIo.open(uri, fileIo.OpenMode.WRITE_ONLY | fileIo.OpenMode.CREATE);
  const packer = image.createImagePacker();
  const data = await packer.packToData(pixelMap, { format: 'image/jpeg', quality: 95 });
  await fileIo.write(file.fd, data);
  await fileIo.close(file);
  packer.release();

  return uri;
}
```

---

## 十、FileKit（文件操作）

```typescript
import { fileIo } from '@kit.CoreFileKit';
import { common } from '@kit.AbilityKit';

// 获取沙箱路径
const filesDir = (context as common.UIAbilityContext).filesDir;    // 文件目录
const cacheDir = (context as common.UIAbilityContext).cacheDir;    // 缓存目录
const tempDir = (context as common.UIAbilityContext).tempDir;      // 临时目录

// 写文件
const writeFile = fileIo.openSync(`${filesDir}/data.json`, fileIo.OpenMode.WRITE_ONLY | fileIo.OpenMode.CREATE);
fileIo.writeSync(writeFile.fd, JSON.stringify({ key: 'value' }));
fileIo.closeSync(writeFile);

// 读文件
const readFile = fileIo.openSync(`${filesDir}/data.json`, fileIo.OpenMode.READ_ONLY);
const stat = fileIo.statSync(readFile.fd);
const buffer = new ArrayBuffer(stat.size);
fileIo.readSync(readFile.fd, buffer);
const content = String.fromCharCode(...new Uint8Array(buffer));
fileIo.closeSync(readFile);

// 删除文件
fileIo.unlinkSync(`${filesDir}/data.json`);

// 检查文件是否存在
try {
  fileIo.accessSync(`${filesDir}/data.json`);
  console.log('文件存在');
} catch {
  console.log('文件不存在');
}
```

---

## 十一、PerformanceAnalysisKit（日志 hilog）

```typescript
import { hilog } from '@kit.PerformanceAnalysisKit';

// 日志级别：debug < info < warn < error < fatal
// %{public}s 表示公开字符串，%{private}s 表示脱敏（日志中显示 <private>）

const DOMAIN = 0x0001;   // 业务域 0x0000 ~ 0xFFFF，区分不同业务模块

hilog.debug(DOMAIN, 'ModuleTag', '调试: %{public}s, 值=%{public}d', '参数', 42);
hilog.info(DOMAIN, 'ModuleTag', '功能 %{public}s 初始化完成', 'Network');
hilog.warn(DOMAIN, 'ModuleTag', '连接超时，重试次数: %{public}d', retryCount);
hilog.error(DOMAIN, 'ModuleTag', '请求失败: %{public}s', error.message);
hilog.fatal(DOMAIN, 'ModuleTag', '严重错误，应用即将崩溃');

// 检查日志级别是否开启（避免无效字符串拼接）
if (hilog.isLoggable(DOMAIN, 'ModuleTag', hilog.LogLevel.DEBUG)) {
  hilog.debug(DOMAIN, 'ModuleTag', '性能数据: %{public}s', expensiveCalc());
}
```

---

## 十二、Kit 导入路径速查

| 能力 | 新式 Kit 导入 | 旧式导入（兼容） |
|------|------------|--------------|
| Ability | `@kit.AbilityKit` | `@ohos.app.ability.UIAbility` |
| 网络 HTTP | `@kit.NetworkKit` | `@ohos.net.http` |
| 数据持久化 | `@kit.ArkData` | `@ohos.data.preferences` |
| 图像 | `@kit.ImageKit` | `@ohos.multimedia.image` |
| 文件 | `@kit.CoreFileKit` | `@ohos.file.fs` |
| 定位 | `@kit.LocationKit` | `@ohos.geoLocationManager` |
| 通知 | `@kit.NotificationKit` | `@ohos.notificationManager` |
| 相机 | `@kit.CameraKit` | `@ohos.multimedia.camera` |
| 媒体库 | `@kit.MediaLibraryKit` | `@ohos.file.photoAccessHelper` |
| 视觉 OCR | `@kit.CoreVisionKit` | `@ohos.ai.textRecognition` |
| 视觉分割 | `@kit.CoreVisionKit` | `@ohos.ai.subjectSegmentation` |
| 日志 | `@kit.PerformanceAnalysisKit` | `@ohos.hilog` |
| 权限 | （保持旧式）| `@ohos.abilityAccessCtrl` |
| 媒体播放 | `@kit.MediaKit` | `@ohos.multimedia.media` |
| AI 语音 | `@kit.CoreSpeechKit` | `@ohos.ai.speechRecognizer` |
| AI 基础 | `@kit.HiaiFoundationKit` | `@ohos.ai.hiaiFoundation` |
| 图形加速 | `@kit.GraphicsAccelerationKit` | - |
| AR Engine | `@kit.AREngineKit` | - |
| 空间重建 | `@kit.SpatialReconKit` | - |
| XEngine | `@kit.XEngineKit` | - |

---

## 十三、MediaKit（音视频播放与录制）

### 13.1 AVPlayer 音视频播放

```typescript
import { media } from '@kit.MediaKit';

@Entry
@Component
struct VideoPlayer {
  @State private avPlayer: media.AVPlayer | null = null;
  @State private isPlaying: boolean = false;
  @State private currentTime: number = 0;
  @State private duration: number = 0;

  async aboutToAppear() {
    // 创建播放器实例
    this.avPlayer = await media.createAVPlayer();

    // 监听状态变化
    this.avPlayer.on('stateChange', (state: string) => {
      if (state === 'prepared') {
        this.duration = this.avPlayer!.duration;
      }
      if (state === 'playing') {
        this.isPlaying = true;
      } else {
        this.isPlaying = false;
      }
    });

    // 监听播放进度
    this.avPlayer.on('timeUpdate', (time: number) => {
      this.currentTime = time;
    });

    // 设置播放地址
    this.avPlayer.url = 'https://example.com/video.mp4';
  }

  async togglePlay() {
    if (!this.avPlayer) return;
    if (this.isPlaying) {
      await this.avPlayer.pause();
    } else {
      await this.avPlayer.play();
    }
  }

  aboutToDisappear() {
    this.avPlayer?.release();
  }

  build() {
    Column() {
      Video({ src: 'https://example.com/video.mp4' })
        .width('100%')
        .autoPlay(false)
        .controls(true)
        .onPrepared(() => { console.info('视频准备好了'); })
        .onFinish(() => { console.info('播放结束'); })
    }
  }
}
```

### 13.2 SoundPool（短音效播放）

> 适用场景：相机快门音、通知音、游戏音效（时长 < 5s，最多 14 个并发流）

```typescript
import { media } from '@kit.MediaKit';

let soundPool: media.SoundPool;
let soundId: number = 0;

async function initSoundPool(context: Context) {
  soundPool = await media.createSoundPool({
    maxNum: 14,   // 最大并发流数
    audioRenderInfo: { usage: media.AudioUsage.AUDIO_USAGE_GAME }
  });

  // 加载音效（支持 fd）
  soundId = await soundPool.load('/system/etc/sound/click.wav');
}

function playClickSound() {
  soundPool.play(soundId, {
    loop: 0,       // 循环次数（0=播一次）
    rate: 1.0,      // 播放速率
    leftVolume: 1.0,
    rightVolume: 1.0
  });
}
```

### 13.3 AVRecorder（音视频录制）

```typescript
import { media } from '@kit.MediaKit';
import { camera } from '@kit.CameraKit';

async function startRecording(context: Context, surfaceId: string) {
  const recorder = await media.createAVRecorder();
  const outputFile = `${context.filesDir}/video_${Date.now()}.mp4`;

  await recorder.prepare({
    audioEncoder: media.AudioEncoder.AAC_LC,
    audioSampleRate: 44100,
    audioChannels: 2,
    videoEncoder: media.VideoEncoder.H264,
    videoWidth: 1920,
    videoHeight: 1080,
    outputFile: outputFile,
    outputFormat: media.ContainerFormat.CFT_MPEG_4
  });

  // 获取录制输入 Surface（从相机传入）
  const inputSurface = await recorder.getInputSurface();
  // TODO: 将 inputSurface 配置到相机输出

  await recorder.start();
  console.info('开始录制');

  // 停止录制
  await recorder.stop();
  await recorder.release();
  console.info(`录制完成: ${outputFile}`);
}
```

---

## 十四、CameraKit（相机开发）

### 14.1 完整相机预览流程

```typescript
import { camera, cameraPicker } from '@kit.CameraKit';
import { BusinessError } from '@kit.BasicServicesKit';

@Entry
@Component
struct CameraPreview {
  private surfaceId: string = '';
  private cameraManager?: camera.CameraManager;
  private session?: camera.PhotoSession;
  private previewOutput?: camera.PreviewOutput;

  async aboutToAppear() {
    // 1. 获取相机管理器
    this.cameraManager = camera.getCameraManager(context);

    // 2. 获取支持的相机列表
    const cameras = this.cameraManager.getSupportedCameras();
    if (cameras.length === 0) {
      console.error('没有可用相机');
      return;
    }
    const backCamera = cameras.find(c => c.cameraType === camera.CameraType.CAMERA_TYPE_BACK);
    const targetCamera = backCamera || cameras[0];

    // 3. 获取相机输出能力
    const capability = this.cameraManager.getSupportedOutputCapability(targetCamera);

    // 4. 创建相机输入
    const cameraInput = this.cameraManager.createCameraInput(targetCamera);
    await cameraInput.open();

    // 5. 创建预览输出
    const previewProfile = capability.previewProfiles[0];
    this.previewOutput = this.cameraManager.createPreviewOutput(previewProfile, this.surfaceId);

    // 6. 创建会话
    this.session = this.cameraManager.createSession(camera.SceneMode.NORMAL_PHOTO) as camera.PhotoSession;

    // 7. 配置会话
    this.session.beginConfig();
    this.session.addInput(cameraInput);
    this.session.addOutput(this.previewOutput);
    await this.session.commitConfig();

    // 8. 启动预览
    await this.session.start();
  }

  aboutToDisappear() {
    this.session?.stop();
    this.previewOutput?.release();
    this.session?.release();
  }

  build() {
    XComponent({ id: 'cameraPreview', type: 'surface', controller: this.controller })
      .onLoad(() => {
        this.surfaceId = this.controller.getXcomponentSurfaceId();
      })
      .width('100%').height('100%')
  }
}
```

### 14.2 系统相机 Picker（最简方式）

```typescript
// 无需申请相机权限，直接拉起系统相机拍照/录像
import { camera } from '@kit.CameraKit';

async function takePhoto() {
  const result = await camera.pick(
    context,
    [camera.PickerMediaType.PHOTO],
    { cameraPosition: camera.CameraPosition.CAMERA_POSITION_BACK }
  );
  if (result.resultCode === 0) {
    return result.resultUri;  // 照片 URI
  }
}
```

---

## 十五、Core Speech Kit（语音识别与合成）

### 15.1 语音识别（ASR）

```typescript
import { speechRecognizer } from '@kit.CorespeechKit';

let asrEngine: speechRecognizer.SpeechRecognizer;

async function initASR() {
  const initParams = {
    language: 'zh-CN',
    online: 1,    // 1=在线, 0=离线
    extraParams: { locate: 'CN', recognizerMode: 'short' }
  };
  asrEngine = speechRecognizer.createEngine(initParams, (err, engine) => {
    if (!err) {
      asrEngine = engine;
    }
  });

  // 设置识别回调
  asrEngine.setListener({
    onStart(sessionId: string, msg: string) {
      console.info('开始识别');
    },
    onResult(sessionId: string, result: string) {
      console.info(`识别结果: ${result}`);
    },
    onComplete(sessionId: string, msg: string) {
      console.info('识别完成');
    },
    onError(sessionId: string, code: number, msg: string) {
      console.error(`错误: ${code} - ${msg}`);
    }
  });
}

async function startRecognition(audioBuffer: ArrayBuffer) {
  await asrEngine.startListening({
    sessionId: String(Date.now()),
    audioInfo: { audioType: 'pcm', sampleRate: 16000, soundChannel: 1, sampleBit: 16 }
  });
  await asrEngine.writeAudio(sessionId, new Uint8Array(audioBuffer));
  await asrEngine.finish(sessionId);
}

function releaseASR() {
  asrEngine.shutdown();
}
```

### 15.2 语音合成（TTS）

```typescript
import { textToSpeech } from '@kit.CoreSpeechKit';

let ttsEngine: textToSpeech.TtsEngine;

async function initTTS() {
  const ttsParams = { language: 'zh-CN', online: 1 };
  ttsEngine = await textToSpeech.createTtsEngine(ttsParams);

  ttsEngine.setListener({
    onStart(sessionId: string, msg: string) {
      console.info('TTS 开始播放');
    },
    onComplete(sessionId: string) {
      console.info('TTS 播放完成');
    },
    onError(sessionId: string, errCode: number, errMsg: string) {
      console.error(`TTS 错误: ${errCode}`);
    }
  });
}

async function speak(text: string) {
  await ttsEngine.speak(text, {
    sessionId: String(Date.now()),
    options: { speed: 1.0, pitch: 1.0, volume: 1.0 }
  });
}
```

---

## 十六、HiAI Foundation Kit（端侧 AI 推理）

> 用于在设备端运行 AI 模型，无需云端，支持麒麟芯片 NPU 硬件加速。

```typescript
// HiAI Foundation Kit 用于端侧 AI 模型推理
// 模型格式：.OM（华为专用格式，需通过 ModelArts 转换）
// 使用流程：加载模型 → 输入数据 → 执行推理 → 读取输出

import { hiai } from '@kit.HiaiFoundationKit';

// 推理引擎初始化
const inferModel = hiai.InferenceModel.create('model.om');

// 设置输入
const inputData = new Float32Array(inputSize);
inputData.fill(1.0);
await inferModel.setInput(0, inputData);

// 执行推理
await inferModel.run();

// 读取输出
const outputData = await inferModel.getOutput(0);

// 释放资源
inferModel.destroy();
```

> 注意：HiAI Foundation Kit 适合需要自定义 AI 模型的场景（如 OCR 后处理、图像分类、目标检测）。Core Vision Kit 已封装了常用视觉 AI 能力，更易用。

---

## 十七、ArkData 数据持久化（完整选型指南）

### 选型决策树

```
需要跨设备同步？ ──是──→ KV-Store（分布式键值库）
  │否
  ▼
数据复杂（需条件查询）？ ──是──→ RelationalStore（关系型数据库）
  │否
  ▼
数据量 < 1万条，频繁读取？ ──是──→ Preferences（用户首选项）
  │否
  ▼
考虑 RelationalStore 或文件存储
```

### 17.1 KV-Store 分布式键值存储

> 支持跨设备同步，适用于登录信息、设置等数据多端协同。

```typescript
import { distributedKVStore } from '@kit.ArkData';

// ⚠️ 必须在 onWindowStageCreate 中初始化
const kvManager = distributedKVStore.createKVManager({ context });

const kvStore = await kvManager.getKVStore<distributedKVStore.SingleKVStore>('my-store', {
  createIfMissing: true,
  kvStoreType: distributedKVStore.KVStoreType.DEVICE_COLLABORATION,  // 跨设备
  securityLevel: distributedKVStore.SecurityLevel.S1
});

// 增/改
await kvStore.put('userInfo', JSON.stringify({ name: 'Alice', age: 20 }));

// 查
const value = await kvStore.get('userInfo');

// 监听变化（跨设备同步）
kvStore.on('dataChange', distributedKVStore.SubscribeType.SUBSCRIBE_TYPE_ALL, (data) => {
  console.info('数据变化:', JSON.stringify(data));
});

// 删
await kvStore.delete('userInfo');
```

---

## 十八、自由流转（分布式能力）

### 18.1 两大流转类型

| 类型 | 场景 | 技术要点 |
|------|------|----------|
| **跨端迁移（Migration）** | 手机→平板继续操作 | `onContinue()` 保存状态，目标设备接续 |
| **多端协同（Collaboration）** | 多设备同时在线协作 | `startAbility()` 跨设备启动，数据分布式共享 |

### 18.2 跨端迁移示例

```typescript
// 在 UIAbility 中声明支持迁移
import UIAbility from '@kit.AbilityKit';

export default class VideoAbility extends UIAbility {
  // 应用迁移时调用（保存状态）
  onContinue(wantParam: Record<string, Object>) {
    // 返回 CONTINUE_SEND_SUCCESS 表示支持迁移
    return UIAbility.ContinuationCallbackResult.CONTINUE_SEND_SUCCESS;
  }

  // 应用接续时调用（恢复状态）
  onContinueReceived(want: Want) {
    const resumeData = want.parameters?.['data'] as Record<string, Object>;
    if (resumeData) {
      this.restoreState(resumeData);
    }
  }
}
```

### 18.3 跨设备数据同步（分布式数据服务）

```typescript
// 使用 KV-Store 的 DEVICE_COLLABORATION 类型实现跨设备数据同步
// 登录后自动同步，多设备间数据实时一致
const kvStore = await kvManager.getKVStore('app-store', {
  kvStoreType: distributedKVStore.KVStoreType.DEVICE_COLLABORATION
});

// 任意设备写入，所有设备自动同步
await kvStore.put('cart', JSON.stringify([{ id: 1, name: '项链', qty: 1 }]));
```

---

## 十九、NDK 开发（C/C++ 高性能模块）

### 19.1 项目结构

```
entry/src/main/
├── cpp/
│   ├── native.cpp          # C++ 实现
│   ├── CMakeLists.txt      # 构建配置
│   └── include/
│       └── mylib.h         # 头文件
├── module.json5             # 配置 native 模块
└── build-profile.json5     # CMake 路径配置
```

### 19.2 N-API 桥接函数模板

```cpp
#include <napi/native_api.h>

// C++ 核心逻辑
int64_t heavy_computation(int n) {
  // CPU 密集型计算
  return fibonacci(n);
}

// N-API 桥接函数（ArkTS 调用入口）
static napi_value Compute(napi_env env, napi_callback_info info) {
  size_t argc = 1;
  napi_value args[1];
  napi_get_cb_info(env, info, &argc, args, nullptr, nullptr);

  int32_t n;
  napi_get_value_int32(env, args[0], &n);        // 提取 ArkTS 参数

  int64_t result = heavy_computation(n);          // 执行 C++ 计算

  napi_value res;
  napi_create_int64(env, result, &res);          // 返回值给 ArkTS
  return res;
}

// 模块注册
static napi_value Init(napi_env env, napi_value exports) {
  napi_property_descriptor desc[] = {
    { "compute", nullptr, Compute, nullptr, nullptr, nullptr, napi_default, nullptr }
  };
  napi_define_properties(env, exports, 1, desc);
  return exports;
}

NAPI_MODULE(NODE_GYP_MODULE_NAME, Init)
```

### 19.3 CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(MyNativeLib)

add_library(native_lib SHARED src/main/cpp/native.cpp)

target_link_libraries(native_lib PUBLIC
    libace_napi.z.so    # Node-API 运行时
    libhilog_ndk.z.so   # 日志
)
```

### 19.4 ArkTS 调用

```typescript
import nativeLib from 'libnative_lib.so';

const result = nativeLib.compute(40);  // 直接调用 C++ 函数
console.info(`C++ 计算结果: ${result}`);
```

---

## 二十、一次开发多端部署（响应式布局）

### 20.1 断点系统

| 断点 | 宽度范围 | 设备类型 |
|------|----------|----------|
| `sm` | < 600vp | 手机 |
| `md` | 600-840vp | 折叠屏、小平板 |
| `lg` | 840-1440vp | 平板 |
| `xl` | > 1440vp | PC、2in1 |

### 20.2 断点自适应示例

```typescript
import window from '@ohos.window';

// 获取当前窗口断点
const windowInfo = await window.getLastWindow(context);
const widthVp = windowInfo.getDensityMetrics().width / windowInfo.getDensityMetrics().density;
const breakpoint = widthVp < 600 ? 'sm' : widthVp < 840 ? 'md' : widthVp < 1440 ? 'lg' : 'xl';

// 监听窗口变化
window.on('windowSizeChange', (size) => {
  // 动态更新断点
  this.currentBreakpoint = this.getBreakpoint(size.width);
});

// WidthBreakpointType 工具类
new WidthBreakpointType(1, 2, 3, 3).getValue(this.currentBreakpoint);
// sm=1, md=2, lg=3, xl=3 → 用于 lanes、columnsTemplate 等
```

### 20.3 四种响应式布局

```typescript
// 1. 重复布局（List 列数随屏幕宽度变化）
List()
  .lanes(new WidthBreakpointType(1, 2, 3, 3).getValue(breakpoint))

// 2. 分栏布局（Navigation 单/双栏切换）
Navigation()
  .mode(this.isWideScreen ? NavigationMode.Split : NavigationMode.Stack)

// 3. 挪移布局（GridCol 栅格分配）
GridRow() {
  GridCol({ span: { xs: 4, md: 8, lg: 4 } })  // 手机占满，平板左侧
  GridCol({ span: { xs: 4, md: 8, lg: 8 } })  // 右侧内容
}

// 4. 缩进布局（两侧留白，居中展示）
GridCol({
  span: { xs: 4, md: 6, lg: 8 },
  offset: { xs: 0, md: 1, lg: 2 }
})
```

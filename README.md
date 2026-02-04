# WayShot V2.0 摄影师风格

## 1. 概述

**一句话描述**: 引入 AI 摄影师风格系统，用户拍照前选择摄影师，直接出对应风格的成片。

**核心改动**:
- 新增「摄影师模式」，与现有「滤镜模式」并列
- 摄影师模式消耗 quota，滤镜模式保持原逻辑
- 照片先存 App 内相册，不再自动保存到系统相册

**成功指标**: 展示-出片率 ≥ 90%

---

## 2. 范围定义

### 本期包含

- 相机页面
  - 摄影师/滤镜双模式切换
  - 摄影师实时 LUT 预览
  - 闪光灯状态联动
  - AI 语音功能
  - IP 形象动效
- 拍照/导入流程
  - Quota 检查与付费引导
  - 照片自动保存到 App 相册
- 相册模块
  - 照片列表与筛选
  - 批量选择/删除/分享
  - 照片详情页
- 引导流程
  - 新用户引导优化
  - 首页摄影师选择弹窗

### 本期不包含

- 更多摄影师风格扩展
- 社交分享功能优化
- 个人主页改版

### 依赖项

- 摄影师素材资源（头像、样图、语音包）
- LUT 滤镜文件
- 服务端摄影师配置接口
- TTS 语音服务

---

## 3. 页面流程

### 3.1 启动流程

\`\`\`mermaid
flowchart TD
    %% 样式定义
    classDef startNode fill:#f9f0ff,stroke:#333,stroke-width:1px;
    classDef processNode fill:#fff,stroke:#333,stroke-width:1px;
    classDef decisionNode fill:#fffbe6,stroke:#333,stroke-width:1px;

    %% 节点定义
    OldUser("老用户启动App"):::startNode
    NewUser("新用户启动App"):::startNode
    NewFlow["新用户引导流程"]:::processNode
    CheckSeen{"是否看过<br>摄影师风格引导"}:::decisionNode
    GuideModal["摄影师选择引导弹窗"]:::processNode
    CameraPage["相机页面"]:::processNode

    %% 连接
    NewUser --> NewFlow --> GuideModal
    OldUser --> CheckSeen
    CheckSeen -- "否" --> GuideModal
    CheckSeen -- "是" --> CameraPage
    GuideModal -- "选择完成" --> CameraPage
\`\`\`

### 3.2 拍照/导入流程

\`\`\`mermaid
flowchart TD
    %% 样式定义
    classDef processNode fill:#fff,stroke:#333,stroke-width:1px;
    classDef decisionNode fill:#fffbe6,stroke:#333,stroke-width:1px;

    %% 入口
    TakePhoto["拍照"]:::processNode
    ImportPhoto["导入照片"]:::processNode
    CheckType{"照片风格类型"}:::decisionNode

    TakePhoto --> CheckType
    ImportPhoto --> CheckType

    %% 滤镜分支
    FilterLogic["会员滤镜逻辑<br>原逻辑不变"]:::processNode
    PhotoDone["拍照完成<br>自动修图"]:::processNode

    CheckType -- "滤镜" --> FilterLogic --> PhotoDone

    %% 摄影师分支
    CheckQuota{"是否有quota"}:::decisionNode
    PreviewRes["拍照结果预览"]:::processNode
    ChoicePay{"用户选择"}:::decisionNode
    PayWall["付费墙"]:::processNode
    ChoicePayResult{"付费结果"}:::decisionNode
    ClosePre["关闭预览<br>照片不保存"]:::processNode

    CheckType -- "摄影师" --> CheckQuota
    CheckQuota -- "是" --> PhotoDone
    CheckQuota -- "否" --> PreviewRes --> ChoicePay

    ChoicePay -- "点击关闭" --> ClosePre
    ChoicePay -- "点击付费" --> PayWall --> ChoicePayResult

    ChoicePayResult -- "取消付费" --> PreviewRes
    ChoicePayResult -- "付费成功" --> PhotoDone
\`\`\`

### 3.3 相册流程

\`\`\`mermaid
flowchart TD
    %% 样式定义
    classDef processNode fill:#fff,stroke:#333,stroke-width:1px;
    classDef decisionNode fill:#fffbe6,stroke:#333,stroke-width:1px;

    %% 入口
    ViewPhoto["查看照片"]:::processNode
    CheckHasPhoto{"是否有照片"}:::decisionNode
    AlbumNo["相册-无照片"]:::processNode
    AlbumYes["相册-有照片"]:::processNode
    CheckType{"照片风格类型"}:::decisionNode

    ViewPhoto --> CheckHasPhoto
    CheckHasPhoto -- "否" --> AlbumNo
    CheckHasPhoto -- "是" --> AlbumYes --> CheckType

    %% 处理逻辑
    Processing["处理中"]:::processNode
    ProcDone["处理完成"]:::processNode
    Result["成片"]:::processNode
    ChoiceAction{"用户选择"}:::decisionNode

    CheckType -- "滤镜" --> Result
    CheckType -- "摄影师" --> Processing --> ProcDone --> Result

    %% 操作
    SaveShare["保存&分享"]:::processNode
    ShareModal["保存成功<br>分享弹层"]:::processNode
    Delete["删除"]:::processNode
    DelSuccess["删除成功"]:::processNode
    RemoveDark["从darkroom中<br>移除照片"]:::processNode

    Result --> ChoiceAction
    ChoiceAction --> SaveShare --> ShareModal
    ChoiceAction --> Delete --> DelSuccess --> RemoveDark
\`\`\`

### 3.4 新用户引导流程

\`\`\`mermaid
flowchart LR
    %% 样式定义
    classDef processNode fill:#fff,stroke:#333,stroke-width:1px;

    %% 节点定义
    Splash["开屏视频"]:::processNode
    Demo1["摄影师1演示<br>Coco"]:::processNode
    Demo2["摄影师2演示<br>Digi"]:::processNode
    Demo3["摄影师3演示<br>Mono"]:::processNode
    Reviews["用户评价页"]:::processNode
    Paywall["订阅墙"]:::processNode
    SelectModal["摄影师选择弹窗"]:::processNode
    CameraPage["相机页"]:::processNode

    %% 连接
    Splash --> Demo1 --> Demo2 --> Demo3 --> Reviews --> Paywall --> SelectModal --> CameraPage
\`\`\`

---

## 4. 功能模块

### 4.1 相机页面

#### 页面结构

\`\`\`
camera_page
├── top_bar
│   ├── flash_toggle              // 闪光灯开关
│   ├── ai_voice_toggle           // 仅摄影师模式显示
│   └── settings_button
├── viewfinder
│   └── live_filter_preview       // 实时 LUT 预览
├── photographer_panel            // 仅摄影师模式显示
│   ├── ip_avatar                 // 动效状态: 呼吸态/说话态
│   └── sample_button             // 点击打开样图弹窗
├── style_selector
│   ├── mode_tabs                 // ["Photographer", "Filter"]
│   └── style_carousel            // 左右滑动/点击选择
└── action_bar
    ├── album_entry               // 进入相册
    ├── capture_button            // 拍照
    └── import_button             // 从系统相册导入
\`\`\`

#### 模式切换逻辑

\`\`\`javascript
// 切换到滤镜模式
function switchToFilterMode() {
  hide(ai_voice_toggle)           // AI 语音开关隐藏
  hide(photographer_panel)        // IP 形象隐藏
  hide(ai_composition_toggle)     // AI 构图开关隐藏
  // 所有与 AI 摄影师相关的 UI 元素均隐藏
}

// 切换到摄影师模式
function switchToPhotographerMode() {
  show(ai_voice_toggle)
  show(photographer_panel)
  show(ai_composition_toggle)
}
\`\`\`

#### 闪光灯联动逻辑

\`\`\`javascript
let flashManuallyChanged = false  // 本次启动是否手动改过

function onPhotographerChange(photographer) {
  // 切换 AI 摄影师时，默认把闪光灯置为摄影师对应的值
  if (!flashManuallyChanged) {
    setFlashMode(photographer.flash_mode_default)
    showToast("Flash mode changed")  // 伴随 toast 提示
  }
  applyLUT(photographer.lut_file_url, photographer.intensity_default)
}

function onFlashManualToggle() {
  // 当用户手动切换闪光灯状态后
  // 本次启动 app 期间，切换 AI 摄影师不再切换闪光灯状态
  flashManuallyChanged = true
}

function onAppLaunch() {
  // 再次启动 app 后，闪光灯状态为上次的状态
  // 但切换 AI 摄影师需要切换闪光灯状态，直到用户再次手动切换
  flashManuallyChanged = false
  restoreLastFlashState()
}

// 切换滤镜无需变更闪光灯状态
function onFilterChange(filter) {
  applyLUT(filter.lut_file_url, filter.intensity_default)
  // 不改变闪光灯状态
}
\`\`\`

#### AI 语音触发逻辑

\`\`\`javascript
let greetedPhotographers = new Set()  // 本次启动已打招呼的摄影师

function onPhotographerSelect(photographer) {
  // 本次 app 启动后首次选中该摄影师：播放打招呼语音
  if (!greetedPhotographers.has(photographer.id)) {
    playVoice(photographer.voice_assets_pkg, 'greeting')
    setAvatarAnimation('speaking')  // IP 形象具备说话动效
    greetedPhotographers.add(photographer.id)
  }
  // 非首次选中同一摄影师：不播放打招呼语音
}

function onCaptureComplete(photographer) {
  // 拍照完成时：播放夸赞语音——夸赞逻辑和原逻辑一致
  playVoice(photographer.voice_assets_pkg, 'praise')
  setAvatarAnimation('speaking')
}
\`\`\`

#### IP 形象动效

\`\`\`javascript
// IP 形象具备呼吸态动效（待机状态下的微动效果）
function setIdleAnimation() {
  setAvatarAnimation('breathing')
}

// IP 形象具备说话动效（AI 语音播放时触发）
function onVoiceStart() {
  setAvatarAnimation('speaking')
}

function onVoiceEnd() {
  setAvatarAnimation('breathing')
}
\`\`\`

---

### 4.2 预览页（quota 不足时）

#### 触发条件

摄影师模式下 quota 不足时，拍摄完成后进入预览页

#### 页面结构

\`\`\`
preview_page
├── photo_preview         // 全屏照片预览
├── close_button          // 右上角关闭按钮
└── payment_banner        // 底部付费引导 banner
    └── cta_button        // 点击跳转付费页
\`\`\`

#### 交互逻辑

\`\`\`javascript
function onBannerClick(userStatus) {
  if (!userStatus.isSubscribed) {
    // 未订阅时点击 banner 跳转会员订阅页面
    navigateTo('subscription_page')
  } else {
    // 已订阅点击 banner 跳转 quota 购买页
    navigateTo('quota_purchase_page')
  }
}

function onPaymentSuccess() {
  // 用户选择付费成功：自动返回相机页，照片保存成功
  savePhotoToAlbum()
  navigateTo('camera_page')
}

function onPaymentCancel() {
  // 用户取消付费：返回预览页，可继续查看照片但不保存
  // 保持在预览页
}

function onCloseButtonClick() {
  // 用户主动关闭预览页：返回相机页，照片不保存
  discardPhoto()
  navigateTo('camera_page')
}
\`\`\`

---

### 4.3 滤镜模式付费逻辑

#### 权益规则

- 所有滤镜拍摄不消耗 quota
- 会员滤镜需要会员身份可以使用

#### 非会员使用会员滤镜流程

\`\`\`javascript
function onCaptureWithProFilter(filter, userStatus) {
  // 用户点击拍摄按钮后正常拍摄
  capturePhoto()

  if (filter.isProFilter && !userStatus.isMember) {
    // 拍摄完成后进入预览页，展示订阅 banner
    showPreviewWithSubscriptionBanner()
  } else {
    // 正常保存
    savePhotoToAlbum()
  }
}

function onSubscriptionBannerClick() {
  // 点击跳转会员订阅页
  navigateTo('subscription_page')
}
\`\`\`

---

### 4.4 导入照片

#### 相机页入口

\`\`\`
camera_page
└── action_bar
    └── import_button     // 点击 icon，拉起系统相册
\`\`\`

#### 系统相册选择

\`\`\`
system_album_picker
├── photo_grid            // 照片网格
├── selection_counter     // 左下角展示选择了多少张
└── confirm_button        // 确认按钮，可以多选或单选照片
\`\`\`

#### 逻辑说明

- 逻辑和流程和拍照相同（导入的照片就相当于拍照）
- 相册权限逻辑——同相机拍摄相同

---

### 4.5 新功能引导弹窗

#### 触发条件

- 新用户在完成新用户引导流程后，首次进入 app 主界面时必须弹出此引导
- 老用户（已安装用户升级到新版本后）首次启动 app 时，如未弹出过此引导则需弹出
- 每个用户账号仅弹出一次，弹出后记录状态，后续不再展示
- 引导状态需与用户设备绑定，避免重复触发

#### 页面结构

\`\`\`
feature_guide_modal
├── header
│   ├── title             // "Find Your Style"
│   └── subtitle          // "Pick a photographer and start shooting"
├── photographer_carousel // 左右滑动切换
│   └── photographer_card
│       ├── sample_images // 大图展示，自动轮播 3-5 张样片
│       ├── avatar        // 摄影师卡通形象/头像
│       ├── name          // 摄影师名称
│       ├── description   // 风格介绍文案
│       └── tags[]        // 风格标签和场景标签
├── hint_text             // "Don't worry, you can change this anytime"
└── cta_button            // "Shoot with {name}"，文案跟随当前卡片变化
\`\`\`

#### 交互逻辑

\`\`\`javascript
function onPhotographerCardChange(photographer) {
  // 卡片切换时样图自动继续轮播
  updateCtaButtonText(\`Shoot with \${photographer.name}\`)
}

function onCtaButtonClick(photographer) {
  // 自动选中对应的摄影师风格
  setCurrentPhotographer(photographer)
  // 直接进入相机拍摄界面
  // 相机默认应用该摄影师的滤镜效果
  navigateTo('camera_page')
}
\`\`\`

---

### 4.6 摄影师演示页（新用户引导）

#### 页面结构

\`\`\`
photographer_demo
├── video_section
│   ├── title             // "What I look"
│   └── video_player      // 模拟拍摄，有抖动感+镜头对焦效果
├── result_section
│   ├── title             // "What I take"
│   └── result_image      // 成片效果
├── description           // 风格描述文案
└── buttons
    ├── shoot_button      // "Shoot with {name}" → 跳转相机
    └── continue_button   // "Continue" → 下一个摄影师
\`\`\`

#### 三个摄影师演示内容

| 摄影师 | What I look | What I take | 文案 |
|-------|-------------|-------------|------|
| Coco | 室内顶光，肤色暗沉，构图随意 | G7x 柔光，肤色清透有光泽，空气感 | Get the G7x vibe instantly. Natural, airy, and radiantly you. |
| Digi | 环境昏暗，噪点多，人脸发黑 | CCD 直闪，人脸曝光完美，复古噪点，Y2K 风 | Use CCD flasher turn Boring Night into Party Queen |
| Mono | 普通街道照片，过曝，平平无奇 | 高反差黑白，颗粒感强，电影叙事感 | Turn a Street into Cinema Masterpiece |

---

### 4.7 首页摄影师选择弹窗

#### 触发条件

- 新用户：付费墙之后（关掉或付费）
- 老用户：如果还没出过，首次打开 app 时出一次

#### 页面结构

\`\`\`
home_photographer_select
├── top_bar
│   ├── title             // "Select Your Artist"
│   ├── lightning_icon    // 闪电入口
│   └── discord_icon      // Discord 入口
├── photographer_carousel // 左右滑动卡片
│   └── photographer_card
│       ├── sample_images // 10张自动轮播（不支持手动切换）
│       ├── avatar
│       ├── name          // 人名+风格
│       ├── vibe_title    // 相机型号标题
│       ├── description   // 正文介绍
│       └── tags[]        // 关键词标签
└── cta_button            // "Shoot with {name}"
\`\`\`

#### 选择后行为

\`\`\`javascript
function onPhotographerSelect(photographer) {
  // 每日发放3个免费quota弹窗保留，先弹弹窗（原有弹窗逻辑、问卷弹窗逻辑保留）

  // App 执行页面跳转，从首页直接进入相机取景框页面
  setCurrentPhotographer(photographer)

  // 对应 IP 进行语音的打招呼
  playVoice(photographer.voice_assets_pkg, 'greeting')

  // 出个简短的 toast 文案
  showToast(photographer.text_toast)

  // 用户无需二次操作，进入相机页时，系统已自动配置好该摄影师对应的所有参数和 UI 状态
  navigateTo('camera_page')
}
\`\`\`

#### Toast 文案配置

| 摄影师 | Toast 文案 |
|-------|-----------|
| Coco | Coco is ready. Find the light. ✨ |
| Digi | Digi is ready. Let's party! ⚡️ |
| Mono | Mono is active. Capture the moment. 🌑 |

---

### 4.8 相册列表

#### 页面结构

\`\`\`
album_list
├── top_bar
│   ├── title                     // "Album"
│   ├── filter_dropdown           // 筛选: 可以选不同的摄影师和滤镜
│   └── select_button             // "Select"（或图标）
├── photo_grid                    // 3 列网格
│   └── photo_thumbnail[]
│       ├── image
│       └── status_overlay        // 生成中: 波浪感/蒙层, 异常: 错误状态
└── batch_action_bar              // 选择模式显示, 覆盖 TabBar
    ├── delete_button             // 左侧 "🗑 Delete (n)" 黑色/暗色背景，警示色文字
    └── share_button              // 右侧 "📥 Share (n)" 品牌色（粉色）背景
\`\`\`

#### 选择模式逻辑

\`\`\`javascript
let selectedPhotos = []
let isSelectMode = false

function onSelectButtonClick() {
  isSelectMode = !isSelectMode
  // 右侧: select 变 cancel
  updateSelectButtonText(isSelectMode ? 'Cancel' : 'Select')
  if (!isSelectMode) selectedPhotos = []
}

function onPhotoTap(photo) {
  if (!isSelectMode) {
    // 点击照片缩略图：进入 darkroom（darkroom 和当前逻辑保持一致）
    navigateTo('photo_detail', { photoId: photo.id })
    return
  }

  // 未完成不能被选中
  if (photo.status === 'processing') return

  if (selectedPhotos.includes(photo.id)) {
    // 未选中态: 图片右上角显示空心圆圈
    selectedPhotos.remove(photo.id)
  } else {
    // 选中态: 图片右上角显示红色实心对勾，图片可能稍微缩小或增加边框高亮
    selectedPhotos.push(photo.id)
  }

  // 计数器: 按钮文案中的 (n) 必须随选中数量实时更新
  updateButtonCounters(selectedPhotos.length)
}

// 禁用状态: 当选中数量为 0 时，两个按钮置灰
function updateButtonCounters(count) {
  setDeleteButtonEnabled(count > 0)
  setShareButtonEnabled(count > 0)
}

function onDeleteTap() {
  // 点击 delete，出 delete 提示弹窗
  showDeleteConfirmDialog(selectedPhotos)
}

function onShareTap() {
  // 点击 share，出已有的 share 面板，第一个是 save to album（不要大的预览图）
  showShareSheet(selectedPhotos)
}
\`\`\`

---

### 4.9 照片详情页

#### 触发条件

点击缩略图进入

#### 页面结构

\`\`\`
photo_detail
├── top_bar
│   ├── back_button               // 点击返回相册列表
│   └── counter                   // 显示当前位置（如 1/3）
├── photo_display
│   ├── main_photo
│   ├── photographer_tag          // 半透明黑色胶囊，位置: 图片左上角或底部中央
│   │                             // 文案: "📸 Shot with {name} {emoji}"
│   │                             // 点击跳转该摄影师的拍摄页面（"再拍一张"的快捷入口）
│   └── before_after_toggle       // Before=纯原图(不带LUT), After=所有效果
├── thumbnail_strip               // 底部缩略图胶卷条，当前选中高亮，左右可快速滑动切换
└── action_bar
    ├── edit_button               // 编辑（如果选择滤镜，无 chips）
    ├── favorite_button           // 收藏按钮，点击后相册缩略图显示小心心
    └── share_button              // 分享
\`\`\`

#### 分享权益逻辑

\`\`\`javascript
function onShareButtonClick(photo, userStatus) {
  if (!userStatus.isMember) {
    // 非会员：保存带有 "Wayshot" 水印的图片，并有升级长条
    saveWithWatermark(photo)
    showUpgradeBanner()
  } else {
    // 会员：保存无水印原图
    saveWithoutWatermark(photo)
  }
}
\`\`\`

#### 摄影师标签文案

| 摄影师 | 标签文案 |
|-------|---------|
| Coco | 📸 Shot with Coco ✨ |
| Digi | 📸 Shot with Digi ⚡️ |
| Mono | 📸 Shot with Mono 🌑 |

---

## 5. 数据结构

### 5.1 摄影师配置（服务端下发）

\`\`\`typescript
interface Photographer {
  // 基础信息
  photographer_id: string       // 摄影师唯一标识，如 "Digi001"
  display_name: string          // 显示名称，如 "Digi"
  avatar_url: string            // IP形象/头像地址（静态图片或动效资源包地址）
  sort_order: number            // 摄影师在滤镜页面的排序位置

  // 风格信息
  vibe_title: string            // 风格，如 "Canon digi · Direct Flash"
  description: string           // 风格介绍文案
  tags: string[]                // 关键词标签列表，如 ["Party", "Flash", "CCD"]
  sample_images: string[]       // 样照 URL 列表，6张，用于首页轮播和 SAMPLE 弹窗

  // 滤镜配置
  lut_file_url: string          // 滤镜 LUT 文件，实时渲染及后期处理的核心文件
  intensity_default: number     // 滤镜默认强度，0.0 - 1.0
  flash_mode_default: number    // 闪光灯默认状态，0:自动, 1:强制开启, 2:关闭

  // 语音配置
  voice_id: string              // TTS 音色标识，如 "soothing_lily_01"
  voice_assets_pkg: string      // 语音包，包含呼吸态、打招呼、夸赞音频
  prompt?: string               // 对应的语音的 prompt

  // 文案
  text_toast: string            // 从首页选择摄影师进入的 toast 文案
}
\`\`\`

---

## 6. 摄影师配置数据

### Coco

| 字段 | 值 |
|------|-----|
| display_name | Coco |
| vibe_title | Canon G7x · Soft Focus |
| tags | \`["Clean", "Glow", "Natural", "Daily", "Outdoor"]\` |
| description | Airy Outdoor captures that crisp, iPhone-like transparency and clarity. Inspired by the trending Canon G7X, this style delivers a soft, glowing aesthetic—ideal for natural light, city walks, and effortless daily snapshots. |
| flash_mode_default | \`1\` (强制开启) |
| text_toast | Coco is ready. Find the light. ✨ |
| 人设 | 松弛感女生，25岁，热爱普拉提和抹茶，追求"毫不费力"的高级美，喜欢明亮的自然光 |
| 音色 | 清透治愈音，中音区，语速平缓从容，声音干净温暖，咬字清晰温柔，口头禅: "So fresh", "Morning light" |
| 适合场景 | 自然户外、阳光草坪、海滩 |

### Digi

| 字段 | 值 |
|------|-----|
| display_name | Digi |
| vibe_title | Canon digi · Direct Flash |
| tags | \`["Party", "Flash", "CCD", "Y2K"]\` |
| description | Captures the chaotic energy of a night out. High exposure, vintage digital grain, and that signature Y2K direct flash look. No skin smoothing, just raw vibes. |
| flash_mode_default | \`1\` (强制开启) |
| text_toast | Digi is ready. Let's party! ⚡️ |
| 人设 | 派对女王，20岁，永远在派对 C 位，喜欢捕捉夜晚的混乱与活力，拒绝磨皮，追求真实的高曝光 |
| 音色 | 元气少女音，语速稍快，音调较高，带有明显的笑意和呼吸感，口头禅: "Slay!", "Love this vibe!" |
| 适合场景 | Party、夜生活、夜晚路边 |

### Mono

| 字段 | 值 |
|------|-----|
| display_name | Mono |
| vibe_title | Ricoh GR · High Contrast |
| tags | \`["B&W", "Street", "Grain", "Mood"]\` |
| description | Timeless high-contrast monochrome. Inspired by street photography legends, emphasizing shadows and emotion over color. Ideal for city textures and storytelling. |
| flash_mode_default | \`2\` (关闭) |
| text_toast | Mono is active. Capture the moment. 🌑 |
| 人设 | 街头观察者，30岁，沉默寡言的艺术家，喜欢在街角观察光影，认为色彩会干扰情绪的表达 |
| 音色 | 低沉磁性音，低音区，语速较慢，带有颗粒感和磁性，情绪起伏不大但很有故事感，口头禅: "Wait for the moment" |
| 适合场景 | 街拍、城市肌理、光影 |

---

## 7. 存量数据处理

- **存量数据**: 不需要把个人主页的都同步到相册里
- **归档位置**: 合并后的所有照片统一进入新增的【App 内相册】
- **用户感知**: 用户升级后，打开新版"个人主页"内容保持不变，但在新增的"相册"中可以看到包含主页照片在内的所有历史创作
- **增量数据逻辑**: 保存、分享之后，自动分享到个人主页
- **删除重装**: 相册内容清空

---

## 8. 埋点需求

照片全链条需记录属于哪个摄影师 or 滤镜

---

## 9. 升级管理

- 订阅强更
- 2.0 版本弱更

---

## 10. 待确认

- [ ] IP 形象使用卡通 IP 还是风格预览图？
- [ ] 语音文件格式和大小限制？
- [ ] 摄影师配置接口具体字段确认？
- [ ] 摄影师素材采用配置的方式下发（字段待确认）

# 护眼小卫士（Eye Care Guardian）

> 华为 ICT 大赛创新赛参赛项目 —— 基于浏览器摄像头的**实时用眼行为监测与提醒**工具。
> 画面与关键点检测**完全在本机完成**，不上传任何视频/图像数据。

## 功能特性

- **五大用眼指标**（实时计算 + 风险条）：
  - 用眼距离指数（脸部占比 + 个人校准基线）
  - 头部歪斜
  - 眨眼频率（双阈值滞回 + 去抖，累计计数）
  - 环境光照（优先调用 AmbientLightSensor，不支持时由画面亮度估算）
  - 连续盯屏时长（基于系统时钟，后台不丢计时）
- **坐姿端正度**（新增）：仅用脸部关键点估算 roll/yaw/pitch/前倾，自动学习你的端正基线，歪头/趴近即时掉分。
- **综合疲劳等级**：上述指标加权（距离 0.28 / 歪斜 0.17 / 眨眼 0.20 / 光照 0.25 / 坐姿 0.10）得到 0–100 分，分正常 / 轻度 / 中度 / 重度四档。
- **后台持续监测**：开启后自动弹出「悬浮监测窗」，最小化主窗口、保持悬浮窗可见即可在后台持续统计（含眨眼）。
- **多窗口数据一致**：主页面与悬浮窗通过 BroadcastChannel 单写者 + 本地快照保持数据完全一致。
- **提醒机制**：系统通知 + 提示音 + 休息页跳转（20-20-20 法则），带冷却避免刷屏。

## 技术栈

- 纯前端单文件应用（`eye-care-guardian.html`），无构建步骤。
- [MediaPipe Tasks Vision · FaceLandmarker](https://ai.google.dev/edge/mediapipe)（VIDEO 模式，本地 `vendor/tasksvision/` 的 wasm + 模型，并带 jsDelivr CDN 兜底，GPU→CPU 超时降级）。
- 后台计时借助 Web Worker，规避浏览器后台节流。

## 本地运行

> 摄像头需要在 `https` 或 `localhost` 下才能授权，**不能用 `file://` 双击打开**。

方式一（Python，最简单）：

```bash
cd <项目目录>
python -m http.server 8000
# 浏览器打开 http://localhost:8000/eye-care-guardian.html
```

方式二（Node，vendor 全本地化，离线可用）：

```bash
cd <项目目录>
node tools/serve.mjs 8788   # 需本地有该脚本；或任意静态服务器
# 浏览器打开 http://127.0.0.1:8788/eye-care-guardian.html
```

## 在线体验

- **GitHub Pages（公开网页，推荐）**：
  ```
  https://aurora-whxy.github.io/eye-care-guardian/
  ```
  自动跳进主程序，无需 `sharecode`，全部资源随仓库托管。

- **WorkBuddy 分享链接**（转发务必带 `sharecode`，否则本地模型无法加载）：
  ```
  https://eye-care-guardian-45559.app.workbuddy.host/?sharecode=WF3EGKITEQ
  ```

> 线上 wasm 可能回退到 CDN，请确保网络可访问 `cdn.jsdelivr.net`；大赛演示建议用上面的本地方式以保证离线可用。

## 使用提示

1. 打开页面 → 点「开启摄像头」（会自动弹出悬浮监测窗）→ 授权摄像头。
2. 点顶部「校准距离」「校准坐姿」坐正坐好采集你的个人基线，之后指标更准。
3. 最小化主窗口、保持悬浮窗可见，即可后台持续监测。
4. 若连悬浮窗也完全隐藏，浏览器会把采样限流到约 1 次/秒，而眨眼仅 100–300 毫秒，此时物理上无法统计——请保持悬浮窗可见。

## 目录结构

```
.
├── eye-care-guardian.html   # 主程序（最新版，含坐姿监测 + 后台同步）
├── index.html               # GitHub Pages 入口，自动跳转主程序
├── eyecare.html             # 早期版本（保留参考）
├── vendor/tasksvision/      # MediaPipe 模型与 wasm（本地依赖）
└── .gitignore
```

## 免责声明

本工具用于**用眼习惯提醒与自我监测**，非医疗诊断设备，结果仅供参考。

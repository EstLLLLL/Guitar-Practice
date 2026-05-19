# 吉他练习日志 PWA

一个本地优先的吉他练习记录工具，支持：

- 练习记录（时长、和弦、歌曲、focus、笔记）
- 30 天练习柱状图、连续天数、累计小时
- 和弦库 + 歌曲库（4 级状态切换：学习中 / 练习中 / 能弹完整 / 熟练）
- **音频录制 + 客观分析**：BPM、节奏稳定性、音准偏差（cents）、力度均匀度
- 浏览器内自实现的 FFT、YIN 音高检测、谱通量 onset 检测
- 波形 / 音高曲线 / RMS 包络可视化
- 让 Claude 解读练习数据 + 录音指标，给针对性改进建议
- 完全离线工作（除 AI 解读），数据存在本设备

## 部署到 GitHub Pages

### 一次性步骤

1. 在 GitHub 建一个新 repo（比如 `guitar-journal`），可以是 public 或 private（GitHub Pro 才能 private + Pages）

2. 把这个文件夹里的所有文件推上去：
   ```bash
   cd guitar-journal-pwa
   git init
   git add .
   git commit -m "initial commit"
   git branch -M main
   git remote add origin https://github.com/<你的用户名>/guitar-journal.git
   git push -u origin main
   ```

3. 在 repo 的 **Settings → Pages**：
   - Source: `Deploy from a branch`
   - Branch: `main` / `/ (root)`
   - 点 Save

4. 等 1-2 分钟，会出现一个 URL 类似 `https://<你的用户名>.github.io/guitar-journal/`

### 在 iPhone 上安装

1. 在 **Safari**（不能用 Chrome 装 PWA）打开上面那个 URL
2. 点底部分享按钮 → "添加到主屏幕"
3. 主屏会出现拨片图标，点开即是全屏 app 体验

### 配置 API Key

第一次用 AI 功能时会弹窗要 Anthropic API key：

1. 去 https://console.anthropic.com/settings/keys 建一个 key
2. 复制粘贴进 app 里的输入框
3. Key 只存在你这台设备的 localStorage 里，从浏览器直接发到 api.anthropic.com，**不经过任何中间服务器**

> ⚠️ 安全提示：如果你把这个 repo 设为 public 并分享 URL，访问者只能用<u>他们自己</u>的 API key（每个设备独立存储）。但仍要小心：不要把你的 API key 提交到 git。

## 数据存储

| 数据类型 | 存储位置 | 大小限制 |
|---------|---------|---------|
| 练习记录、和弦、歌曲、录音元数据 | `localStorage` | 浏览器限制约 5-10MB |
| 录音音频文件 | `IndexedDB` | 每段 < 5MB，总容量较大 |
| API key | `localStorage`（key: `gj:anthropic_api_key`） | — |

**重要：iOS Safari 在 7 天无访问后可能清理 PWA 存储。**安装到主屏幕（standalone 模式）能缓解这个问题，但<u>仍建议定期点设置 → 导出数据 (JSON)</u> 做备份。

## 更新 App

修改代码后：
1. 编辑文件
2. **必须更新 `sw.js` 里的 `VERSION` 字符串**（比如 `v1` → `v2`），否则用户拿不到新版本
3. push 到 GitHub
4. 用户下次打开 app 时（在线状态）会自动拉新版本

## 文件结构

```
guitar-journal-pwa/
├── index.html         # 完整 app（HTML + CSS + JS 单文件）
├── manifest.json      # PWA 元数据
├── sw.js              # Service Worker（离线缓存）
├── .nojekyll          # 阻止 GitHub Pages 走 Jekyll 处理
├── icons/             # PWA / iOS 图标
│   ├── icon-192.png
│   ├── icon-512.png
│   ├── icon-maskable-512.png
│   ├── apple-touch-icon.png
│   ├── favicon-32.png
│   └── favicon-16.png
└── README.md
```

## 已知 iOS 限制

- **录音权限**：每次冷启动 app 第一次录音都会重新弹麦克风权限（iOS 标准行为）
- **后台运行**：PWA 切到后台时 JS 会暂停，长录音被打断的话录音会丢失。建议短录音（< 90 秒）
- **音频格式**：iOS Safari 的 MediaRecorder 输出 `audio/mp4`（不是 webm/opus），代码已自动处理
- **存储清理**：见上面 iOS 7 天规则
- **导出文件**：从设置导出 JSON，iOS PWA 会弹出"分享"或保存到"文件"

## 局限说明（请阅读）

音频分析的算法层面是「客观指标」级别的诊断，不是「专业老师听音」：

- ✓ **能**算出来的：BPM、节奏稳定性、平均音准偏差（cents）、力度均匀度、拨弦时间点
- ✗ **不能**算出来的：和弦音色是否干净、闷弦准确度、技巧表现力、音色好坏
- 单音 / 旋律模式音准检测最准；和弦扫弦模式跳过音高分析（多音不可靠）
- "可疑帧"只是启发式提示，不一定真的是问题

## 技术栈

- 单文件 HTML / CSS / vanilla JS
- Chart.js（从 CDN 加载，已被 service worker 缓存）
- Web Audio API + MediaRecorder
- 自实现：radix-2 FFT、YIN pitch detection、谱通量 onset detection、Hann 窗、RMS 包络
- IndexedDB（音频 blob）+ localStorage（元数据）

没有任何 build 步骤，没有依赖管理。改完 push 就完事。

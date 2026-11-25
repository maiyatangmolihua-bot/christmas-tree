# 圣诞树网页版 · 本地部署包

来源仓库：Hwescon/christmas-tree-v17.6（Grand Luxury Tree v17.6.5）
处理方式：把原本依赖 jsdelivr / Google Fonts / Google 模型仓库的部分全部下载到本地，改成本地路径。

## 和原版的区别

| 项目 | 原版 | 这个包 |
|---|---|---|
| three.js | jsdelivr CDN | `assets/three/` 本地 |
| MediaPipe 手势 | jsdelivr + Google Storage | `assets/mediapipe/`、`assets/models/` 本地 |
| 字体（5 个 Google 字体） | fonts.googleapis.com | `assets/fonts/` 本地（104 个 woff2 分片） |
| 外网依赖 | 有（国内 jsdelivr 基本打不开） | **0**，完全离线可用 |
| 摄像头/手势 | 打开页面就请求权限并下载 25MB 模型 | 首次点击/按键后才启动，先看树再说 |

## 本地预览

```powershell
# 在解压后的目录里
python -m http.server 8188 --bind 127.0.0.1
# 浏览器打开
start http://127.0.0.1:8188/
```

> 必须走 http 或 https，直接双击 index.html（file://）会因为浏览器模块 CORS 限制白屏。

## 局域网分享（同一个 WiFi / 同一路由器）

```powershell
python -m http.server 8188 --bind 0.0.0.0
```

然后让别人访问 `http://<你的内网IP>:8188/`。
查内网 IP：`ipconfig`，看无线网卡那一段的 IPv4 地址。

注意：
- 第一次可能被 Windows 防火墙挡住，弹窗选"允许"，或用管理员执行：
  `New-NetFirewallRule -DisplayName "Tree 8188" -Direction Inbound -Protocol TCP -LocalPort 8188 -Action Allow`
- 局域网 http 下浏览器**不给摄像头权限**（非安全上下文），别人只能看树、用鼠标/键盘互动；手势要 https。

## 公网部署（真正能"点进去就看到"）

### 方案 A：GitHub Pages（推荐，仓库本来就在 GitHub 上）

1. 新建仓库（或直接用现有的 `Hwescon/christmas-tree-v17.6`）
2. 把本目录所有文件推到仓库根目录（`index.html` 必须在根）
3. 仓库 Settings → Pages → Source 选 `Deploy from a branch` → 分支 `main`、目录 `/ (root)` → Save
4. 等 1–2 分钟，访问 `https://<用户名>.github.io/<仓库名>/`

命令行版（装了 git 之后）：

```bash
git init
git add .
git commit -m "self-hosted christmas tree"
git branch -M main
git remote add origin https://github.com/<用户名>/<仓库名>.git
git push -u origin main
```

### 方案 B：Netlify Drop（不用命令行）

1. 打开 https://app.netlify.com/drop
2. 把这个文件夹整个拖进去
3. 几秒后得到一个 `https://xxx.netlify.app` 链接，可直接发人

### 方案 C：Cloudflare Pages / Vercel

同样是把文件夹当作静态站点上传，构建命令留空、输出目录填 `/` 即可。

### 分享时注意

- 公网必须是 **https**，否则摄像头不可用（手势失效，但树和鼠标互动正常）
- 手机上打开会自适应竖屏，效果和桌面不同但可用
- 页面文件合计约 30MB（其中 17MB 是手势 wasm、7.6MB 是手部模型），**只有访客触发摄像头才会下载**；纯看树的首屏只有约 5MB

## 目录结构

```
index.html                      单文件页面（CSS/JS 都内联）
assets/three/                   three.js r160 + 用到的 addons
assets/mediapipe/               手势识别 ESM + wasm
assets/models/hand_landmarker.task
assets/fonts/                   5 个字体 + fonts.css
```

## 改内容

页面左侧面板可直接改：祝福语两行、字体、大小、颜色、粒子密度、雪花、背景音乐（本地上传）、照片（上传会存 IndexedDB，只存在访客自己浏览器里）。
快捷键：空格聚合 / Z 散开 / X 抓照片 / 方向键旋转 / H 隐藏界面。

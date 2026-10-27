# 3D粒子交互系统

一个基于Three.js的实时3D粒子交互系统，支持手势控制、音乐可视化和自定义形状。

## ✨ 本仓库改动（相对上游）

本仓库在上游基础上做了两批工程优化，**功能与观感保持不变**，改动集中在性能与可维护性：

### 渲染与性能

- **粒子计算下沉到 shader**：缩放过渡与逐粒子错峰改由 vertex shader 完成。每帧 CPU 只更新 5 个 uniform，不再遍历上万个粒子、也不再每帧上传约 120KB 的位置缓冲
- **移除纹理依赖**：圆形光斑改用 `gl_PointCoord` 程序化生成，连纹理对象都不再创建
- **像素比封顶 2x**：高分屏下避免 GPU 填充率成倍上涨
- **手势检测节流**：MediaPipe 从「每帧推理」改为约 15fps，并加了忙等保护，不再与渲染抢主线程

### 结构

- **消除重复代码**：`index2.html` 原内联了 1400+ 行与 `script.js` 雷同（且已漂移）的脚本，现统一引用外部文件
- **依赖本地化**：Three.js / TensorFlow.js / HandPoseDetection / MediaPipe / Font Awesome 全部放入 `vendor/`，首屏不再依赖外网 CDN
- **模型多源回退**：MediaPipe 模型按 `vendor/mediapipe` → jsdelivr → unpkg 顺序尝试，单点 CDN 故障不会直接掉到模拟手势

### 修复

- `alphaTest` 由 `0.5` 调至 `0.01`：原值会把径向渐变的柔和边缘切成硬边圆盘
- `server.py`：`ssl.wrap_socket` 在 Python 3.12+ 已被移除，改用 `ssl.SSLContext` —— HTTPS 模式在新版本 Python 上恢复可用
- `server.py`：端口绑定失败时提示真实端口号，并补充 Windows 10013（Hyper-V 保留端口段）的排查指引
- `server.py`：生成 `musicList.js` 时固定行尾，避免跨平台产生整文件 diff

## 🎯 功能特性

![本地截图](images/1.png)
![本地截图](images/2.png)

### 交互控制
- **手势控制**：通过摄像头检测双手张合控制粒子缩放与扩散
- **鼠标交互**：支持鼠标拖动旋转视角、滚轮缩放
- **形状选择**：提供多种预设形状（爱心、花朵、土星、球体、星星等）
- **自定义形状**：支持上传图片生成粒子排列

### 视觉效果
- **3D粒子系统**：实时渲染10000+粒子，支持形状变形
- **渐变色彩**：可自定义双色调渐变效果
- **拖尾效果**：粒子运动时产生平滑拖尾
- **内部粒子填充**：支持形状内部粒子填充

### 音乐功能
- **音乐播放**：支持选择和播放本地音乐文件
- **自动扫描**：服务器启动时自动扫描music文件夹生成播放列表
- **进度控制**：音乐播放进度条和时间显示

### 系统特性
- **响应式设计**：适配不同屏幕尺寸
- **全屏模式**：支持一键进入全屏体验
- **HTTPS支持**：确保摄像头等敏感功能正常工作
- **并发处理**：使用多线程服务器处理并发请求
- **跨域支持**：适配映射/内网穿透场景

## 🛠️ 技术栈

### 前端技术
- **Three.js r128**：3D图形渲染
- **TensorFlow.js**：机器学习框架
- **HandPoseDetection**：手势检测
- **MediaPipe Hands**：手部关键点识别
- **JavaScript (ES6+)**：前端交互逻辑
- **HTML5/CSS3**：页面结构和样式

### 后端技术
- **Python 3**：服务器端开发
- **http.server**：Web服务器
- **socketserver**：多线程支持
- **Pathlib**：文件系统操作
- **SSL/TLS**：HTTPS支持

## 📁 项目结构

```
├── README.md             # 项目说明文档
├── index1.html           # 主页面（版本1，引用外部 script.js）
├── index2.html           # 主页（默认入口，同样引用外部 script.js）
├── script.js             # 3D粒子系统核心逻辑（两份页面共用）
├── styles.css            # 样式文件
├── musicList.js          # 音乐列表配置（自动生成）
├── server.py             # Web服务器（支持HTTPS、并发）
├── vendor/               # 前端依赖（本地化，无需联网）
│   ├── three.min.js
│   ├── tf.min.js
│   ├── hand-pose-detection.min.js
│   ├── hands.min.js
│   └── fontawesome/      # 图标字体（css + webfonts）
├── images/               # 项目截图和资源图片
│   ├── 1.png             # 系统界面截图1
│   └── 2.png             # 系统界面截图2
├── music/                # 音乐文件目录
│   ├── 【Shakira】疯狂动物城1主题曲《Try Everything》.mp3
│   ├── 【Shakira】疯狂动物城2主题曲《Zoo》.mp3
│   └── 【无损音质】《最初的记忆 (DJ版)》- DJ阿智 Remix.mp3
```

## 🚀 快速开始

### 环境要求
- **Python 3.6+**：运行服务器
- **现代浏览器**：Chrome 90+、Firefox 88+、Safari 14+（支持WebGL和MediaStream）
- **摄像头**：用于手势控制（可选）

### 启动服务器

1. **进入项目目录**：
   ```bash
   cd Gesture-Particle-Interaction-Effects
   ```

2. **运行服务器（推荐方式）**：
   ```bash
   python server.py -p 8765 --no-https
   ```

   推荐先用这个组合，原因见下：

   - **端口**：默认 `80` 需要管理员权限；部分 Windows 机器上 `8000`/`8080` 会落在 Hyper-V 保留段内，绑定直接失败（错误码 10013）。用 `netsh interface ipv4 show excludedportrange protocol=tcp` 可查看保留段
   - **协议**：`localhost` 本身属于安全上下文，**HTTP 下摄像头同样可用**；只有需要通过局域网 IP 访问时才必须用 HTTPS

   启动后服务器会自动扫描 `music` 文件夹生成 `musicList.js`。

3. **访问系统**：
   - `http://localhost:8765`

   需要 HTTPS 时直接去掉 `--no-https`（首次运行会用 openssl 生成自签名证书）：
   ```bash
   python server.py -p 8765
   ```

### 命令行参数

```bash
# 使用自定义端口
python server.py -p 8765

# 不使用HTTPS（本地 localhost 调试推荐，摄像头仍可用）
python server.py -p 8765 --no-https
```

## 🎮 使用说明

### 基本操作
1. **选择形状**：点击左侧控制面板的形状按钮
2. **播放音乐**：在音乐选择框中选择音乐，点击播放按钮
3. **手势控制**：
   - 打开摄像头权限
   - 将双手放在摄像头前，张合手掌控制粒子缩放
   - 保持双手距离适中，系统会自动检测

### 高级功能
1. **上传图片**：点击"上传图片生成形状"按钮，选择图片
2. **全屏模式**：点击右上角全屏按钮
3. **隐藏/显示控制面板**：点击左侧切换按钮
4. **调整粒子参数**：在控制面板中调整粒子数量、大小、颜色等

## ⚙️ 配置说明

### 服务器配置（server.py）

```python
# 服务器端口
DEFAULT_PORT = 80

# 是否默认使用HTTPS
DEFAULT_USE_HTTPS = True

# 服务器绑定地址
SERVER_HOST = "0.0.0.0"

# 请求队列大小
REQUEST_QUEUE_SIZE = 50

# 默认HTML文件
DEFAULT_HTML_FILE = "index2.html"
```

### 音乐配置（自动生成）

服务器启动时会自动扫描`music`文件夹，并生成`musicList.js`文件，无需手动配置。

### 前端配置（script.js）

```javascript
// 粒子数量（可在控制面板调整）
let particleCount = 10000;

// 默认粒子大小
let particleSize = 0.33;

// 默认颜色
let color1 = '#ff6ec7';
let color2 = '#7873f5';

// 默认形状
let currentShape = 'heart';
```

## 📋 注意事项

### 浏览器兼容性
- 推荐使用Chrome浏览器以获得最佳体验
- Firefox和Safari也能正常运行，但部分功能可能受限
- 旧版本浏览器可能不支持WebGL或MediaStream API

### 摄像头权限
- 首次使用需要允许浏览器访问摄像头
- 使用HTTPS协议是摄像头权限的必要条件
- 若摄像头无法启动，请检查浏览器权限设置

### 性能优化
- 粒子数量建议根据设备性能调整（5000-20000）
- 低配置设备可降低粒子数量以提高帧率
- 关闭拖尾效果可提升性能

### 音乐文件
- 仅支持MP3格式的音乐文件
- 音乐文件需放在`music`文件夹中
- 文件名建议使用UTF-8编码

## 🛡️ 安全说明

- 服务器使用自签名SSL证书，首次访问浏览器会提示安全风险，点击"高级"-"继续访问"即可
- 本项目仅用于本地或局域网演示，不建议部署在公网环境
- 请勿在生产环境使用自签名证书

## 📝 更新日志

### v1.0.0
- 初始版本发布
- 实现基础3D粒子系统
- 支持手势控制功能
- 添加音乐播放功能

### v1.1.0
- 新增自定义形状功能
- 支持图片上传生成粒子形状
- 优化性能和用户体验

### v1.2.0
- 实现自动音乐列表生成
- 添加HTTPS支持
- 优化服务器端代码
- 支持映射/内网穿透

### v1.3.0
- 粒子缩放计算下沉到 vertex shader，移除每帧 O(粒子数) 的 CPU 遍历与位置缓冲上传
- 圆形光斑改为程序化生成，移除纹理依赖
- 像素比封顶 2x；手势检测节流至约 15fps
- 删除 `index2.html` 中重复的内联脚本，两个页面统一引用 `script.js`
- 前端依赖本地化到 `vendor/`，MediaPipe 模型支持多源回退
- 修复 `alphaTest` 导致的粒子硬边问题
- 修复 Python 3.12+ 下 HTTPS 不可用（`ssl.wrap_socket` 已移除）
- 改进端口绑定失败的错误提示与排查指引

## 🤝 贡献

欢迎提交Issue和Pull Request来帮助改进这个项目！

## 📄 许可证

本项目采用MIT许可证，详见LICENSE文件。

## 🙏 致谢

- [Three.js](https://threejs.org/) - 3D图形库
- [TensorFlow.js](https://www.tensorflow.org/js) - 机器学习框架
- [MediaPipe](https://mediapipe.dev/) - 手势检测技术
- 所有开源社区的贡献者们

---

**享受3D粒子交互的乐趣！** 🎉

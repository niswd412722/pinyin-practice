# 拼音打字练习

练习拼音输入的打字工具 iOS App。

## 功能模式

| 模式 | 说明 |
|------|------|
| **看字打字** | 显示汉字，输入对应拼音 |
| **声母韵母** | 先打声母，再打韵母，分两步 |
| **韵母专项** | 专门练习韵母（ü/ong/iang 等易混淆韵母） |
| **听写** | 显示拼音，输入汉字 |
| **错题本** | 收录答错的题，反复练习 |

## 难度

- **简单**：单音节词（妈妈、学习）
- **常用**：双音节常用词（北京、天气）
- **成语**：四字成语

## 使用方法

1. 打开 App 直接开始
2. 顶部选择模式和难度
3. 输入框输入答案，回车或按"下一题"
4. 不用输入声调

## 安装

### 方式一：自己编译
```bash
npm install
npx cap sync ios
# 用 Xcode 打开 ios/App/App.xcodeproj
# Product → Archive → Distribute
```

### 方式二：用在线签名平台
1. 下载最新 zip 包
2. 上传到 [蒲公英](https://pgyer.com) 或 [香蕉云签名](https://www.pgyer.com/doc/香蕉云签名)
3. 下载签名后的 IPA 安装

### 方式三：GitHub Actions 自动构建
推送代码后自动构建 IPA，下载 artifact。

## 技术栈

- 前端: HTML + CSS + JS（无框架依赖）
- iOS 容器: Capacitor
- 触控反馈: Web Audio API（内嵌，无需外部音频文件）
- 振动反馈: Capacitor Haptics 插件

## 项目结构

```
pinyin-practice/
├── public/index.html       # 完整应用代码（HTML/CSS/JS）
├── ios/App/               # iOS 原生项目
│   └── App/public/index.html  # 同步后的 web assets
├── server.js              # 本地调试服务器
└── SPEC.md                # 设计规格
```

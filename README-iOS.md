# 拼音打字练习 - iOS 项目

## 如何打包签名

### 方式一：在线签名平台（推荐）
使用以下平台之一上传本项目 zip 包：
- 蒲公英：https://pgyer.com
- 香蕉云签名：https://www.pgyer.com/doc/香蕉云签名
- Fir.im：https://fir.im

**上传时选择 iOS 项目**，平台会在云端 Mac 完成编译和签名。

### 方式二：使用 Xcode（需要 Mac）
1. 解压 zip
2. 用 Xcode 打开 `ios/App/App.xcodeproj`
3. 选择签名团队（Sign with your team）
4. Product → Archive → Distribute

### 在线构建步骤
1. 将 zip 上传到 GitHub（私有仓库）
2. 在在线签名平台关联 GitHub 仓库
3. 平台自动拉取并编译

## 项目信息
- App ID: com.pinyin.practice
- App 名称: 拼音打字练习
- 最低 iOS 版本: 13.0

## 技术栈
- 前端: Capacitor + 原生 HTML/CSS/JS
- iOS 容器: Capacitor iOS

## 界面特性
- 看字打字 / 听写 / 速度模式
- 简单 / 常用 / 成语 三个难度
- 声母韵母参考表
- 实时反馈
- 手机适配

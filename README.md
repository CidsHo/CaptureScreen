# 捕捉Quest3中的摄像头截图画面并回传

*仅测试在Unity上开发Meta Quest应用的情况*

*Last Available in 7.13.2024*

在Unity中使用ScreenShotLoader插件来捕获摄像头画面，将设备拍摄到的截屏图像传出。

因为Quest3目前没有开放摄像头权限，无法使用实时视频流来实现识别操作，故出此下策来实现一些功能。

## 0. 准备
将ScreenShotLoader（见附件）安装至Assets下，同时载入Meta XR All-in-one SDK(也可以视乎需要安装），完成环境配置。

## 1. 配置插件


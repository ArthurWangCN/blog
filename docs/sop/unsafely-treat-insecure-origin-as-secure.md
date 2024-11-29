# 新版chrome 解决在http协议下无法调用摄像头和麦克风的问题（不安全）

解决办法：

在浏览器地址栏中输入 `chrome://flags/#unsafely-treat-insecure-origin-as-secure`，回车，将该选项置为Enabled，

在输入框中输入需要访问的地址，多个地址使用“,”隔开，

然后点击右下角弹出的Relaunch按钮，

自动重启浏览器之后就可以在添加的http地址下调用摄像头和麦克风了。

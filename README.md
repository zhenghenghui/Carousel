### 后端代码地址：https://github.com/zhenghenghui/fileParsed.git
### 实现的是 axios 与 django 之间的交互

### 1、完成轮播图基本布局   
### 2、借助 navigator.mediaDevices.getUserMedia 完成前端录音功能，并生成 .wav 音频文件
### 3、使用 axios 发送网络请求，参数为 .wav 音频文件
### 4、将 .wav 音频文件转换为 .pcm 文件
### 5、借助科大讯飞提供的 SDK 实现语音识别
### 6、django 返回语音识别文本
### 7、前端拿到相应文本后判断是否包含“上一张”或“下一张”，并触发对应事件
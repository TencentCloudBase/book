# AI智能图象增值服务

## 功能概述

本文对应实现例子为[tcb-demo-ai](https://github.com/TencentCloudBase/tcb-demo-ai)。本增值服务，AI智能图像能力是借助了腾讯云的[人脸识别](https://cloud.tencent.com/product/facerecognition/developer)、[人脸核身](https://cloud.tencent.com/product/faceid/developer)和[人脸融合](https://cloud.tencent.com/product/facefusion/developer)功能，通过云开发的云函数和存储简化了素材的存储、配置的拉取和服务的调用 [tcb-servicve-sdk](https://github.com/TencentCloudBase/tcb-service-sdk) 来实现。

后续还会陆续支持[智能鉴黄](https://cloud.tencent.com/product/pornidentification)、[图片标签](https://cloud.tencent.com/product/image-tag)、[文字识别 OCR](https://cloud.tencent.com/product/ocr)等服务。

## 体验
<p align="center">
    <img src="https://main.qcloudimg.com/raw/fb6e0a10f9870824e19b06170f4ed558.png" width="500px">
    <p align="center">扫码体验</p>
</p>

## DEMO 源码
本章的案例代码，是在 [tcb-demo-ai](https://github.com/TencentCloudBase/cloudbase-examples/tree/master/miniprogram/tcb-demo-ai)。

## DEMO 接入流程
1. 请使用[微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)打开 `DEMO` 源码，在根目录下的 `project.config.json` 文件，填写您的小程序 `appid`。

2. 通过此[微信小程序第三方绑定流程](https://www.qcloud.com/login/mp?s_url=https%3A%2F%2Fconsole.cloud.tencent.com%2Fcam%2Fcapi)登录小程序对应的腾讯云帐号(需要小程序管理员权限)，然后在[云API密钥](https://console.cloud.tencent.com/cam/capi) 里获取 `SecretId` 和 `SecretKey`。

3. 在腾讯云的[智能图像控制台](https://console.cloud.tencent.com/ai)，开通相应的服务：

<p align="center">
    <img src="https://main.qcloudimg.com/raw/d4d30dff15fdb69f30f18c450b7274f6.png" width="1000px">
    <p align="center">开通服务</p>
</p>

4. 本案例，前端页面(`client/pages/`)和云函数(`cloud/functions`)一一对应，如下：

|功能|前端页面|云函数
|--|--|--
|人脸融合|face-fusion|FaceFusion
|活体人脸检测|liveness-recognition|GetLiveCode 和 LivenessRecognition
|身份信息认证|idcard-verification|IdCardVerification
|人脸检测与分析|face-detect|DetectFace
|通用印刷体识别|ocr-general|GeneralBasicOCR
|身份证识别|ocr-idcard|IDCardOCR


>! 如果需要体验某个功能，需要在对应的云函数里参照 `config/example.js` 新建 `config/index.js`，并填入上面拿到的`SecretID` 和 `SecretKey`，然后创建并部署云函数。

5. 如果是体验以下的功能，还需要做额外的准备工作：

### 人脸融合
如果想体验人脸融合，开通服务后，需要【创建活动】并【添加素材】，要获得以下配置：

* `ProjectId` (活动 ID)，可在[人脸融合控制台](https://console.cloud.tencent.com/ai/facemerge/index)中查看
* `ModelId` (素材 ID)，可在[人脸融合控制台](https://console.cloud.tencent.com/ai/facemerge/index)中查看

<p align="center">
    <img src="https://main.qcloudimg.com/raw/a10ec6cdc6400d94535ec5b76a0a01a3.png" width="800px">
    <p align="center">人脸融合--活动管理面板</p>
</p>

<p align="center">
    <img src="https://main.qcloudimg.com/raw/06ffe29cb8f924aeff9e04cadca884fe.png" width="800px">
    <p align="center">人脸融合--活动素材管理面板</p>
</p>

### 人脸识别

人脸识别已经升级至全新的版，控制台地址请访问：[人脸识别新控制台](https://console.cloud.tencent.com/aiface)。

如果你进入了旧版控制台，你会看到顶部有提示可以跳转到新版：

<p align="center">
    <img src="https://main.qcloudimg.com/raw/2fc998f815138fb90f6f45be293b613f.png" width="600px">
    <p align="center">人脸识别新版控制台跳转提示</p>
</p>

## 源码介绍

### 活体人脸核身

本案例实现了该服务的一些基础能力。整个逻辑流程如下：

<p align="center">
    <img src="https://main.qcloudimg.com/raw/dd1666f2e7f9b18f9ac009ef0c46dce1.png" width="600px">
    <p align="center">实现逻辑</p>
</p>

其中云函数 `LivenessRecognition` 的大体逻辑如下：

```js
const {
  IdCard,
  Name,
  VideoFileID,
  LivenessType = 'SILENT',
  ValidateData
} = event

try {
  // 获取视频内容的字符合串
  let fileContent = await tcbService.utils.getContent({
      fileID: VideoFileID
  })

  if (!fileContent) {
      return { code: 10002, message: 'fileContent is empty' }
  }

  const result = await tcbService.callService({
      service: 'ai',
      action: 'LivenessRecognition',
      data: {
      IdCard,
        Name,
        VideoBase64: fileContent.toString('base64'), // 视频内容转 base64
        LivenessType,
        ValidateData
      },
      options: {
        secretID: SecretID,
        secretKey: SecretKey // 调用其它腾讯云账号的 AI 资源
    }
  })

  return result
}
catch (e) {
  return { code: 10001, message: e.message }
}
```

在小程序端，需要有类似的遮罩，才能提供视频通过的概率。
<p align="center">
    <img src="https://main.qcloudimg.com/raw/c4991650a6adc8b21ffe619a5756788d.png" width="400px">
    <p align="center">视频遮罩</p>
</p>

那在小程序端怎么可以让图片等元素，盖在 `<camera>`, `<video>` 等原生的组件上面呢？答案是使用 `<cover-view>` 和 `<cover-image>`，譬如：

```html
<camera
    device-position="front"
    flash="off"
    binderror="error"
>
    <cover-view class="camera-cover">
        <cover-image 
            class="camera-image"    
            src="image path"
        >
        </cover-image>
    </cover-view>
    <cover-view
      class="number"
      wx-if="{{isRecording}}"
    >
        请念数字：{{number}}
  </cover-view>
</camera>
```

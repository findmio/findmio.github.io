---
title: "AutoX.js 获取手机截图上传至 OSS"
date: 2024-04-15T19:21:55+08:00
draft: false
tags: ["autox.js", "rpa", "阿里云"]
---

## 前言

[AutoX.js](https://github.com/kkevsekk1/AutoX) 使用的是 [Rhino](https://github.com/mozilla/rhino) 作为 `JavaScript` 引擎，并不支持完整的 `nodejs` 模块，所以没法直接调用 `nodejs` 下的 [OSS](https://www.npmjs.com/package/oss) 包。

下面展示使用 [OSS Android SDK](https://help.aliyun.com/zh/oss/developer-reference/installation-1) 完成上传的过程

## 准备工作

1. 前往 [OSS Android SDK](https://help.aliyun.com/zh/oss/developer-reference/installation-1)，下载依赖包

2. 将 `aar` 格式的 `SDK` 转换为 `dex`

这里使用 Android Studio 提供的工具 `d8`，进入到 Android Studio 安装目录下，`d8` 一般在 `build-tools` 下，`MacOS` 下为 `~/Library/Android/sdk/build-tools/{version}/`

```bash
./d8 --release --output output_folder oss-android-sdk-2.9.13.aar
```

会生成一个 `classes.dex` 的文件

3. 拷贝 `classes.dex` 到手机

## 截图

```javascript
// 安卓版本高于 Android 9，需要手动确认授权
if (device.sdkInt > 28) {
    // 等待截屏权限申请并同意
    threads.start(function () {
        // 这里的系统提示框可能不是运行，改成对应文本即可
        packageName("com.android.systemui").text("运行").waitFor();
        text("运行").click();
    });
}

// 申请截屏权限，false 竖屏
if (!requestScreenCapture(false)) {
    console.error("请求截图失败");
    exit();
}

// 获取到截图
var image = images.captureScreen();
```

## 上传

这里使用了字节数组的方式上传，也可以保存到本地，再上传

```javascript
// 使用绝对路径导入 SDK
var dexPath = files.path("/storage/emulated/0/Download/classes.dex");
runtime.loadDex(dexPath);

// 导入 OSS 中的类
importClass(com.alibaba.sdk.android.oss.ClientConfiguration);
importClass(com.alibaba.sdk.android.oss.ClientException);
importClass(com.alibaba.sdk.android.oss.OSS);
importClass(com.alibaba.sdk.android.oss.OSSClient);
importClass(com.alibaba.sdk.android.oss.ServiceException);
importClass(com.alibaba.sdk.android.oss.common.auth.OSSCredentialProvider);
importClass(
    com.alibaba.sdk.android.oss.common.auth.OSSStsTokenCredentialProvider
);
importClass(com.alibaba.sdk.android.oss.model.PutObjectRequest);
importClass(com.alibaba.sdk.android.oss.model.PutObjectResult);

// 下面的参数是在阿里云生成
var OSS_ENDPOINT = "yourEndpoint";
var OSS_ACCESS_KEY_ID = "yourAccessKeyId";
var OSS_ACCESS_KEY_SECRET = "yourAccessKeySecret";
var bucketName = "yourBucket";

var credentialProvider = new OSSStsTokenCredentialProvider(
    OSS_ACCESS_KEY_ID,
    OSS_ACCESS_KEY_SECRET,
    ""
);

// OSSClient 的配置类
var conf = new ClientConfiguration();
conf.setConnectionTimeout(60000); // 建立连接的超时时间，单位为毫秒。默认为60000毫秒
conf.setSocketTimeout(60000); // Socket层传输数据的超时时间，单位为毫秒。默认为60000毫秒
conf.setMaxConcurrentRequest(5); // 最大并发数。默认为5
conf.setMaxErrorRetry(2); // 请求失败后最大的重试次数。默认2次

var oss = new OSSClient(context, OSS_ENDPOINT, credentialProvider, conf);

// 文件名
var filename = "screenshot.jpg";

// 使用字节数组上传
var imageBytes = images.toBytes(image, "jpg", 100);

var put = new PutObjectRequest(bucketName, filename, imageBytes);

putResult = oss.putObject(put);

// 上传后的链接
var ossUrl = `https://${bucketName}.${OSS_ENDPOINT}/${filename}`;

// 上传成功
if (putResult.getStatusCode() === 200) {
    console.log(ossUrl);
}
```

## 参考链接

-   [AutoX.js v6.5.9](https://github.com/kkevsekk1/AutoX/releases/tag/6.5.9)
-   [aliyun-oss-sdk-android-2.9.13.aar](https://help-static-aliyun-doc.aliyuncs.com/file-manage-files/zh-CN/20230420/jsol/oss-android-sdk-2.9.13.aar)
-   [OSSClent 配置](https://help.aliyun.com/zh/oss/developer-reference/initialization-1#section-755-5xq-f9c)
-   [OSS 创建 AccessKey](https://help.aliyun.com/document_detail/53045.html)

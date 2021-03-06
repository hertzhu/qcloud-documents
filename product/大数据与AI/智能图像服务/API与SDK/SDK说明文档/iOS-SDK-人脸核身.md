## 开发准备

### SDK 获取

人脸核身的 iOS SDK 下载地址：[iOS SDK](https://mc.qcloudimg.com/static/archive/71b1c0bcb84ba926866af194d9f9743e/CISDK_1.0.2.zip) 。

### 开发准备
-  iOS 8.0+ ；
-  手机必须要有网络（ GPRS 、3G 或 Wifi 网络等）；
-  开发者使用人脸识别功能前，需要先进行 [腾讯云账号注册](https://cloud.tencent.com/register)（详细指引请参考 [注册腾讯云](https://cloud.tencent.com/document/product/378/9603)），并 [创建存储桶](https://cloud.tencent.com/document/product/460/10637)，从而获得 APPID 、SecretId 和 SecretKey 等（获取 APPID 可参考 [域名管理](https://cloud.tencent.com/document/product/460/6937)）。
### SDK 配置
#### SDK 导入

万象优图的 iOS SDK 压缩包（CIClientSDK.zip）由一个 .a 静态库和一个包含头文件的文件夹 Headers 组成，如下图所示。上传包提供了**支持 bitcode **与**不支持 bitcode **两个版本，可根据业务需要进行选择。
![上传SDK](https://mccdn.qcloud.com/static/img/05f5a1d6768985aa11b23c3808914989/image.png)
![下载SDK](https://mccdn.qcloud.com/static/img/190e5c8c4920ba4d7334f7ba64fd3839/image.png)
将解压后的 CISDK 拖入工程目录，Xcode 会自动将其加入链接库列表中。
![导入 SDK 包](https://mccdn.qcloud.com/static/img/96dda4e5f2e4f8fab3fbda3de1cd8e25/image.png)

#### 工程配置
在 Build Settings 中设置 Other Linker Flags，加入参数 -ObjC。
![参数配置](https://mccdn.qcloud.com/static/img/58327ba5d83809c77da158ff95627ef7/image.png)
在工程 info.plist 文件中添加 App Transport Security Settings 类型，然后在 App Transport Security Settings 下添加 Allow Arbitrary Loads 类型 Boolean ，值设为‘YES’。
## 签名获取
**签名类型：**

| 类型   | 含义            |
| ---- | ------------- |
| 多次有效 | 有效时间内多次始终有效   |
| 单次有效 | 与资源 URL 绑定，一次有效 |

**签名获取：**
移动端 SDK 中用到的签名，推荐使用服务器端 SDK，并由移动端向业务服务器请求。SIGN 的具体生成和使用请参照 [签名与鉴权](/document/product/641/12409) 。

## 初始化
引入上传 SDK 的头文件 ‘CIClient .h’，使用目录操作时，需要先实例化 CIClient  对象。
#### 方法原型
```objective-c 
-(instancetype)initWithAppId:(NSString*)appId```


#### 参数说明

| 参数名称  | 类型         | 是否必填 | 说明            |
| ----- | ---------- | ---- | ------------- |
| APPID | NSString * | 是    | 项目 ID，即APP ID。 |
#### 示例
```objective-c
CIClient *client= [[CIClient alloc] initWithAppId:appId]```
## 快速入门
这里演示的对比用户人脸身份照片的基本流程，更多细节可以参考 demo；在进行这一步之前必须在腾讯云控制台上申请万象业务的 APPID。

### STEP - 1 初始化 CIClient
#### 示例
```objective-c
CIClient *client= [[CIClient alloc] initWithAppId:appId]```
### STEP - 2 初始化对比人脸身份照片的 task

在这里我们假设您已经申请了自己业务的存储桶。SDK 所有的任务对应了相应的 task，只要把相应的 task 参数传递给 client，就可以完成相应的动作；这里以创建人任务来演示
#### 示例

```objective-c
 CIIDCardCompareTask *task = [CIIDCardCompareTask new];
    task.filePath = path;
    task.bucket = bucket;
    task.idcardNumber = number;
    task.idcardName = name;
    task.sign = _sign;
    CIClient *client= [[CIClient alloc] initWithAppId:appId];
    client.completionHandler = ^(CIITaskRsp *resp, NSDictionary *context){
        if (resp.retCode == 0) {
         //sucess
        }else{}
    };
    client.progressHandler = ^(NSInteger bytesWritten,NSInteger totalBytesWritten,NSInteger totalBytesExpectedToWrite){
          //progress
    };
    [client IDCardCompare:task]；
```

### 对比人脸身份照片
自带人脸识别数据库，可实时为国内公民提供真实身份信息核验，具体步骤如下：
1. 实例化 CIIDCardCompareTask  对象；
2. 调用 CIClient 的 IDCardCompare 方法，将 CIIDCardCompareTask 对象传入。
3. 通过 CITaskRsp 的对象返回结果

#### CIICreatPersonTask 的属性说明

| 参数名称         | 类型         | 是否必填 | 说明                                       |
| ------------ | ---------- | ---- | ---------------------------------------- |
| filePath     | NSString * | 是    | 图片的路径                                    |
| photoUrl     | NSString * | 是    | 图片的 url , filePath 和 photoUrl 只提供一个即可,如果都提供,只使用 url |
| idcardNumber | NSString * | 是    | 用户身份证号码                                  |
| idcardName   | NSString * | 否    | 用户身份证姓名                                  |
| bucket       | NSString * | 是    | 目录所属存储桶名称                           |
| sign         | NSString * | 是    | 签名                                       |
| sessionId    | NSString * | 否    | 相应请求的 session 标识符，可用于结果查                   |

#### 返回结果说明

通过 CITaskRsp 的对象返回结果

| 属性名称    | 类型             | 说明       |
| ------- | -------------- | -------- |
| retCode | int            | 任务描述代码   |
| descMsg | NSString *     | 任务描述信息   |
| data    | NSDictionary * | 任务请求返回数据 |

#### 示例

```objective-c

  CIIDCardCompareTask *cm = [CIIDCardCompareTask new];
    task.filePath = path;
    task.bucket = bucket;
    task.idcardNumber = number;
    task.idcardName = name;
    task.sign = _sign;
    CIClient *client= [[CIClient alloc] initWithAppId:appId ];
    client.completionHandler = ^(CIITaskRsp *resp, NSDictionary *context){
        if (resp.retCode == 0) {
			//sucess
        }else{
			//fail
		}
    };
    [client IDCardCompare:cm];
```

### 获取唇语

通过调用此接口获取一个唇语验证字符串，用于用户录制视频时使用，具体步骤如下：

1. 实例化 CIGetLipIdentificationStringTask  对象；
2. 调用 CIClient  的 getLipIdentificationString 方法，将 CIGetLipIdentificationStringTask 对象传入；
3. 通过 CITaskRsp 对象返回结果。

#### CIGetLipIdentificationStringTask 参数说明

| 参数名称   | 类型         | 是否必填 | 说明             |
| ------ | ---------- | ---- | -------------- |
| seq    | NSString * | 否    | 标识请求序列号        |
| bucket | NSString * | 是    | 目录所属存储桶名称 |
| sign   | NSString * | 是    | 签名             |

#### 返回结果说明

通过 CITaskRsp 的对象返回结果

| 属性名称    | 类型             | 说明       |
| ------- | -------------- | -------- |
| retCode | int            | 任务描述代码   |
| descMsg | NSString *     | 任务描述信息   |
| data    | NSDictionary * | 任务请求返回数据 |

#### 示例

```objective-c

    CIIDCardCompareTask *cm = [CIIDCardCompareTask new];
    cm.bucket = bucket;
    cm.sign = _sign;/
    CIClient *client= [[CIClient alloc] initWithAppId:appId];
    client.completionHandler = ^(CIITaskRsp *resp, NSDictionary *context){
        if (resp.retCode == 0) {
            //sucess
        }else{}
    };
    [client getLipIdentificationString:cm];
```
###  人脸核身验证
通过调用此接口根据用户上传的照片和视频，进行人脸核身验证，具体步骤如下：

1.实例化 CILipIdentificationCompareTask  对象；
2.调用 CIClient  的 lipIdentificationCompare 方法，将 CILipIdentificationCompareTask 对象传入；
3.通过 CITaskRsp的 对象返回结果信息。

#### CILipIdentificationCompareTask参数说明

| 参数名称        | 类型         | 是否必填 | 说明                                       |
| ----------- | ---------- | ---- | ---------------------------------------- |
| videoPath   | NSString * | 是    | 视频的路径                                    |
| photoPath   | NSString * | 是    | 与视频对比的图片                                 |
| validate    | NSString * | 是    | 执行 CIGetLipIdentificationStringTask 返回的唇语验证数据 |
| bucket      | NSString * | 是    | 目录所属存储桶名称                           |
| sign        | NSString * | 是    | 签名                                       |
| compareFlag | NSString * | 否    | 录制的视频是否和图片作对比，yes 是对比，no 不做对比            |
| seq         | NSString * | 是    | 相应请求的 session 标识符，可用于结果查                   |
#### 返回结果说明
通过 CIITaskRsp 的对象返回结果信息

| 属性名称    | 类型             | 说明       |
| ------- | -------------- | -------- |
| retCode | int            | 任务描述代码   |
| descMsg | NSString    *  | 任务描述信息   |
| data    | NSDictionary * | 任务返回业务信息 |
#### 示例

```objective-c

  CILipIdentificationCompareTask *cm = [CILipIdentificationCompareTask new];
    cm.videoPath = path;
    cm.validate= validate;
    cm.compareFlag= flag;  
    cm.bucket = bucket;
    cm.sign = sign;
    CIClient *client= [[CIClient alloc] initWithAppId:appId];
    client.completionHandler = ^(CIITaskRsp *resp, NSDictionary *context){
        if (resp.retCode == 0) {
			//sucess
		}else{}
    };
    [client lipIdentificationCompare:cm];
```

### 真实身份信息核验

调用此接口自可实时为国内公民提供真实身份信息核验。根据用户的身份证号、姓名，与用户上传的图像进行人脸相似度对比。具体步骤如下：

1. 实例化 CIIDNumbeVideoComparTask  对象；
2. 调用 CIClient  的 IDCardCompareVideo 命令，传入 CIIDNumbeVideoComparTask  对象；
3. 通过 CITaskRsp 的对象返回结果信息。


#### CIIDNumbeVideoComparTask 参数说明

| 参数名称         | 类型         | 是否必填 | 说明                                       |
| ------------ | ---------- | ---- | ---------------------------------------- |
| videoPath    | NSString * | 是    | 视频的路径                                    |
| validate     | NSString * | 是    | 执行 CIGetLipIdentificationStringTask 返回的唇语验证数据 |
| idcardNumber | NSString * | 是    | 用户身份证号码                                  |
| idcardName   | NSString * | 否    | 用户身份证姓名                                  |
| faceList     | NSArray *  | 是    | 删除人脸 id 的列表）                               |
| bucket       | NSString * | 是    | 目录所属存储桶名称                           |
| sign         | NSString * | 是    | 签名                                       |
| seq        | NSString * | 是    | 相应请求的 session 标识符，可用于结果查        


#### 返回结果说明

通过 CITaskRsp 的对象返回结果信息

| 属性名称    | 类型             | 说明       |
| ------- | -------------- | -------- |
| retCode | int            | 任务描述代码   |
| descMsg | NSString    *  | 任务描述信息   |
| data    | NSDictionary * | 任务返回业务信息 |

#### 示例

```objective-c

    CIIDNumbeVideoComparTask *cm = [CIIDNumbeVideoComparTask new];
    cm.videoPath = path;
    cm.validate= validate;
    cm.compareFlag= flag;  
    cm.bucket = bucket;
    cm.sign = sign;

    CIClient *client= [[CIClient alloc] initWithAppId:appId];
    client.completionHandler = ^(CIITaskRsp *resp, NSDictionary *context){
        if (resp.retCode == 0) {
            //sucess;
        }else{}
    };
    [client IDCardCompareVideo:cm];
```

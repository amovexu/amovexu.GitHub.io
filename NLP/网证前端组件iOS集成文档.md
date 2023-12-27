
# 网证前端组件(CAS)集成文档
## 一，简介

网证前端组件-用于 iOS 和安卓 APP 组件，用于网证开通、下载、认证以及网证二维码展码等业务。

### 1.1 组件说明见下表

|组件名称 | 描述 | 是否必须 |
|------|--------------|------|
|CTID_Verification.framework | CTID认证组件| 是 |
| NLPBase.framework | NLP工具库组件| 是 |
| NLPCASBase.framework | 网证基础业务组件含网证开通及下载| 是 |
| NLPCASVerify.framework | 认证组件，实现网证认证，二维码识别等| 可根据业务选取 |
| NLPCASQrCode.framework | 展码组件，实现网证展码以及获取码图| 可根据业务选取 |

### 1.2 引用资源见下表

|资源名称 | 描述 | 是否必须 |
|------|--------------|------|
| NLPMedia.bundle | NLP工具组件资源文件| 是 |
| NLPCASMedia.bundle |  NLP网证相关资源文件| 是 |
| NLPCAS_iOS_License.lic | 网证前端组件授权文件| 是 |

### 1.3 引用第三方库表

| 名称 | 描述 | 是否必须 |
|------|--------------|------|
| 'AFNetworking' , '~> 4.0.1'| 网络请求工具库| 是 |
| MJExtension | json模型解析库 | 是 |
| MJRefresh | 通用刷新框架| 是 |
| WebViewJavascriptBridge | 原生web交互组件库| 如果有使用展码组件则必须引入|
| SGQRCode | 二维码扫描库| 需要二维码扫描则建议引入，也可用集成方原有组件 |


## 二，接入前准备

- 1，网证下载时需要使用活体控件采集活体照片，活体控件由集成方自行选用市场成熟的活体方案，并按SDK提供的接口协议进行唤起以及回传照片。

- 2，组件集成前，集成方需向我司申请组件授权文件，并放到工程目录中。


## 三，接入步骤
#### 3.1 加入网证组件
参考组件说明表根据业务需要将压缩包SDK文件夹内的framework文件加入工程中。

#### 3.2 添加资源依赖
将压缩包内SDK/Resource文件夹中的资源文件加入工程中，同时添加集成方申请的授权文件《 NLPCAS_iOS_License.lic》。

#### 3.3 添加第三方库依赖
利用cocopod引入依赖的第三方组件，参见1.3第三方组件表。
```
 platform :ios, ‘9.0’
 pod 'AFNetworking' , '~> 4.0.1'
 pod 'MJExtension'
 pod 'MJRefresh'
 pod 'WebViewJavascriptBridge', :git => 'https://github.com/SabinLee/WebViewJavascriptBridge.git',:branch => ‘master’
 pod 'SGQRCode' # 扫码建议使用这个库，也可以使用集成方已有库

```
#### 3.4 工程配置

- 4.1 开启APP相机使用权限

- 4.2 编译设置 other link flag 增加如下选项“-lstdc++”,“-lc++”。

- 4.3 关闭bitCode

- 4.4 如果贵司私有化服务端，并且不支持https访问，请开启工程中http访问权限


#### 3.5 集成活体控件
实现协议`<NLPRASBaseProtocol>`中唤起活体方法，回传检测结果。参见代码

`@interface AppDelegate ()<NLPRASBaseProtocol>`
 
```ObjC
   //  
 [NLPRASConfig shareInstance].delegate = self; //启动APP时或者调用网证功能前 设置委托
  //
// 实现唤起活体方法
-(void)startLivenessDetector:(NLPCASLivenessCallBackBlk) nlpLivenessCallBackBlk curVC:(UIViewController*)curVC{
    NLPFaceRecognitionVC *vc = [[NLPFaceRecognitionVC alloc]init];
    vc.hidesBottomBarWhenPushed = YES;
    [curVC.navigationController pushViewController:vc animated:YES];
    vc.nlpLivenessCallBackBlk = ^(NSString * _Nonnull photoStr, NLPBaseMsg * _Nonnull errMsg) {
        if(nlpLivenessCallBackBlk){
            nlpLivenessCallBackBlk(photoStr,errMsg);
        }
    };
}

```
活体采集后的照片采用base64压缩回传SDK,参见代码
```ObjC
 NSData *compressData = UIImageJPEGRepresentation(photoImg, 1);
 NSString *pstr = [compressData base64EncodedStringWithOptions:0];
```
#### 3.6 扫一扫功能
如果开启展码页面扫一扫功能，则需实现协议`<NLPRASBaseProtocol>`中唤起扫一扫方法，回传扫码结果。参见代码

```Objc
-(void)startScanQrCode:(NLPQrCodeScanCommpleteBlk) qrResultBlk curVC:(UIViewController*)curVC{
// 功能未开启，可以不实现
    XYZQQRCodeViewController *vc = [[XYZQQRCodeViewController alloc]init];//扫码控件由集成方自选,建议SGQRCode
       vc.tip = @"请扫描二维码";

    [curVC presentViewController:vc animated:YES completion:^{

    }];
    vc.qrResultBlk = qrResultBlk;
}
```

#### 3.7 桌面快捷方式
如果开启桌面快捷方式，需要集成方自行开发快捷网页页面，并实现协议`<NLPRASBaseProtocol>`中唤起创建快捷方式的方法。参见代码
```Objc
-(void)nlpSentQrCodeToDesktop{
    
    // 地址改为贵司部署快捷方式页面的地址
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        if([[UIDevice currentDevice] systemVersion].floatValue > 10){
            // 地址改为贵司部署快捷方式页面的地址
            //部署页面需将页面中的“nlpcasdesktopurl://”加上贵司APP前缀，否则所有的快捷方式都会打开同一个APP
            [[UIApplication sharedApplication] openURL:[NSURL URLWithString:kNLPFastKeySetupWebUrl] options:@{UIApplicationOpenURLOptionsSourceApplicationKey:@"efznlpcasdesktopurl://"} completionHandler:^(BOOL success) {
           }];
        }else{
            // 地址改为贵司部署快捷方式页面的地址
            [[UIApplication sharedApplication] openURL:[NSURL URLWithString:kNLPFastKeySetupWebUrl]];
        }
    });
    
}
```
关于快捷方式实现详细可以参考[iOS快捷方式创建](https://blog.csdn.net/SOHU_TECH/article/details/128337388?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-128337388-blog-128967558.235%5Ev39%5Epc_relevant_anti_t3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-128337388-blog-128967558.235%5Ev39%5Epc_relevant_anti_t3)

创建展码快捷方式后，需要实现从快捷进入APP后跳转展码页面
```Objc
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options NS_AVAILABLE_IOS(9_0){
 if([url.absoluteString isEqual:@"nlpcasdesktopurl://"]){ // 展码快捷方式进入APP
        [self gotoCode]; 
        return YES;
    }else{
        return NO;
    }
}
```

## 四，接口说明

---

<font color = red size = 5 face = "黑体">*4.1 展码组件*</font> 

#### 4.1.1 网证展码
网证展码接口，调用后进入SDK展码功能，由SDK的标准页面进行二维码展示以及相关业务，使集成方更专注自身业务开发。

- 接口

`+(void)showQrCode:(NLPRASQrCodeReq*)req complete:(NLPRASServiceResultBlock)completeBlk;`

- 参数说明

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| req | NLPRASQrCodeReq | 展码请求|是|

NLPRASQrCodeReq属性表

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| navigationController | UINavigationController | 当前导航栏控制器|是|
|userId| NSString | 用户ID，调用方传入，用于区分同一个APP同一部手机中不同的用户。传空则仅支持一个用户|建议传入|
|name| NSString | 用户有效身份证件姓名，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
|number| NSString | 用户有效身份证件号码，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
| phone | NSString | 手机号码|否|
| bizExtData | NSString | 业务透传数据|否|
| nlpAppId | NSString | 应用ID|应用ID，可以不传|
| idCardInfoEditMode | TRASIdCardInfoEditTModeDef | 两要素页面控制项,如果未指定模式则从`[NLPRASConfig shareInstance].idCardInfoEditMode`读取|否|

- 回调说明

NLPRASServiceResultBlock 结构

|参数名称 | 类型| 描述 
|:------|------|------|
| code | NSString | 操作结果代码，成功为0|
| msg | NSString | 操作结果提示|

- 调用代码参考

```Objc
    NLPRASQrCodeReq *req = [NLPRASQrCodeReq createReqWith:[NLPDemoAppConfigHelper shareInstance].config.useId navigationController:self.navigationController name:[NLPDemoAppConfigHelper shareInstance].config.name number:[NLPDemoAppConfigHelper shareInstance].config.number phone:[NLPDemoAppConfigHelper shareInstance].config.phone nlpAppId:nil];
    
    [NLPRASQrCodeService showQrCode:req complete:^(NLPRASServiceResult * _Nullable result) {
        if([result isValidMsg]==NO){
            [self showResult:result];
        }
        
    }];
```
#### 4.1.2 获取码图

获取码图接口，集成方调用该接口即可获取网证二维码码图，渲染在集成方自行开发的展码页面中。

-  接口

`+(void)getDigitalQrCode:(NLPRASQrCodeReq*)req logoImg:(nullable UIImage*)logoImg complete:(NLPRASServiceResultBlock)completeBlk;`

- 参数说明

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| req | NLPRASQrCodeReq | 生码请求|是|
| logoImg | UIImage | 码图中间logo，不传则使用SDK默认CTID logo|否|


NLPRASQrCodeReq属性见[4.1.2中 NLPRASQrCodeReq]


-  回调说明

NLPRASServiceResultBlock 数据结构

|参数名称 | 类型| 描述 
|:------|------|------|
| code | NSString | 操作结果代码，成功为0|
| msg | NSString | 操作结果提示|
| data | NLPRASQrCodeImgModel | 码图信息|

data NLPRASQrCodeImgModel 属性表

|参数名称 | 类型| 描述 
|:------|------|------|
| ctidQrCode | NSString | 二维码模位图| 
| ctidQrCodeWidth | CGFloat | 二维码模位图宽度|
| qrCodeBase64Str | NSString |  二维码图片base64 |

qrCodeBase64Str还原成图片参考代码：
```Objc
 NSData *imageData = [[NSData alloc] initWithBase64EncodedString: qrCodeBase64Str  options:0];
 UIImage *image = [UIImage imageWithData:imageData];
```
- 调用代码参考

```Objc
  NLPRASQrCodeReq *req = [NLPRASQrCodeReq createReqWith:[NLPDemoAppConfigHelper shareInstance].config.useId navigationController:self.navigationController name:[NLPDemoAppConfigHelper shareInstance].config.name number:[NLPDemoAppConfigHelper shareInstance].config.number phone:[NLPDemoAppConfigHelper shareInstance].config.phone nlpAppId:nil];
 
    UIImage *image = [UIImage nlp_bundleImageNamed:@"NLPCAS_ctidLog" bundleName:@"NLPCASMedia"];
    @weakify(self);
    [NLPRASQrCodeService getDigitalQrCode: req logoImg:image complete:^(NLPRASServiceResult * _Nullable result) {
        @strongify(self);
        NLPBaseMsg *resultMsg = [NLPBaseMsg createWithCode:result.code msg:result.msg];
        if(result.data){
            NLPRASQrCodeImgModel *model = [NLPRASQrCodeImgModel mj_objectWithKeyValues:result.data];
            resultMsg.data = model;
        }
        [self showResult:resultMsg];
    }];
```
#### 4.1.3 展码配置项
- 展码配置单例

`[NLPRASQrCodeConfig shareInstance]`

- 参数说明

参数名称 | 类型| 描述 | 默认值 |
|:------|------|------|:------|
| licName | NSString | 授权文件名|默认同基础业务库授权文件一致如 NLPCAS_iOS_License，可以不设置|
| availableTimes | NSInteger | 码图的可扫次数|1|
| qrCodeDuration | NSInteger |码图有效时间，以s为单位|60|
| showQRCodeCheckPeriod | NSInteger |出码校验时间间隔，以s为单位|180|
| healthSwitchStatus | TRASHealthStatusSwitchDef |健康状态|TRASHealthStatus_onLineConfig 从服务端获取配置|
| hasQrCodeScan | BOOL |是否在展码页显示扫一扫入口|YES |
| enableScreenShootQrCodePage | BOOL |是否允许展码截图，默认关闭|NO|

---
<font color = red size = 5 face = "黑体">*4.2 认证组件*</font> 

#### 4.2.1 认证授权接口

用于网证认证授权相关业务，当接口调用仅为认证是否本人操作，不需要取数则SDK走认证流程，如果有取数据的需求如姓名身份证等则走授权流程，由用户决定是否授权改数据项。

- 接口

`+(void)verifyAndAuthService:(NLPRASVerifyReq*)req complete:(NLPRASServiceResultBlock)completeBlk;`

- 参数说明

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| req | NLPRASVerifyReq | 认证或授权请求|是|

NLPRASVerifyReq属性表

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| navigationController | UINavigationController | 当前导航栏控制器|是|
|userId| NSString | 用户ID，调用方传入，用于区分同一个APP同一部手机中不同的用户。传空则仅支持一个用户|建议传入|
|name| NSString | 用户有效身份证件姓名，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
|number| NSString | 用户有效身份证件号码，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
| phone | NSString | 手机号码|否|
| bizExtData | NSString | 业务透传数据|否|
| nlpAppId | NSString | 应用ID|应用ID,非必传，为空则取APP授权文件应用ID|
| idCardInfoEditMode | TRASIdCardInfoEditTModeDef | 两要素页面控制项,如果未指定模式则从`[NLPRASConfig shareInstance].idCardInfoEditMode`读取|否|
| identifyDataList | NSArray | 授权数据项，仅认证可以传空，授权根据需求选择`<NLPCASBase/NLPRASDigitalIdentityDataDefine.h>`定义的数据项组合传递|
| authMode | TRASDigitalIDAuthMode |认证模式  TRASDigitalIDAuth_WithOutPhoto = 1,//  仅网证认证 TRASDigitalIDAuth_WithPhoto,// 网证+正常人像 认证 TRASDigitalIDAuth_WithPhotoHD // 网证+高清正常照片 认证|

-  回调说明

NLPRASServiceResultBlock 结构

|参数名称 | 类型| 描述 |
|:------|------|------|
| code | NSString | 操作结果代码，成功为0|
| msg | NSString | 操作结果提示|
| data | NSDictionary | 认证结果|

data字典值：

|参数名称 | 类型| 描述 |
|:------|------|------|
| verifyCode | NSString | 认证凭据号，如果是授权可通过凭据号到认证平台取数据|


- 调用代码参考

```ObjC
[NLPDemoAppConfigHelper shareInstance].config.nlpAppId =  [self.bizAppIdTF.text nlp_trimmingWhitespace];
    [NLPDemoAppConfigHelper saveDemoAppConfigToLocal];

    NLPRASVerifyReq *req = [NLPRASVerifyReq createReqWith:[NLPDemoAppConfigHelper shareInstance].config.useId navigationController:self.navigationController name:[NLPDemoAppConfigHelper shareInstance].config.name number:[NLPDemoAppConfigHelper shareInstance].config.number phone:[NLPDemoAppConfigHelper shareInstance].config.phone nlpAppId: [NLPDemoAppConfigHelper shareInstance].config.nlpAppId ];
    req.bizExtData = self.bizExtDataTF.text;
    req.authMode = [NLPDemoAppConfigHelper shareInstance].config.authMode;
    req.identifyDataList = [[NLPDemoAppConfigHelper shareInstance].config.identifyDataStr componentsSeparatedByString:@","]; // 仅认证可传空
    req.navigationController = self.navigationController;
    [NLPRASVerifyService verifyAndAuthService:req complete:^(NLPRASServiceResult * _Nullable result) {
        [self showResult:result];
    }];

```
#### 4.2.2 授权查询接口

用于查询应用授权信息。

-  接口

`+(void)authorizationInfoRequst:(NLPRASVerifyReq*)req complete:(NLPRASServiceResultBlock)completeBlk;`

-  参数说明

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| req | NLPRASVerifyReq | 认证或授权请求|是|

NLPRASVerifyReq属性表

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| navigationController | UINavigationController | 当前导航栏控制器|是|
|userId| NSString | 用户ID，调用方传入，用于区分同一个APP同一部手机中不同的用户。传空则仅支持一个用户|建议传入|
|name| NSString | 用户有效身份证件姓名，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
|number| NSString | 用户有效身份证件号码，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
| phone | NSString | 手机号码|否|
| bizExtData | NSString | 业务透传数据|否|
| nlpAppId | NSString | 应用ID|应用ID,非必传，为空则取APP授权文件应用ID|
| idCardInfoEditMode | TRASIdCardInfoEditTModeDef | 两要素页面控制项,如果未指定模式则从`[NLPRASConfig shareInstance].idCardInfoEditMode`读取|否|
| identifyDataList | NSArray | 授权数据项，传入根据需求选择的`<NLPCASBase/NLPRASDigitalIdentityDataDefine.h>`定义的数据项组合|
| authMode | TRASDigitalIDAuthMode |认证模式  TRASDigitalIDAuth_WithOutPhoto = 1,//  仅网证认证 TRASDigitalIDAuth_WithPhoto,// 网证+正常人像 认证 TRASDigitalIDAuth_WithPhotoHD // 网证+高清正常照片 认证|

-  回调说明

NLPRASServiceResultBlock 结构

|参数名称 | 类型| 描述 |
|:------|------|------|
| code | NSString | 操作结果代码，成功为0|
| msg | NSString | 操作结果提示|
| data | NSDictionary | 认证结果|

data字典值：

|参数名称 | 类型| 描述 |
|:------|------|------|
| verifyCode | NSString | 认证凭据号，如果是授权可通过凭据号到认证平台取数据|


-  调用代码参考

```ObjC
 NLPRASVerifyReq *req = [NLPRASVerifyReq createReqWith:[NLPDemoAppConfigHelper shareInstance].config.useId navigationController:self.navigationController name:[NLPDemoAppConfigHelper shareInstance].config.name number:[NLPDemoAppConfigHelper shareInstance].config.number phone:[NLPDemoAppConfigHelper shareInstance].config.phone nlpAppId: [NLPDemoAppConfigHelper shareInstance].config.nlpAppId ];
    req.bizExtData = self.bizExtDataTF.text;
    
    req.authMode = [NLPDemoAppConfigHelper shareInstance].config.authMode;
    req.identifyDataList = [[NLPDemoAppConfigHelper shareInstance].config.identifyDataStr componentsSeparatedByString:@","];
    req.navigationController = self.navigationController;
    [NLPRASVerifyService authorizationInfoRequst:req complete:^(NLPRASServiceResult * _Nullable result) {
        [self showResult:result];
    }];
```
#### 4.2.3 取消授权

用于取消应用授权信息。

- 接口

`+(void)authorizatioCancel:(NLPRASVerifyReq*)req complete:(NLPRASServiceResultBlock)completeBlk;`

-  参数说明

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| req | NLPRASVerifyReq | 认证或授权请求|是|

NLPRASVerifyReq属性表

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| navigationController | UINavigationController | 当前导航栏控制器|是|
|userId| NSString | 用户ID，调用方传入，用于区分同一个APP同一部手机中不同的用户。传空则仅支持一个用户|建议传入|
|name| NSString | 用户有效身份证件姓名，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
|number| NSString | 用户有效身份证件号码，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
| phone | NSString | 手机号码|否|
| bizExtData | NSString | 业务透传数据|否|
| nlpAppId | NSString | 应用ID|应用ID,非必传，为空则取APP授权文件应用ID|
| idCardInfoEditMode | TRASIdCardInfoEditTModeDef | 两要素页面控制项,如果未指定模式则从`[NLPRASConfig shareInstance].idCardInfoEditMode`读取|否|
| identifyDataList | NSArray | 授权数据项，传入根据需求选择的`<NLPCASBase/NLPRASDigitalIdentityDataDefine.h>`定义的数据项组合|
| authMode | TRASDigitalIDAuthMode |认证模式  TRASDigitalIDAuth_WithOutPhoto = 1,//  仅网证认证 TRASDigitalIDAuth_WithPhoto,// 网证+正常人像 认证 TRASDigitalIDAuth_WithPhotoHD // 网证+高清正常照片 认证|

-  回调说明

NLPRASServiceResultBlock 结构

|参数名称 | 类型| 描述 |
|:------|------|------|
| code | NSString | 操作结果代码，成功为0|
| msg | NSString | 操作结果提示|
| data | NSDictionary | 认证结果|

data字典值：

|参数名称 | 类型| 描述 |
|:------|------|------|
| verifyCode | NSString | 认证凭据号，如果是授权可通过凭据号到认证平台取数据|


-  调用代码参考

```ObjC
 NLPRASVerifyReq *req = [NLPRASVerifyReq createReqWith:[NLPDemoAppConfigHelper shareInstance].config.useId navigationController:self.navigationController name:[NLPDemoAppConfigHelper shareInstance].config.name number:[NLPDemoAppConfigHelper shareInstance].config.number phone:[NLPDemoAppConfigHelper shareInstance].config.phone nlpAppId: [NLPDemoAppConfigHelper shareInstance].config.nlpAppId ];
    req.bizExtData = self.bizExtDataTF.text;
    
    req.authMode = [NLPDemoAppConfigHelper shareInstance].config.authMode;
    
    req.identifyDataList = [[NLPDemoAppConfigHelper shareInstance].config.identifyDataStr componentsSeparatedByString:@","];
    req.navigationController = self.navigationController;
    [NLPRASVerifyService authorizatioCancel:req complete:^(NLPRASServiceResult * _Nullable result) {
        NSLog(@"");
        [self showResult:result];

    }];
```
#### 4.2.4 二维码验码

二维码验码，用于扫一扫登录，扫一扫绑定等场景。

-  接口

`+(void)qrCodeBusinessService:(NLPRASQrCodeVerifyReq*)req complete:(NLPRASServiceResultBlock)completeBlk;`

- 参数说明

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| req | NLPRASQrCodeVerifyReq | 认证或授权请求|是|

NLPRASQrCodeVerifyReq属性表

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| navigationController | UINavigationController | 当前导航栏控制器|是|
|userId| NSString | 用户ID，调用方传入，用于区分同一个APP同一部手机中不同的用户。传空则仅支持一个用户|建议传入|
| qrCodeStr | NSString | 二维码码值|是|
|name| NSString | 用户有效身份证件姓名，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
|number| NSString | 用户有效身份证件号码，目前仅支持身份证 |当`idCardInfoEditMode`处于非可编辑模式时必须传入，可编辑模式可以不传，由用户输入|
| phone | NSString | 手机号码|否|
| bizExtData | NSString | 业务透传数据|否|
| nlpAppId | NSString | 应用ID|应用ID,非必传，为空则取APP授权文件应用ID|
| idCardInfoEditMode | TRASIdCardInfoEditTModeDef | 两要素页面控制项,如果未指定模式则从`[NLPRASConfig shareInstance].idCardInfoEditMode`读取|否|

-  回调说明

NLPRASServiceResultBlock 结构

|参数名称 | 类型| 描述 |
|:------|------|------|
| code | NSString | 操作结果代码，成功为0|
| msg | NSString | 操作结果提示|
| data | NSDictionary | 认证结果|

data字典值：

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|------|
| verifyCode | NSString | 认证凭据号，大部分业务都无此值，预留字段|否 |



-  调用代码参考

```ObjC
 NLPRASQrCodeVerifyReq *req = [NLPRASQrCodeVerifyReq createReqWith:[NLPDemoAppConfigHelper shareInstance].config.useId navigationController:self.navigationController name:[NLPDemoAppConfigHelper shareInstance].config.name number:[NLPDemoAppConfigHelper shareInstance].config.number phone:[NLPDemoAppConfigHelper shareInstance].config.phone nlpAppId:nil];
    req.qrCodeStr = qrCode;
    req.navigationController = self.navigationController;
    [NLPRASVerifyService qrCodeBusinessService:req complete:^(NLPRASServiceResult * _Nullable result) {
        [NLPProgressHUD showToast:result.msg inView:self.view];
    }];
```

#### 4.2.5 认证配置项
- 展码配置单例

`[NLPRASVerifyConfig shareInstance]`

- 参数说明

参数名称 | 类型| 描述 | 默认值 |
|:------|------|------|:------|
| licName | NSString | 授权文件名|默认同基础业务库授权文件一致如 NLPCAS_iOS_License，可以不设置|
| authDataAgreementUrl | NSString | 个人信息授权协议地址| 可不设置，默认NLP标准文案 |
| authDataAgreementTip | NSString | 个人信息授权协议提示文本| 可不设置，默认NLP标准文案 |

---

<font color = red size = 5 face = "黑体">*网证基础业务组件*</font> 


#### 4.3.1  判断本地是否有网证

判断本地是否有网证，有返回YES。

- 接口

`+(BOOL)hasLocalDigitalIDFile:(nullable NSString*)userId;`

-  参数说明

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| userId | NSString | 用户ID，调用方传入，用于区分同一个APP同一部手机中不同的用户。传空则仅支持一个用户|建议传入|


-  返回说明

返回YES则代表本地有网证，NO则无网证。

- 调用代码参考

```ObjC

 if([NLPCASBaseSDKService hasLocalDigitalIDFile:[NLPDemoAppConfigHelper shareInstance].config.useId]){
        [NLPProgressHUD showToast:@"本地有网证"];
    }else{
        [NLPProgressHUD showToast:@"不存在网证信息"];
    };

```

#### 4.3.2  删除本地网证

删除本地网证

- 接口

`+(void)deleteLocalDigitalIDFile:(nullable NSString*)userId;`

-  参数说明

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| userId | NSString | 用户ID，调用方传入，用于区分同一个APP同一部手机中不同的用户。传空则仅支持一个用户|建议传入|


-  返回说明

无返回值。

- 调用代码参考

```ObjC

 [NLPCASBaseSDKService deleteLocalDigitalIDFile:[NLPDemoAppConfigHelper shareInstance].config.useId];

```

#### 4.3.3 替换已有网证的uerid

替换本地已经下过网证的用户id，替换后原用户id下的网证文件将为空改存到新的用户id下

- 接口

`+(void)replaceUserOrgUserId:(nullable NSString*)orgUserId withUserId:(NSString*)newUserId completeBlk:(nullable NLPRASServiceResultBlock)completeBlk;`

-  参数说明

|参数名称 | 类型| 描述 | 是否必传 |
|:------|------|------|:------|
| orgUserId | NSString | 原用户ID，调用方传入，用于区分同一个APP同一部手机中不同的用户|是|
| newUserId | NSString | 新用户ID，调用方传入，用于区分同一个APP同一部手机中不同的用户|是|

-  回调说明

NLPRASServiceResultBlock 结构

|参数名称 | 类型| 描述 |
|:------|------|------|
| code | NSString | 操作结果代码，成功为0|
| msg | NSString | 操作结果提示|

- 调用代码参考

```ObjC

[NLPCASBaseSDKService replaceUserOrgUserId:orgUserId withUserId:newUserId completeBlk:^(NLPRASServiceResult * _Nullable result) {
                @strongify(self);
                if([result isValidMsg]){
                    [NLPProgressHUD showToast:@"覆盖成功" inView:nil];
                }else{
                    [NLPProgressHUD showToast:result.msg inView:nil];
                }
            }];
```


- 控制台日志开关

`[NLPBaseLogManager sharedInstance].enableLog = NO;// 默认关闭`

- 基础业务组件配置

`[NLPRASConfig shareInstance]`

- 参数说明

参数名称 | 类型| 描述 | 默认值 |
|:------|------|------|:------|
| licName | NSString | 授权文件名|默认同基础业务库授权文件一致如 NLPCAS_iOS_License，可以不设置|
| delegate | NLPRASBaseProtocol | <font color = red size = 3 face = "黑体">委托，主要用于实现唤起活体以及快捷桌面等功能，调用任意网证SDK接口前必须先设置</font>|无|
| cameraPermissionCheckBySDK | BOOL |相机权限是否由SDK检查|YES|
| enablePersonalInfoRemid | BOOL |个人信息收集弹窗开关|NO|
| enableForeigner | BOOL |是否支持外籍人士|NO|
| idCardInfoEditMode | TRASIdCardInfoEditTModeDef |SDK姓名身份证输入页编辑模式: TRASIdCardInfo_EditEnable允许编辑，TRASIdCardInfo_EditForbiden禁止编辑，TRASIdCardInfo_EditHidden不显示SDK姓名身份证输入页面| TRASIdCardInfo_EditEnable|
| CTIDUserGuideWebUrl | NSString| 展码页面关于网证的说明页面,默认NLP地址，清空则从平台配置获取地址|默认NLP标准页面|


## 五，附录

### 5.1 错误码表

| 错误码 | 描述| 
|:------|------|
| 0 | 操作成功 |
| -1001 | 未知错误 |
| -1002 | SDK内部调用出错 |
| -1003 | 取消 |
| -1004 | 参数错误 |
| -1005 | 网络数据缺失 |
| -1100 | 授权文件错误 |
| -1999 | 功能暂不支持 |
| 20130 | bid已删除 |
| 20131 | bid已禁用 |
| 220001 | 网证已注销 |
| 220002 | 网证未开通 |
| 220003 | 网证已冻结 |
| 220004 | 网证已失效 |
| 220005 | 网证不存在 |
| 220012 | 网证已过期 |

### 5.2 更新记录

| 版本号 | 日期 | 描述|
| ------|------|------|
| v1.0.5 | 2020-08-21 |基于didp版本SDK改造并发布|
| v1.0.6 | 2020-09-01 |优化网证相关错误处理流程|
| v2.0.0 | 2020-09-11 |增加快捷出码功能|
| v2.1.0 | 2020-11-06 |优化已有流程提升用户体验|
| v3.0.0 | 2020-12-10 |剥离活体库，支持第三方活体|
| v3.1.0 | 2021-01-12 |增加出码校验功能|
| v3.2.0 | 2021-02-25 |更新身份认证库，优化部分流程|
| v3.3.0 | 2021-04-29 |增加出码校验方式|
| v3.4.0 | 2021-10-28 |1，增加工作证 2，增加个人信息功能|
| v3.5.0 | 2022-03-10 |增加web容器接口鉴权功能|
| v3.5.1 | 2022-03-20 |1，增加个人授权功能 2，支持双链路认证|
| v3.5.2 | 2022-03-25 |增加网证下载以及二维码下载模式|
| v3.5.3 | 2022-04-21 |增加本地码功能，优化出码校验|
| v3.6.1 | 2022-07-21 |1，增加 CAS 被扫唤起电子证照前端组件2，修复已知问题|
| v3.6.2 | 2022-08-09 |1，统一扫一扫验码接口 2，增加 CAS 主扫窗口申请码以及福建码唤起电子证照前端组件|
| v3.6.3 | 2022-09-09 |优化网证出错处理|
| v3.6.4 | 2022-09-14 |增加高清人像授权|
| v3.6.5 | 2022-10-20 |1，增加证件输入信息页面控制选项 2，优化下载流程|
| v3.6.6 | 2022-12-06 |1，增加境外人士数字身份入口 2，优化已有流程提升用户体验|
| v3.6.7 | 2023-02-21 |1，增加境外人士数字身份入口 2，优化已有流程提升用户体验|
| v3.6.8 | 2023-05-26 |统一网证错误处理|
| v3.6.9 | 2023-06-30 |增加码值识别接口|
| v3.7.0 | 2023-07-14 |1,开放获取网证接口（受限）|
| v3.8.0 | 2023-09-12 |1,增加V3双链路认证，2，优化已有流程提升用户体验|
| v4.0.0 | 2023-12-25 |CAS组件拆分，更换SDK信息保存方式，减少SDK体积|

### 5.3 个人授权数据项

| 数据项代号 | 描述| 
|:------|------|
| name | 姓名 |
| number | 证件号码 |
| validDateStart | 证件起始日期|
| validDateEnd | 证件截止日期|
| phone | 手机号码 |
| photo | 照片 |
| photoDataHd | 高清照片 |

### 5.4 旧版本升级指南
- 1,删除NLPCAS.framework引用，改为引入NLPCASBase.framework，NLPCASVerify.framework，NLPCASQrCode.framework并替换NLPBase.framework，CTID_Verification.framework以及资源文件NLPMedia.bundle和NLPCASMedia.bundle

- 2，NLPCASConfigDelegate改为NLPRASBaseProtocol


- 3,头文件引用`#import <NLPCAS/NLPCASManager.h> ` 改为
```Objc 
#import <NLPCASQrCode/NLPCASQrCode.h>  
#import <NLPCASVerify/ NLPCASVerify.h> 
#import <NLPCASBase/NLPCASBase.h>
```
或者根据项目用到的功能选择性引用.

- 4，CAS初始化部分
原初始化代码

```Objc
[NLPCASManager sharedInstance].enableDebug = YES  
[NLPCASManager sharedInstance].sdkConfig.delegate = self;  
```

改为
```Objc
[NLPRASConfig shareInstance].delegate = self;  

[NLPBaseLogManager sharedInstance].enableLog = YES;// 根据需要选择是否开启
```
- 5 网证认证

原代码
```Objc
@weakify(self);
[[NLPCASManager sharedInstance] setCASCTIDVerifyCommpleteBlk:^(NLPCASCTIDFileBaseInfo * _Nonnull fileInfo, NLPBaseMsg * _Nonnull errorMsg) {
       @strongify(self);
        [self showResult:fileInfo msg:errorMsg];
        }];
   [[NLPCASManager sharedInstance]CASReq:self type:type uID:CasUid idCard:self.number name:self.name phone:@""];
```
改为

```Objc
@weakify(self);
    NLPRASVerifyReq *req = [NLPRASVerifyReq createReqWith:@"userID" navigationController:self.navigationController name:nil number:nil phone:nil nlpAppId:nil];
    req.authMode = TRASDigitalIDAuth_WithPhoto;
    
    [NLPRASVerifyService verifyAndAuthService:req complete:^(NLPRASServiceResult * _Nullable result) {
        @strongify(self);
      //  [self showResult:result];
    }];

```

- 6 展码

原代码
```Objc
@weakify(self);
[[NLPCASManager sharedInstance] setCASQrCodeDownloadCommpleteBlk:^(NLPCASCTIDFileBaseInfo * _Nonnull fileInfo, NLPBaseMsg * _Nonnull errorMsg) {
 @strongify(self);
        if([errorMsg isValidMsg] == NO){
            [self showResult:fileInfo msg:errorMsg];
        }

        }];
[[NLPCASManager sharedInstance]CASReq:self type:NLPCASDownloadCTIDAndCreateQrCodeReq uID:CasUid idCard:self.number name:self.name phone:@""];
```
改为

```Objc
@weakify(self);
NLPRASQrCodeReq *req = [NLPRASQrCodeReq createReqWith:[NLPDemoAppConfigHelper shareInstance].config.useId navigationController:self.navigationController name:[NLPDemoAppConfigHelper shareInstance].config.name number:[NLPDemoAppConfigHelper shareInstance].config.number phone:[NLPDemoAppConfigHelper shareInstance].config.phone nlpAppId:nil];
     @strongify(self);
    [NLPRASQrCodeService showQrCode:req complete:^(NLPRASServiceResult * _Nullable result) {
        if([result isValidMsg]==NO){
            [self showResult:result];
        }
        
    }];
```
- 7 个人授权

原代码

```Objc
    NLPCTIDAuthorizationReq *req = [NLPCTIDAuthorizationReq createAuthorizationReqWithUID:@"user_001" authModeType:[NLPCTIDAuthorizationReq conversionAuthModeTypeByAuthMode: mode] idCardNumber:self.number name:self.name phone:@"" nlpAppId:[self.bizAppIdTF.text nlp_trimmingWhitespace] requestIdentifyDataList:identifyDataList];
    
    [[NLPCASManager sharedInstance]nlpCTIDAuthorization:self parameter:req complete:^(NLPBaseMsg * _Nonnull handleMsg) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.8 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            if([handleMsg isValidMsg]){
                [NLPOHAlertView showAlertWithTitle:@"授权结果" message:(handleMsg.msg.length ? handleMsg.msg : @"" ) dismissButton:@"取消"];
            }else{
                [NLPOHAlertView showAlertWithTitle:@"授权结果" message:(handleMsg.msg.length ? handleMsg.msg : @"未知错误" ) dismissButton:@"取消"];
            }
            
        });
    }];
```
改为

```Objc
 NLPRASVerifyReq *req = [NLPRASVerifyReq createReqWith:[NLPDemoAppConfigHelper shareInstance].config.useId navigationController:self.navigationController name:[NLPDemoAppConfigHelper shareInstance].config.name number:[NLPDemoAppConfigHelper shareInstance].config.number phone:[NLPDemoAppConfigHelper shareInstance].config.phone nlpAppId: [NLPDemoAppConfigHelper shareInstance].config.nlpAppId ];
    req.bizExtData = self.bizExtDataTF.text;
    req.authMode = [NLPDemoAppConfigHelper shareInstance].config.authMode;
    req.identifyDataList = [[NLPDemoAppConfigHelper shareInstance].config.identifyDataStr componentsSeparatedByString:@","];
    req.navigationController = self.navigationController;
    [NLPRASVerifyService verifyAndAuthService:req complete:^(NLPRASServiceResult * _Nullable result) {
        [self showResult:result];
    }];
```

























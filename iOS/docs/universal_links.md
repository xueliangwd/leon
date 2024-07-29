# 适配iOS13微信SDK更新
操作流程:
https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Access_Guide/iOS.html#jump3
Universal Links存放域名：
https://sign-mkt.jd.com/
https://sign-mkt.jd.com/apple-app-site-association
涉及修改组件：
- 1、JDPSSocialSDKManagerModule、JDPSAppManagerModule 和JDPSAppModule的delegate文件
- 2、工程文件infoplist 修改
Universal Links文件：
apple-app-site-association
{ "applinks":{ "apps":[
], "details":[ { "appID":"UB7E55V2KX.com.jdms.jdmobile", "paths":[ "*", "/" ] }, { "appID":"355A2TSX74.com.jingdong.jdms", "paths":[ "*", "/" ] } ] }}
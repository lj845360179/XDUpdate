## XDUpdate

<a href="https://996.icu"><img src="https://img.shields.io/badge/link-996.icu-red.svg"></a>

#### Android 自动更新 / 阿里云 OSS 一键上传更新

- 支持Android 8.1，不会因FileUriExposedException而无法安装下载的APK

- JSON、APK、Map文件的URL需要支持外链，即可以被直接访问，可考虑放在Git仓库、OSS或自己的服务器上

![Alt text](https://raw.githubusercontent.com/xingda920813/XDUpdate/master/Screenshot_notification.png)

![Alt text](https://raw.githubusercontent.com/xingda920813/XDUpdate/master/Screenshot_dialog.png)

![Alt text](https://raw.githubusercontent.com/xingda920813/XDUpdate/master/Screenshot_downloading.png)

## 引入

build.gradle中添加

	compile 'com.xdandroid:xdupdate:+'

## 自动更新
#### 1.准备描述更新信息的JSON文件

```
{
"versionCode":4,                          //新版本的versionCode,int型
"versionName":"1.12",                     //新版本的versionName,String型
"url":"http://contoso.com/app.apk",       //APK下载地址,String型
"note":"Bug修复",                         //更新内容,String型
"md5":"D23788B6A1F95C8B6F7E442D6CA7536C", //32位MD5值,String型
"size":17962350                           //大小(字节),int型
}
```

#### 2.构建XdUpdateAgent对象

```
XdUpdateAgent updateAgent = new XdUpdateAgent
.Builder()
.setDebugMode(false)	//是否显示调试信息(可选,默认:false)
.setUpdateBean(XdUpdateBean updateBean)	//设置通过其他途径得到的XdUpdateBean(2选1)
.setJsonUrl("http://contoso.com/update.json")	//JSON文件的URL(2选1)
.setShowDialogIfWifi(true)	//设置在WiFi下直接弹出AlertDialog而不使用Notification(可选,默认:false)
.setOnUpdateListener((needUpdate, updateBean) -> {
	//取得更新信息JSON后的回调(可选)，回调在主线程，可执行UI操作；
	//needUpdate为是否需要更新，updateBean为JSON对应的数据结构
	if (!needUpdate) Toast.makeText(context,"您的应用为最新版本",Toast.LENGTH_SHORT).show();
})
.setDownloadText("立即下载")	//可选,默认为左侧所示的文本
.setInstallText("立即安装(已下载)")
.setLaterText("以后再说")
.setHintText("版本更新")
.setDownloadingText("正在下载")
.setIconResId(R.mipmap.ic_launcher)	//设置在通知栏显示的通知图标资源ID(可选,默认为应用图标)
.build();
```

#### 3.检查更新

```
updateAgent.update(getActivity());
```

适用于 App 入口的自动检查更新。默认策略下：

1.若用户选择“以后再说”或者划掉了通知栏的更新提示，则**当天**对**该版本**不再提示更新，防止当天每次打开应用时都提示导致用户不胜其烦；

2.在任何网络环境下，均推送一条通知栏更新提示，点击通知后弹出对话框，防止直接弹框带来不好的用户体验。

可调用 XdUpdateAgent.Builder.setShowDialogIfWifi(true) 设置在 WiFi 下直接弹出更新提示框 (AlertDialog) 而不使用 Notification 的形式。

```
updateAgent.forceUpdate(getActivity());
```

适用于应用“设置”页面的手动检查更新。此方法无视上面的 2 条默认策略，如果有更新，总是对用户进行提示，且总是使用提示框 (AlertDialog) 的形式。

#### 4.若不想使用JSON文件，可传入由其他途径得到的XdUpdateBean

```
	XdUpdateAgent.Builder.setUpdateBean(XdUpdateBean updateBean);
```

可使用第三方推送服务的自定义消息/透传功能，接收到服务端推送过来的JSON(String)后，解析成一个XdUpdateBean，传入上述方法，即可使用推送带过来的JSON进行更新提示。

注意不是普通消息，这样会直接在通知栏上显示内容，不会进到自定义的代码处理块。

## 阿里云 OSS 一键上传更新

位于/XdUploadClient/下，XdUpdateClient.jar为程序主体，XdUpdateClient.cmd为Windows下使用的上传脚本，XdUpdateClient.sh为Linux下使用的上传脚本，config.properties为配置文件，其他文件为源码。

一般使用只需把上述 4 个文件放到一个目录（下面称为工作目录）下即可。

#### 1.将更新过的APK命名为 (包名).apk，放到工作目录下

#### 2.编辑config.properties配置文件

```
packageName = com.xdandroid.myproject		//包名
releaseNote = Bug修复		//更新内容
cdnDomain = http://my-project.oss-cn-shenzhen.aliyuncs.com/		//文件URL的主机名部分(斜线后置)
endpoint = http://oss-cn-shenzhen.aliyuncs.com		//OSS的Endpoint(无斜线)
accessKeyId = xXxxxXxXxxXxxxxX		//OSS的AccessKeyId
accessKeySecret = xXxxxxxXXxxXxxxXxxXxxXXXXxxXxx		//OSS的AccessKeySecret
bucketName = my-project		//OSS的BucketName
pathPrefix = download/		//文件URL的路径部分（不含文件名, 斜线后置）
```

上传后的APK安装包的URL为 : cdnDomain + pathPrefix + packageName + ".apk"

上传后的JSON文件的URL为 : cdnDomain + pathPrefix + packageName + ".json"

#### 3.将JSON文件的URL填入XdUpdateAgent.Builder的setJsonUrl(String jsonUrl)

#### 4.运行XdUpdateClient.cmd/XdUpdateClient.sh，等待上传完成

Linux系统下，XdUpdateClient.sh需具有"可执行"文件系统权限。

#### 5.指定使用的配置文件(可选)

运行XdUpdateClient.jar时可以带一个参数，传入配置文件的路径，即可使用该配置文件，而不是默认的config.properties。

```
	java -jar XdUploadClient.jar my-project.properties
```

若不带参数运行XdUpdateClient.jar，将使用与XdUpdateClient.jar同目录下的config.properties。


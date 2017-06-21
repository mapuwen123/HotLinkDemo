# HotLinkDemo

本示例项目针对[Bugly](https://bugly.qq.com/v2/index)和[Wally](https://github.com/Meituan-Dianping/walle)的应用

## Bugly简介

腾讯Bugly，为移动开发者提供专业的异常上报和运营统计，帮助开发者快速发现并解决异常，同时掌握产品运营动态，及时跟进用户反馈。

## Wally简介

Walle（瓦力）：Android Signature V2 Scheme签名下的新一代渠道包打包神器

瓦力通过在Apk中的APK Signature Block区块添加自定义的渠道信息来生成渠道包，从而提高了渠道包生成效率，可以作为单机工具来使用，也可以部署在HTTP服务器上来实时处理渠道包Apk的升级网络请求。

### Bugly使用

Bugly官网提供两种SDK分别是

#### 异常上报SDK

* 异常上报
* 统计(渠道, 用户, 安装数, 打开数...)等功能

#### 应用升级SDK

* 应用升级
* 热更新以及异常上报SDK所有功能
* 所以如果引用应用升级SDK就无需在重复引用异常上报SDK

### 项目导入
官方文档讲了两种导入方式,自动导入和手动导入;

#### 手动导入
通过在官网手动下载相关SDK文件的方式导入到项目;<br/>
(本示例采取自动导入的方式,所以对这种方式不做过多赘述)

#### 自动导入
通过配置gradle的方式导入到项目;

##### 配置build.gradle

在位于项目的根目录 `build.gradle` 文件中添加Gradle插件的依赖, 如下:

```groovy
buildscript {
    dependencies {
        // tinkersupport插件, 其中lastest.release指拉取最新版本，也可以指定明确版本号，例如1.0.4
        classpath "com.tencent.bugly:tinker-support:1.0.8"
    }
}
```

并在当前App更目录新建 `tinker-support.gradle` 文件:

![github-01.png](/images/01.png "github-01.png")

`tinker-support.gradle` 文件详细内容见项目;

在位于App的根目录 `build.gradle` 文件中添加, 如下:

```groovy
// 依赖插件脚本
apply from: 'tinker-support.gradle'

android {
    defaultConfig {
        ndk {
            //设置支持的SO库架构
            abiFilters 'armeabi' //, 'x86', 'armeabi-v7a', 'x86_64', 'arm64-v8a'
        }
    }
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}

dependencies {
    // 多dex配置
    compile "com.android.support:multidex:1.0.1" 
    //其中latest.release指代最新版本号，也可以指定明确的版本号，例如2.2.0
    compile 'com.tencent.bugly:nativecrashreport:latest.release' 
    compile 'com.tencent.bugly:crashreport_upgrade:1.3.1'
}
```

##### 参数配置

在AndroidMainfest.xml中进行以下配置：

* 权限配置
```groovy
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_LOGS" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

* Activity配置
```groovy
<activity
    android:name="com.tencent.bugly.beta.ui.BetaActivity"
    android:configChanges="keyboardHidden|orientation|screenSize|locale"
    android:theme="@android:style/Theme.Translucent" />
```

* 配置FileProvider
###### 注意：如果您想兼容Android N或者以上的设备，必须要在AndroidManifest.xml文件中配置FileProvider来访问共享路径的文件。
```groovy
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="${applicationId}.fileProvider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_paths"/>
</provider>
```

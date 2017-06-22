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

并在当前App根目录新建 `tinker-support.gradle` 文件:

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

##### 混淆配置

为了避免混淆SDK，在Proguard混淆文件中增加以下配置：
```groovy
-dontwarn com.tencent.bugly.**
-keep public class com.tencent.bugly.**{*;}
-keep class android.support.**{*;}
```

##### SDK初始化

需要在Application中进行初始化操作:
```groovy
Bugly.init(getApplicationContext(), "注册时申请的APPID", false);
```
该初始化方法为统一初始化操作,异常上报无需重复初始化;<br/>
参数解析：
* 参数1：上下文对象

* 参数2：注册时申请的APPID

* 参数3：是否开启debug模式，true表示打开debug模式，false表示关闭调试模式

##### 应用更新

SDK初始化完成便开启更新检测,只需在Bugly平台上传更新包即可;

##### 热更新

热更新需要生成基线包和补丁包:

* 基线包

首先应在 `tinker-support.gradle` 文件中配置:
```groovy
    tinkerId = "1.0.2-base"
```

注意:必须确保tinkerId的唯一性;

基线包无需配置如下参数可以注释:
```groovy
    // 编译补丁包时，必需指定基线版本的apk，默认值为空
    // 如果为空，则表示不是进行补丁包的编译
    //     @{link tinkerPatch.oldApk }
    baseApk = "${bakPath}/${baseApkDir}/app-release.apk"

    // 对应tinker插件applyMapping
    baseApkProguardMapping = "${bakPath}/${baseApkDir}/app-release-mapping.txt"

    // 对应tinker插件applyResourceMapping
    baseApkResourceMapping = "${bakPath}/${baseApkDir}/app-release-R.txt"
```

编译:<br/>
![github-02.png](/images/02.png "github-02.png")
![github-03.png](/images/03.png "github-03.png")

安装编译好的基线包并运行后SDK会自动上传当前基线版本;

* 补丁包

与基线包一样,补丁包也应对`tinker-support.gradle` 文件进行配置:
``` groovy
    /**
     * 此处填写每次构建生成的基准包目录
     */
    def baseApkDir = "app-0621-13-13-25"
    
    tinkerId = "1.0.2-patch"
    
    // 编译补丁包时，必需指定基线版本的apk，默认值为空
    // 如果为空，则表示不是进行补丁包的编译
    //     @{link tinkerPatch.oldApk }
    baseApk = "${bakPath}/${baseApkDir}/app-release.apk"

    // 对应tinker插件applyMapping
    baseApkProguardMapping = "${bakPath}/${baseApkDir}/app-release-mapping.txt"

    // 对应tinker插件applyResourceMapping
    baseApkResourceMapping = "${bakPath}/${baseApkDir}/app-release-R.txt"
```

注意:baseApkDir应与要修复的基线包路径对应;

编译:<br/>
![github-04.png](/images/04.png "github-04.png")
![github-05.png](/images/05.png "github-05.png")

### Walle使用

##### 配置build.gradle

在位于项目的根目录 `build.gradle` 文件中添加Gradle插件的依赖, 如下:

```groovy
buildscript {
    dependencies {
        classpath 'com.meituan.android.walle:plugin:1.1.3'
    }
}
```

并在当前App根目录新建 `multiple-channel.gradle` 文件:

![github-06.png](/images/06.png "github-06.png")
`multiple-channel.gradle` 文件详细内容见项目;

在位于App的根目录 build.gradle 文件中添加, 如下:

```groovy
// 多渠道使用walle示例（注：多渠道使用）
apply from: 'multiple-channel.gradle'
dependencies {
    compile 'com.meituan.android.walle:library:1.1.3'
}
```

创建channel配置:

![github-07.png](/images/07.png "github-07.png")
![github-08.png](/images/08.png "github-08.png") <br/>
channel用于配置渠道信息;

命令行打多渠道包：

gradlew clean assembleReleaseChannels <br/>
![github-09.png](/images/09.png "github-09.png")

生成渠道包:

![github-10.png](/images/10.png "github-10.png")

### 如何获取渠道信息？

如果你已经集成了Bugly的异常上报，你就可以通过以下方式来塞入渠道信息：

```groovy
// 获取渠道信息
String channel = WalleChannelReader.getChannel(getApplication());
Bugly.setAppChannel(getApplication(), channel);
// 这里实现SDK初始化，appId替换成你的在Bugly平台申请的appId
Bugly.init(getApplication(), "YOUR_APP_ID", true);
```

### 一个补丁修复所有渠道

生成热更新补丁按照上文步骤即可,无需分渠道生成补丁;

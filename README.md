### Android Studio多渠道打包

----为了方便统计各个平台的安装情况，配合运营推广，需要统计各个平台的安装情况。

**1. 修改Module:app build.gradle**

----在Module:app build.gradle的android下添加：

```
productFlavors {

    // UC
    uc {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "uc"]
    }
    
    // 百度
    baidu {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "baidu"]
    }
    
    // 应用宝
    yyb {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "yyb"]
    }
    
    // 酷安市场
    kuan {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "kuan"]
    }
    
    // 小米
    xiaomi {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
    }
    
    // 360
    qh360 {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "qh360"]
    }
        
    // 豌豆荚
    wandoujia {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
    }
}
```
或优化格式：

```
productFlavors {
    uc {}
    baidu {}
    yyb {}
    kuan {}
    xiaomi {}
    qh360 {}
    wandoujia {}
}

productFlavors.all { flavor ->
    flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
}
```

**2. 在AndroidManifest里面加上渠道标识**

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="xxx.com.xxx">
    
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <meta-data
            android:name="UMENG_CHANNEL"
            android:value="${UMENG_CHANNEL_VALUE}" />
            
        ...省略

    </application>
    
</manifest>
```

**3. 执行命令gradlew assembleRelease**

----在Android Studio的Terminal输入命令： **gradlew assembleRelease**，表示生成所有Release包，生成的包在app\build\outputs\apk目录下。

**assemble命令**

----assemble是Gradle中的编译打包命令。

用法如下：
- gradlew assembleWandoujiaRelease：打wandoujia渠道的release版本
- gradlew assembleWandoujiaDebug：打wandoujia渠道的debug版本
- gradlew assembleWandoujia：打wandoujia渠道版本（会生成wandoujia渠道的Release和Debug版本）
- gradlew assembleRelease：打全部Release版本

----打全部Release版本


当渠道包版本比较多时，可以自定义所打APK包名称，用以区分：

```
// 自定义输出配置，这里加上APK版本号1.0
applicationVariants.all { variant ->
    variant.outputs.each { output ->
        def outputFile = output.outputFile
        if (outputFile != null && outputFile.name.endsWith('.apk')) {
            // 输出apk名称为zhaoqy_v1.0_wandoujia.apk
            def fileName = "zhaoqy_v${defaultConfig.versionName}_${variant.productFlavors[0].name}.apk"
            output.outputFile = new File(outputFile.parent, fileName)
        }
    }
```

**4. build.gradle配置文件**
```
apply plugin: 'com.android.application'

def packageTime() {
    return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
}
 
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"
 
    defaultConfig {
        applicationId "com.xxx.xxx" //packagename
        multiDexEnabled true //dex突破65535限制
        minSdkVersion 15
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
 
     //签名
    signingConfigs {
    
        debugConfig {
            storeFile file("../zhaoqy_keystore")      //签名文件
            storePassword "123456"
            keyAlias "123456"
            keyPassword "123456"  //签名密码
        }
        
        release{
            storeFile file("../zhaoqy_keystore")      //签名文件
            storePassword "123456"
            keyAlias "123456"
            keyPassword "123456"  //签名密码
        }
    }
 
 
    // 注意：signingConfigs代码块一定要写在buildTypes前面，否则会报下面的错：
    // Could not find property 'debugConfig' on SigningConfig container.
    
    buildTypes {
    
        release {
            shrinkResources true
            zipAlignEnabled true
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
            
            // 自定义输出配置
            applicationVariants.all { variant ->
                variant.outputs.each { output ->
                    def outputFile = output.outputFile
                    if (outputFile != null && outputFile.name.endsWith('.apk')) {
                        // 输出apk名称为zhaoqy_v1.0_wandoujia.apk
                        def fileName = "zhaoqy_v${defaultConfig.versionName}_${variant.productFlavors[0].name}.apk"
                        output.outputFile = new File(outputFile.parent, fileName)
                        // 或 appName_v1.0_2018-07-03_wandoujia.apk
                        // def fileName = "appName_v${defaultConfig.versionName}_${packageTime()}_${variant.productFlavors[0].name}.apk"
                        // output.outputFile = new File(outputFile.parent, fileName)
                    }
                }
            }
        }
        
        debug {
            minifyEnabled false
            shrinkResources false
            signingConfig signingConfigs.debug
        }
    }
 
    // 多渠道打包配置
    productFlavors {
        uc {}
        baidu {}
        yyb {}
        kuan {}
        xiaomi {}
        qh360 {}
        wandoujia {}
    }
 
    productFlavors.all {
        flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
    }
}
 
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.4.0'
}

```




签名配置，见[Android Studio之gradle的buildTypes内部配置](https://github.com/zhaoqingyue/ZQYAndroidNotes/blob/master/Android%20Studio/Android_Studio%E4%B9%8Bgradle%E7%9A%84buildTypes%E9%85%8D%E7%BD%AE.md)。
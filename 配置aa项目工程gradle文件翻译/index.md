# 配置AA项目工程Gradle文件(翻译)


# 构建Gradle工程
## 注释处理的Gradle插件
> 首选方式

你可以通过使用Gradle插件[android-apt]来添加AndroidAnnotation处理。
这里有一个快速的工程示例：
<!--more -->
```xml
buildscript {
    repositories {
      mavenCentral()
    }
    dependencies {
        // replace with the current version of the Android plugin
        classpath 'com.android.tools.build:gradle:1.0.0'
        // replace with the current version of the android-apt plugin
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}

repositories {
    mavenCentral()
    mavenLocal()
}

apply plugin: 'com.android.application'
apply plugin: 'android-apt'
//将XXX填写成你要使用的AA版本号.eg. AAVersion = ‘3.1’
def AAVersion = 'XXX'

dependencies {
    apt "org.androidannotations:androidannotations:$AAVersion"
    compile "org.androidannotations:androidannotations-api:$AAVersion"
}

apt {
    arguments {
        androidManifestFile variant.outputs[0].processResources.manifestFile
        // if you have multiple outputs (when using splits), you may want to have other index than 0

        resourcePackageName 'com.myproject.package' //填写应用包名

        // If you're using Android NBS flavors you should use the following line instead of hard-coded packageName
        // resourcePackageName android.defaultConfig.packageName

        // You can set optional annotation processing options here, like these commented options:
        // logLevel 'INFO'
        // logFile '/var/log/aa.log'
    }
}

android {
    compileSdkVersion 19
    buildToolsVersion "20.0.0"

    defaultConfig {
        minSdkVersion 9
        targetSdkVersion 19
    }

    // This is only needed if you project structure doesn't fit the one found here
    // http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Project-Structure
    sourceSets {
        main {
            // manifest.srcFile 'src/main/AndroidManifest.xml'
            // java.srcDirs = ['src/main/java', 'build/generated/source/apt/${variant.dirName}']
            // resources.srcDirs = ['src/main/resources']
            // res.srcDirs = ['src/main/res']
            // assets.srcDirs = ['src/main/assets']
        }
    }
}
```
** 注意：**不要忘了修改`android.sourceSets.main`配置内容来适应你自己的项目需求。比如，ItelliJ的源文件在 `src/main`目录下，但在Eclipse的源文件在`src`目录下。你应该为(AA)生成的类指定一个写入的目录。

## AndroidAnnotations Gradle 插件
> 已被官方弃用

你可以用[Gradle AndroidAnnotations Plugin](https://github.com/ealden/gradle-androidannotations-plugin) 这个插件来构建AndroidAnnotation的Gradle工程项目。这个插件输出的文件是基于注解编译的；也可以通过配置IDEA工程项来使用AndroidAnnotations。
不幸的是，这个插件现在没人维护了，所以建议各位弃用。
关于这个插件的介绍 [plugin wiki](https://github.com/ealden/gradle-androidannotations-plugin/wiki)。


# Android中实用的Gradle配置

AndroidStudio已经成为Android开发的主流开发工具，如果你还在使用Eclipse，那么赶紧换吧。在AndroidStudio中，包的导入、插件的配置、打包编译等都是通过``/app/buidl.gradle``文件配置的。
在平时的开发中，经常碰到的问题如下：

* 友盟多渠道包打包
* 包的混淆
* 签名
* 集成微博、微信等三方平台登录
* 集成高德、百度等三方地图
* 环信集成
* 方法数超过65535数量限制

下面简单介绍使用AndroidStudio使用Gradle编译的好处。
<!--more-->

* 通过配置使用正式签名+混淆，无需像Eclpise一样导出签名包再安装。因为Eclipse默认使用``debug.keystore``
* 团队间相互合作时，避免由于不同pc上的``debug.keystore``在同一台设备上运行时卸载应用后安装，节省开发调试时间
* 通过配置，解决65535方法数量限制
* 配置多渠道包打包及导出
* 由于微信登录需要包为正式签名的包，使用``debug.keystore``签名的包无法调用微信Api，通过简单配置即可
* 由于最终的App需要打包签名+混淆，如果直到App发布时再做混淆，这酸爽你懂的~

问题也说了，好处也讲了，下面上代码：

**build.gradle**
	
	apply plugin: 'com.android.application'

	def releaseTime() {
	    return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
	}

	buildscript {
	    repositories {
	        mavenCentral()
	    }
	}

	tasks.withType(JavaCompile) { options.encoding = "UTF-8" }

	android {
	    compileSdkVersion 22
	    buildToolsVersion "22.0.1"

	    defaultConfig {
	        applicationId "com.package.sample" //修改为你的应用包包名
	        minSdkVersion 15
	        targetSdkVersion 22
	        versionCode 13
	        versionName "1.5.5"

	        // dex突破65535的限制
	        multiDexEnabled true
	        // 对应AndroidManifest.xml 中的<meta/> 配置内容
			manifestPlaceholders = [APP_CHANNEL: "UMENG_CHANNEL"]
	    }

	    //编译的java版本
	    compileOptions {
	        sourceCompatibility JavaVersion.VERSION_1_7
	        targetCompatibility JavaVersion.VERSION_1_7
	    }

	    signingConfigs {
	        debug {
	            // No debug config
	        }

	        release {
	            try {
	            	//appsign.jks为正式签名key文件，位于本文放在app/目录下
	                storeFile file("appsign.jks")
	                //配置在gradle.properties文件下中具体代码，看下文
	                storePassword RELEASE_STORE_PASSWORD
	                keyAlias RELEASE_KEY_ALIAS
	                keyPassword RELEASE_KEY_PASSWORD
	            }
	            catch (ex) {
	                throw new InvalidUserDataException("You should define RELEASE_STORE_PASSWORD and " +
	                        "RELEASE_KEY_ALIAS and" + "RELEASE_KEY_PASSWORD in gradle.properties."
	                )
	            }
	        }
	    }

	    buildTypes {
	        debug {
	            // 显示Log
	            buildConfigField "boolean", "LOG_DEBUG", "true"
	            versionNameSuffix "-debug"
	            minifyEnabled false
	            zipAlignEnabled false
	            shrinkResources false
	            signingConfig signingConfigs.debug
	        }

	        release {
	            // 不显示Log
	            buildConfigField "boolean", "LOG_DEBUG", "true"
	            versionNameSuffix "-released"
	            debuggable true
	            minifyEnabled true
	            zipAlignEnabled true
	            // 移除无用的resource文件
	            shrinkResources true
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	            signingConfig signingConfigs.release

	            applicationVariants.all { variant ->
	                variant.outputs.each { output ->
	                    def outputFile = output.outputFile
	                    if (outputFile != null && outputFile.name.endsWith('.apk')) {
	                        // 输出apk名称为App_v1.0_2016-01-12_tencent.apk
	                        def fileName = "App_v${defaultConfig.versionName}_${releaseTime()}_${variant.productFlavors[0].name}.apk"
	                        output.outputFile = new File(outputFile.parent, fileName)
	                    }
	                }
	            }
	        }

	    }

	    lintOptions {
	        // if true, stop the gradle build if errors are found
	        abortOnError false
	    }

	    packagingOptions {
	        exclude 'META-INF/ASL2.0'
	        exclude 'META-INF/MANIFEST.MF'
	        exclude 'META-INF/LICENSE.txt'
	        exclude 'META-INF/NOTICE.txt'
	    }

	    //添加.so库依赖
	    sourceSets {
	        main {
	            jniLibs.srcDirs = ['libs']
	        }
	    }

	    // 渠道打包
	    productFlavors {
	        google {}
	        youyoumob {}
	        tencent {}
	        wandoujia {}
	        _360 {}
	        baidu {}
	        meizu {}
	        xiaomi {}
	        huawei {}
	    }
	    productFlavors.all { flavor ->
	        flavor.manifestPlaceholders = [APP_CHANNEL: name]
	    }

	}

	dependencies {
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    compile 'com.android.support:design:22.2.0'
	    compile 'com.android.support:appcompat-v7:22.+'
	    compile 'com.android.support:recyclerview-v7:22.+'
	    //引用本地库
	    compile files('libs/gson-2.2.4.jar')
	    //引用远程库
	    compile 'com.squareup.picasso:picasso:2.5.2'
	    //引入解决方法数超过65535限制的库
	    compile 'com.android.support:multidex:1.0.0'
	}

配置文件中``gradle.properties``文件内容

	RELEASE_STORE_PASSWORD = ##签名密码
	RELEASE_KEY_ALIAS = ##秘钥别名
	RELEASE_KEY_PASSWORD= ##别名对应的密码

多渠道配置``AndroidManifest.xml``文件
	
	<application>

		.....

		<meta-data
            android:name="UMENG_CHANNEL"
            android:value="${APP_CHANNEL}" />

	</application>

配置编译的包种类，选择release版本的渠道包编译，从而解决微信等三方平台授权的调试问题。
如图：
	![buildVariants](http://7u2j5d.com1.z0.glb.clouddn.com/A534C768-CA5F-47E6-8485-77F006F6746D.png)
在配置完成后，编译出的文件就是打包签名+混淆后的签名包，从此再也不用担心微信等三方平台需要正式签名app的问题了。跟平时调试开发一样没有任何不适，唯一的地方就是签名和混淆会导致编译的时间稍微长了一点，但是简化了之前的繁琐步骤。

解决方法数65535数量限制，不仅仅需要配置build.gradle文件，另外需要在自定义Application中的``onCreate()``方法中加入如下代码：
	
	@Override
    public void onCreate() {
        //MultiDex 支持 65535 方法数量限制
        MultiDex.install(getApplicationContext()); // 注意，一定要在super.onCreate()方法之前调用，否则无效
        super.onCreate(); 
    }

--END--

至此，已经介绍完了Android中比较实用的Gradle配置，这个基本上可以满足一般的App开发需求，如果在开发过程中需要配置调试等问题。请阅读《Android Gradle Plugin 手册》

**参考资料**

1. 《Gradle Android Plugin 中文指导手册》：https://chaosleong.gitbooks.io/gradle-for-android/content/
2. 《Groovy 官方教程》-因为Gradle配置文件使用的是Groovy语法：http://www.groovy-lang.org/differences.html
3. 《Gradle插件用户指南(译)》： https://rinvay.github.io/android/2015/03/26/Gradle-Plugin-User-Guide(Translation)

-----------------------


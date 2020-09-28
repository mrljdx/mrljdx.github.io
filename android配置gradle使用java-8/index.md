# Android配置Gradle使用Java 8

目前Java 8的新特性``Lambda`` 结合 ``RxJava`` 在一起使用可以简化大量的代码，下面简单介绍在AndroidStudio中配置Gradle使得支持Java 8。

## 添加插件retrolambda

	apply plugin: 'me.tatarka.retrolambda'
	buildscript {
	    repositories {
	        mavenCentral()
	    }
	    dependencies {
	        classpath 'me.tatarka:gradle-retrolambda:3.2.4' //for java 8 lamda
	    }

	}

	dependencies {
		//从 maven central 获取最新版本插件
	    retrolambdaConfig 'net.orfjackal.retrolambda:retrolambda:+'
	    //本地版本，retrolambda.jar 位于libs目录
	    // retrolambdaConfig files('libs/retrolambda.jar')
	}

<!--more-->

## 配置``android{}``区块Java版本
	
	//java版本8
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

## 配置``proguard.pro``混淆文件
	
	## see https://github.com/evant/gradle-retrolambda for java 8
	-dontwarn java.lang.invoke.*

## 配置完成之后的``build.gradle``文件
	
	apply plugin: 'com.android.application'
	apply plugin: 'me.tatarka.retrolambda'

	buildscript {
	    repositories {
	        mavenCentral()
	    }
	    dependencies {
	        classpath 'me.tatarka:gradle-retrolambda:3.2.4' //for java 8 lamda
	    }

	}

	android {

		compileSdkVersion 23
	    buildToolsVersion "23.0.1"

	    defaultConfig {
	        applicationId "com.mrljdx.sample"
	        minSdkVersion 10
	        targetSdkVersion 23
	        versionCode 1
	        versionName "1.0"
	    }

		//java版本8
	    compileOptions {
	        sourceCompatibility JavaVersion.VERSION_1_8
	        targetCompatibility JavaVersion.VERSION_1_8
	    }

	    .....//省略其他配置

	}

	dependencies {
	    compile fileTree(include: ['*.jar'], dir: 'libs')
	    testCompile 'junit:junit:4.12'
	    //java 8 配置
	    retrolambdaConfig 'net.orfjackal.retrolambda:retrolambda:+'
	}

 END

 ---------------------



# 在Android中嵌入Lua脚本-基于AndroidStudio开发环境

翻看自己在2011-11-11（光棍节）当天写的博客[《Android中嵌入lua脚本，初步进阶》](http://mrljdx.blog.51cto.com/2251374/711618) 发现博客中的内容部分已经无效了，随着Android的发展AndroidStudio开发工具已经成为主流。下面重新开始，简单的说一下在Android中如何嵌入Lua脚本语言。
在开始介绍之前，说一下为什么今天又想到怎么在Android嵌入Lua脚本语言，因为这有可能成为一种Android应用技术解决方案。在游戏方面早在很早以前就有“愤怒的小鸟”把Lua作为关卡脚本语言运用其中。

## 准备工作

1. 下载[lua5.3.2源码](http://www.lua.org/ftp/)和[luajava 1.1源码](http://files.luaforge.net/releases/luajava/luajava/LuaJava1.1)。
2. 使用Android NDK ，JNI编译下载的``Lua`` 和``LuaJava``源码生成.so文件。
3. 按照以下步骤配置AndroidLua库文件：
	* luajava下的org文件夹拷贝到工程``src/main/java``目录下
	* 将``jniLibs/armeabi``下的``libluajava.so``重命名为``libluajava-1.1.so``

<!--more-->

在做完准备工作后，就可以在Android中简单的使用Lua脚本语言了。

由于本文并不是介绍Lua语法细节的，所以只是把部分代码拷贝运行查看即可。

## 在Android中运行Lua简单示例

	package com.mrljdx.androidlua;

	import android.os.Bundle;
	import android.support.v7.app.AppCompatActivity;
	import android.util.Log;
	import android.widget.TextView;

	import org.keplerproject.luajava.LuaState;
	import org.keplerproject.luajava.LuaStateFactory;

	/**
	 * Created by Mrljdx on 16/1/4.
	 */
	public class AndroidLuaActivity extends AppCompatActivity {
	    public static final String TAG = "AndroidLuaActivity";

	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_android_lua);

	        TextView tvLua = (TextView) findViewById(R.id.tvLua);

	        LuaState luaState = LuaStateFactory.newLuaState();
	        luaState.openLibs();//打开标准库

	        luaState.LdoString("text = 'Hello Android, I am Lua.'");

	        luaState.getGlobal("text");

	        String text = luaState.toString(-1);
	        Log.e(TAG, "text is ： " + text);
	        tvLua.setText(text);

	        luaState.LdoFile("hello.lua");

	    }

	}

## 本文源码下载

AndroidLua 源码地址：[https://github.com/mrljdx/AndroidLua](https://github.com/mrljdx/AndroidLua) 喜欢的star一下~

## END

基于AndroidStudio编译在Android中使用Lua的so参考和查阅了很多资料，之前也做过使用Lua脚本写的游戏。那时候Nokia还健在~以前的Lua语法都忘得差不多，不过以后在Android中使用Lua的情景应该会很多！
希望有想把Lua运用在Android应用开发中的朋友一起探讨学习，废话就不多说了，多多交流开卷有益~

** 参考资料**

* [AndroLua ：Lua and LuaJava ported to Android ， Eclipse上编译的](https://github.com/mkottman/AndroLua)
* [在Android Studio中利用gradle来自动编译jni](http://coolerfall.com/tools/use-ndk-in-android-studio/)

-------------------


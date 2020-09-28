# 单例模式(Singleton Pattern)


## 什么是单例模式
首先来看一段单例模式的通用代码：
```java
public class Singleton{
    //自行实例化
	private static final Singleton instance = new Singleton();
	/**
	*1,构造函数为私有，不能通过new获得对象实例，限制实例产生
	*2,自行实例化
	*/
	private Singleton(){
	};
	//只能通过调用Singleton来获取其实例
	public static Singleton getInstance(){
		return instance;
	}
	//单例模式中的其他方法尽量是static，
	public static void otherFuc(){
	}
} 
```
<!--more-->
通过代码中的注释可以发现，单例模式中的构造方法为私有的，是不能被外部通过new来实例化的。Singleton类通过private构造方法能确保在一个应用中只产生一个Singleton的实例，并且是自行实例化的。
## 单例模式使用场景
在一个应用中，要求一个类只产生一个对象，如果出现多个类的对象就会导致应用出现问题的情况。（比如，Android中Application类就只能产生一个，通过这个单例类来实现缓存数据的存储等）
**具体使用场景如下：**

+  需要定义大量的静态常量和方法（如工具类）的环境，可以采用单例模式。（也可以直接定义这些常量和方法为static直接.使用）
+ 创建一个对象需要消耗过多的资源（比如I/O，访问数据库等），可以采用单例模式。（Android中DBHelper可以参考参考）
+ 在整个项目中需要一个共享访问点和共享数据，比如Android中通过Share Perfrence来共享数据等，Application类就是单例类。
+ 要求生成唯一序列号的环境。

## 单例模式优缺点
### 优点

+  由于单例模式在内存中只有一个实例，减少了内存开销，特别是那种一个对象需要不断的创建、销毁而且类的性能又无法优化的情况下，单例模式你值得拥有。
+ 由于单例模式只生成一个实例，可以减少系统的性能开销。当一个对象的产生需要较多的资源时，如读取配置、产生其他依赖对象时，则可以通过在启动应用的时候，直接产生一个单例对象，然后用永久驻留内存的方式来解决。（Android开发中也经常会遇到，比如百度地图、推送服务的初始化，AsyncImagerLoader初始化等）<**注意：**在使用单例模式的时候要注意JVM的GC机制，单例在内存中被干掉了，再使用就会报异常>
+ 单例模式可以避免对系统资源的多重占用，比如一个写文件的操作，由于只有一个实例，所以可以避免对同一个文件资源同时写操作。
+ 单例模式可以在系统中设置一个全局的访问点，优化和共享资源访问，例如可以设计一个单例类来负责数据表的映射处理。

### 缺点

+ 单例模式一般没有接口，扩展很困难，所以如果想扩展单例类，唯一的办法就是修改类的代码。（为什么单例类没有接口？因为单例类实在内部自行实例化的，并且提供单一实例，而接口或抽象类是不能被实例化的。）
+ 在并行开发中，如果单例类没开发完全，则对于测试来说，是不利的。想一想单例没有抽象也没有接口，测试人员也无法使用mock的方式来虚拟一个对象。
+ 单例模式（在类中干很多事）和单一职责（只做一件事）是相悖的，当然仍和设计模式都不是绝对要遵从的，视具体的情况而定，白猫黑猫抓到老鼠才是好猫。

## 注意事项
 在高并发的情况下，要注意单例模式的线程同步问题，避免产生多个实例。单例模式有很多中不同的实现方法。开头的那段代码是不会产生多个实例滴。但是下面这段代码就有可能产生：
```java
public class Singleton{
	//未实例化
	public static Singleton instance = null;
	//防止从外部被实例化
	private Singleton(){
	}
	//获取实例
	public static Singleton getInstance(){
		if(instance==null){
			instance = new Singleton();
		}
		return instance;
	}
}
```
该单例模式在低并发的情况下不容易出现问题，如果系统并发量大可能在内存中出现多个单例类的实例和预期（内存中只存在一个单例）不一样。比如线程A，在执行到``if(instance==null)``时，在单例将要被初始化，但是还没被初始化的时候（此时``instance==null``），线程B，也执行到``if(instance==null)``判断为``true``，那么A拥有了一个单例实例，而B也会创建一个单例实例。这个时候内存中就存在两个实例，就和预期只存在一个不符。
那么如何解决这种“异常”情况呢？

+  简单粗暴的方式，就是像文章开篇那样 ``private static Singleton instance = new Singleton()``这种（饿汉式单例）防止出现多个单例对象。
+ 还有一种就是在getInstance时，使用``synchronzined``来实现线程锁。比如下面的两种方式（饱汉式单例）

```java
	//方式1
	public static synchronzined Singleton getInstance(){
		if(instance==null){
			instance = new Singleton();
		}
		return instance;
	}
	//-----------------我是分割线----------------
	//方式2
	private static Object lock=new Object();
	public static Singleton getInstance(){
		if(instance==null){
			synchronized (lock)
            {
	            if(instance==null){
					instance = new Singleton();
				}
            }
		}
		return instance;
	}
```
其次需要考虑对象复制的情况。在Java中，对象默认是不能被复制的，如果实现了Cloneable接口，并实现了clone方法则可以直接通过对象复制的方式创建一个新对象。
一般情况下，不用考虑单例的复制情况。很少出现一个单例类还主动要求被复制的情况。解决这个问题的方法，就是别让单例类实现Cloneable接口就好了。

## 有上限的多例模式
有上线的多例模式是单例模式的一种扩展，比如只要求一个类在内存中只存在三个对象实例怎么做？代码如下：
```java
public class MultiSingletons{
	//定义上限为3个实例
	private static int maxNum = 3;
	private static ArrayList<MultiSingletons> mInstances = new Array<MultiSingletons>();
    //产生3个实例对象
	static{
		for(int=0;i<maxNum;i++){
			mInstaces.add(new MultiSingletons());
		}
	}
	
	private MultiSingletons(){
	}
	
	public static ArrayList<MultiSingleton> getMInstances(){
		return mInstances;
	}
}
```
## 小结
单例模式在平时的开发中应用的比较广泛，比如Android开发中经常使用单例模式。
不过在使用单例模式时，一定要注意JVM的垃圾回收机制，就是单例对象如果在内存中长久不使用，那么JVM就会认为该对象为垃圾，在CPU空闲的情况下，该对象会被清除掉，下次调用该对象则需要重新产生一个对象。
如果在单例类中，保存了程序的一些状态值，那么这些状态值会在变成初始值。这个时候应用拿到的值就不是之前保存在单例中的值，就会出现故障。对于需要记录状态值的情况，自行管理单例的生命周期，或者通过观察者模式，记录单例中值的变化，然后保存值到文件中，那么在下次发现变化时，可以将正确的值恢复，从而避免数据丢失。

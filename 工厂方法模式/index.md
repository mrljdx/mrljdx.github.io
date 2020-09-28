# 工厂方法模式


## 什么是工厂方法模式
首先来看一下Android手机和Iphone手机使用工厂方法是怎么生产的，你就懂了。
![工厂模式类图](http://img.blog.csdn.net/20150404105016883)

<!--more-->

如图，首先定义了工厂的抽象类``Factory`` 用于生产手机，其次，定义了一个手机抽象类``Mobile`` 来定义手机的共有方法（打电话），然后定义了一个抽象方法，手机的特点。
在定义了工厂和产品（手机）后，就可以根据需求来指定手机生产的厂家和手机的具体要求了。我们定义一个``富士康工厂``（FusicoFactory）继承抽象工厂（Factory），然后定义了两种手机Android和Iphone继承抽象手机类（Mobile）。这样就，就可以开始根据自己的需要来生产手机了。
```java
/**
 * 工厂方法使用
 * @author luson
 *
 */
public class Main {
	public static void main(String[] args){
		//定义工厂
		Factory factory = new FusicoFactory();
		//定义产品
		Mobile android = factory.createMobile(AndroidMobile.class); //生产Android手机
		Mobile iphone = factory.createMobile(IphoneMobile.class); //生产Iphone手机
		System.out.println("生产手机特性如下：");
		android.property(); //android手机特性
		iphone.property();//iphone手机特性
	}
}
```
> 输出结果：
> 生产手机特性如下：
Android系统
IOS系统

## 工厂模式的好处

+ 良好的封装性，代码结构清晰。一个对象的创建是有条件约束的，如一个调用者需要一个具体的产品对象，只要知道这个产品的类名即可，不用知道创建对象的过程，降低模块耦合。（这点遵循迪米特法则）
+ 工厂模式的扩展性非常好，在增加产品类的情况下，只要适当的修改具体的工厂类或者扩展一个具体的工厂类，就可以适应需求的变化。比如突然想生产Nokia手机，那么扩展一个Nokia的手机产品，交给富士康生产即可。富士康工厂生产手机的方法不需要做任何改变。
+ 产品类的实现如何变化都不影响调用者正常调用，比如``Mobile android = factory.createMobile(XiaoMiMob.class);`` 他不需要去知道生产什么样的手机，值需要告诉工厂，要生产手机，至于最后生产的手机是怎样的，则由传入的“手机模型”决定。这就是工厂方法的灵活性。
+ 工厂方法模式是典型的解耦框架。高层模块只需要知道产品的抽象类，其他的实现类都不需要关心，符合迪米特法则。也符合依赖倒置原则，只依赖产品的抽象，而不需要去依赖实现。同时这也符合里氏替换原则，在父类出现的地方，也可以用子类来替换，不信你试试。把 ``Mobile android`` 替换为 ``AndroidMobile android`` 也是没有问题的。

## 使用场景
1. 当需要灵活的、可扩展的框架时，可以考虑采用工厂方法模式。任何具体的事物都可以抽象为对象。
2. 可以将工厂方法模式用在异构项目中。比如一些外包项目中，和已有的系统对接等。
3. 在测试驱动开发框架下，在测试类A的时候，可以使用工厂方法模式，把和类A关联的类B虚拟出来，避免类A与类B耦合。比如前面提到的依赖倒置设计原则。使用JMock和EazyMock可以很方便的进行单元测试。

## 工厂方法模式的扩展
工厂方法模式可以和很多其他的模式结合使用，这样能设计出更好的架构。
#### 简单工厂模式
有时候为了方便起见，可以将工厂中的方法设为static静态方法，这样在调用工厂的方法时，可以直接.使用，而不需要去创建一个工厂实例，然后再调用。这和平时在开发Android应用是，定义一些工具类一样。这样做是很方便，缺点就是对这个类扩展起来比较难，只能在内部修改。虽然不符合开闭原则，但是用起来顺手就好。毕竟简单才是真嘛！（PS：我写的工具类，从来也没想过让其他类来继承这个工具类并扩展，都是直接修改，主要是以添加/重载方法为主。）

### 升级为多工厂类
本文中，在介绍生产手机时，只介绍了一家工厂FusicoFactory，但是手机的生产需求可能不同，这个时候可以多定义几个工厂来生产不同需求的手机。然后定义每个工厂类的不同职责，比如有的工厂生产智能手机，那么定义一个生产智能手机的方法，而有的工厂只生产功能机，那么可以定义一个功能机方法，分别来实现手机产品的生产。

### 使用工厂方法来生产单例类
比如前面所说的单例模式，在工厂方法模式中有能怎么实现呢？代码如下：
首先定义一个单例类Singleton：
```java
public class Singleton {
	
	private Singleton(){
		
	}
	
	
	public void showSingleton(){
		System.out.println("I am a Singleton");
	}
}
```
其次，定义一个生产单例的工厂SingletonFactory
```java
public class SingletonFactory{
	private static Singleton instance;
	static try{
		Class cl = Class.forName(Singleton.class.getName);
		//获得无参构造
		Constructor constructor = cl.getDeClaredConstructor();
		//设置无参构造函数是可访问的
		constructor.setAccessible(true);
		//产生一个实例对象
		instance = constructor.newInstance();
		}catch(Exception e){
		    //处理异常
		｝
	}
	public static Singleton getInstance(){
		return instance;
	}
}
```
简单回顾一下单例模式的核心是，“在内存中只存在单例类的一个对象。”在这里通过Java反射机制来获取获得类，然后获取类的构造器，设置访问权限，生成一个对象，然后提供访问，保证了内存中对象的唯一。
> 注：其他类也可以通过反射的方式来建立一个单例对象，这个通过工厂方法创建单例对象的框架，可以扩展，用于项目中产生一个单例构造器，所有需要产生单例的类都需要遵循（private 构造函数）的规则，然后通过扩展这个框架，只需要输入一个类型就可以获得这个类的单例对象。

### 延迟初始化
延迟初始化，就是在一个对象被利用完之后，不立即释放，工厂类保留其初始状态，等待在此被调用。以前面生产手机的工厂方法为例，示例代码如下：
定义手机抽象类：
```java
public abstract class Mobile{
	//手机公共的方法，打电话
	public void call(String phoneNum){
	}
	//手机型号
	public abstract void property();
}
```
定义Android手机类：
```java
public class Android extends Mobile{
	//实现手机型号
	@Override
	public void property(){
		//Android系统
	}
}
```
定义Iphone手机类
```java
public class Iphone extends Mobile{
	//实现手机型号
	@Override
	public void property(){
		//IOS系统
	}
}
```
定义延迟加载的工厂类
```java
/**
 * 延迟加载的手机工厂类
 * @author luson
 *
 */
public class MobileFactory {
    //定义缓存容器
	private static final Map<String,Mobile> map = new HashMap();
	public static synchronized Mobile createMobile(String type) throws Exception{
		Mobile phone = null;
		//如果缓存中有，则直接拿缓存数据
		if(map.containsKey(type)){
			phone = map.get(type);
		}else{
			if(type.equals("Android")){
				phone = new Android();
			}else if(type.equals("Iphone")){
				phone = new Iphone();
			}
			//将数据放入缓存
			map.put(type, phone);
		}
		return phone;
	}
}
```
这里代码比较简单，定义了一个Map容器，来容纳所有产生的对象，如果需要的对象存在，则直接取出，如果不存在，则新建一个再保存到Map中，方便下次调用。
延迟加载框架在平时的开发中还是比较有用的，比如图片类的加载，在Android开发中，经常要请求网络图片，图片的本地缓存，内存缓存都可以通过扩展这套框架来实现。
而且，延迟加载框架还可以通过判断Map的大小，来限制一些类的实例化数量，这样的处理很有意义，比如Android中Bitmap的使用，一定要谨慎，处理不好容易内存溢出。这就需要限制。

## 小结
工厂方法模式在项目中使用的还是比较频繁的，随便找一套开源的代码就可以看到工厂方法模式的身影。软件设计的乐趣所在，就是懂得灵活运用各种设计模式，来让自己的软件架构更加灵活稳定，平时在开发时，可以多思考工厂方法模式和其他设计模式的组合使用（比如模版方法模式、单例模式、原型模式等）
其实设计模式是一种授人以渔，真正想运用好，还是需要自己动手做项目，通过项目实践来体验设计模式之美。个人建议多看大神代码，有能力的话参与一些开源项目。

# Android性能优化笔记


## 性能优化概要：

+ 不要在主线程中做耗时操作，比如网络请求，文件读写等。
+ 使用ViewStub来推迟初始化，在运行时展开资源。
+ 使用RelativeLayout代替嵌套LinearLayout，尽可能扁平化布局。减少创建对象的数量，也会让事件的处理速度更快。（了解View事件的传递机制）
+ 主线程中尽可能少做事，把耗时操作交给子线程。
+  Sqlite数据库，通过事务来操作。（当数据库在持久存储中使用事务耗时更短）
+ 对字符串操作用StringBuffer性能比较好
+ 避免建立对象，避免使用枚举，避免使用浮点数
+ 一般类中最好不用setter和getter
+ 将成员变量缓存到本地，因为访问成员变量比访问本地变量要慢的多。
+ 使用final关键字定义类中的常量或方法，在编译时能帮助编译器优化代码。
+ 使用实体类比接口好，因为调用一个接口的引用比调用一个实体类要多花一倍的时间。（偶尔不利于设计，看情况）
+ 在处理字符串的时候，使用String.indexOf()，String.lastIndexOf()等特殊实现的方法。这些方法都是C/C++实现的，比起Java循环要快10~100倍。
<!--more-->
## 使用场景举例

> （一） 在一个Activity中，会根据传入的结果，或者网络请求的结果不同，来显示两种不同的界面，应该如何实现来提升性能？

一般情况下，我们会在这个Activity的布局xml中设置两个视图，分别显示或隐藏来展示效果，但是在onCreate方法中setContentView()展开布局时，有些View是没有必要初始化的，这时在实例化布局视图对象时，会多实例化不必要的视图对象。
这时，如果使用ViewStub来定义视图，实现延迟加载，可以提升应用的性能。如果视图需要展开，可以使用ViewStub 的inflate()方法在代码中展开视图。当然还有其他展开视图的方式比如setVisibility()。
```xml
<ViewStub
   android:id="@+id/myid"
   android:layout="@layout/mylayout"
   />
```
调用：
```java
View view = findViewById(R.id.myid);
view.setVisibility(View.VISIBLE);//ViewStub被展开的布局mylayout所替换
view = findViewById(R.id.myid);//获取展开后的布局 
```
如果代码中通用性很重要，无论在布局中有无使用ViewStub，都能正常工作，那么以上这种方法是首选。不过调用两次findViewById()会影响性能。为了部分解决这个问题，既然调用setVisibility(View.VISIBLE)会把ViewStub从父容器中删除，可以在第二次调用findViewById()之前检查视图是否有父视图。对仍然使用ViewStub来说，第二次调用findeViewById()还是没有优化，它只保证没用ViewStub时，只调用一次findeViewById()。修改后代码如下：
```java
View view = findViewById(R.id.myid);
view.setVisibility(View.VISIBLE);
if(view.getParent()==null){
	//用了ViewStub，需要展开后的新视图替换
	view = findViewById(R.id.myid);
}else{
	//没用，则不处理任何事情
}
```
不过个人建议，还是别怕麻烦，该怎么显示调用就怎么用:
```xml
<ViewStub
	android:id="@+id/mystubid"
	android:inflateId="@+id/myid"
	android:layout="@layout/mylayout"
/>
```
代码中展开布局：
```java
ViewStub stub = (ViewStub)findViewById(R.id.mystubid);
View inflateVIew = stub.inflate(); //inflateView定义在mylayout.xml的布局中
```
另外一种在代码中使用setVisibility()展开布局，这种方式由于没有指明ViewStub类，所以更通用：
```java
View view = findViewById(R.id.mystubid);
view.setVisibility(View.VISIBLE);//展开的布局取代ViewStub
view = findViewById(R.id.myid);//得到展开的布局视图
```

>（二）使用``<merge/>``标签来合并布局

一般布局的顶层元素是一个FrameLayout。因为Activity内容视图的父视图是一个FrameLayout。如果自定义视图的父视图为FrameLayout，则可以使用``<merge/>`` 标签来合并两个视图，可以少创建一个FrameLayout视图对象。

>（三）SQLite数据库性能优化

虽然大多数应用不会是SQLite的重度使用者，与数据库打交道少。比如如下SQL语句：
> CREATE TABLE user (name TEXT,password TEXT)
INSERT INTO user VALUES ('mrljdx','hellosql')

创建一个user表，有name和password两列，然后插入用户数据。当执行SQLite时，如下，SQLite内部是编译执行的：
```java
SQLiteDAtabase db = SQLiteDataBase.create(null);//数据库在内存中
db.execSQL("CREATE TABLE user(name TEXT,password TEXT)");
db.execSQL("INSERT INTO user VALUES ('mrljdx','hellosql')");
db.close();//操作完成后，关闭数据库
```
简历数据库的最优方案如下：
```java
public void createUserTable(){
	SQLiteStatement stmt = db.complieStatement("INSERT INTO user VALUES(?,?)");
	int i =0 ;
	for(String name:sUserNames){
		String pwd = sPwds[i++];
		stmt.clearBindings();
		stmt.bindString(1,name);//替换第一个?号
		stmt.bindString(2,pwd);//替换第二个?号
		stmt.executeInsert();
	}
}
```
由于只进行了一次语句编译，并且绑定值是比编译更轻量级的操作，所以，这个方法比那种在循环内不断执行SQL语句编译的方式要好很多。而且使代码更具可读性。
在Android中也定义了一些可以提高性能的类。比如使用DatabaseUtils.InsertHelper在数据库中插入多行，这样也只需编译一次INSERT语句。和上述的方法类似。

>（四） SQLite事务的原子提交特性及性能

事务就是将一组动作当作一个整体，比如在插入操作中，有一条记录插入失败，则意味着数据库的所有插入的行为都失败。要么没有插入任何一条数据，不会只建立部分表。
```java
public void createUserTable(){
	try{
		db.beginTransaction(); //开始事务
		SQLiteStatement stmt = db.complieStatement("INSERT INTO user VALUES(?,?)");
		int i =0 ;
		for(String name:sUserNames){
			String pwd = sPwds[i++];
			stmt.clearBindings();
			stmt.bindString(1,name);//替换第一个?号
			stmt.bindString(2,pwd);//替换第二个?号
			stmt.executeInsert();
		}
		db.setTransactionSuccessful();//删除这一调用则不会提交任何改动
	}catch(Exception e){
		//处理异常
	}finally{
		db.endTransaction();//结束事务
	}
}
```
在插入800条用户数据时，使用事务插入数据，比上面不使用要快将近(100毫秒)。
以上数据库是在内存中，而不是保存在持久存储（SD卡或ROM）上。在数据库工作时，大量的时间花费在访问持久存储的读/写上。如果不使用事务，则会使数据库上的耗时明显增加。

> （五）限制数据库的访问方式来加快查询速度

数据库查询中，会返回一个Cursor（游标）对象，然后用它来遍历结果。以下比较两种遍历数据库的方法：
方法一：创建Cursor获取数据库中的两列数据：
```java
public void iterateBothColums(){
	Cursor c = db.query("user",null,null,null,null,null,null);
	if(c.moveToFirst){
		do{
		}while(c.moveToNext());
	}
	c.close;//结束时一定要关闭Cursor否则会出现异常
}
```
方法二：创建的Cursor只获取第一列
```java
public void iterateFirstColum(){
	Cursor c = db.query("user",new String[]{"name"},null,null,nul,null,null);//与一的唯一区别
	if(c.moveToFirst){
		do{
		}while(c.moveToNext());
	}
	c.close;//结束时一定要关闭Cursor否则会出现异常
}
```
由于没有从第二列读取数据，第二种方法比第一种方法要快不少。迭代所有行时，将所有行作为一个事务处理会更快。由此可以发现，值读取需要的数据才是王道。调用查询时，选择正确的参数，可以使性能有客观的提升。只需要指定数量的行，指定调用查询的次数，则可以进一步减少数据库的访问时间。

## END
由于个人水平有限，文中不免有错误之处，还请指正。

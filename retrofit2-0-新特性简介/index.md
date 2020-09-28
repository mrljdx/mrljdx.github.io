# Retrofit2.0 新特性简介


> Retrofit-2.0.0-beta3在2016-01-05号发布，由于是测试版所以想把Retrofit-1.x+Okhttp-2.x 升级到 Retrofit-2.x+OkHttp-3.x 的同学们请慎重，因为改动很大。

## 前言
Retrofit 作为简化 HTTP 请求的库，已经运行多年，很多公司也都有采用Retrofit+OkHttp来作为网络请求框架。最近反编译了几个应用以 [英语流利说](http://www.liulishuo.com/) 和 [豌豆荚一览](http://www.wandoujia.com/yilan?utm_expid=103241880-19.gRjQTfyBQdKyoLM1pJIkTA.0) （PS:免费广告~）为例，发现这两个应用也是使用的Retrofit+OkHttp的网络请求框架。

在Android网络请求框架中，个人比较偏爱[Spring Android](http://projects.spring.io/spring-android/) 特别是结合[AndroidAnnotations](https://github.com/excilys/androidannotations/wiki/Rest%20API)一起使用，那感觉很棒。感兴趣的朋友可以去研究一下SpringAndroid。
跑题了，下面来谈谈Retrofit2.0的新特性

<!--more-->

---------------------

## Retrofit 2.0 新特性

### Retrofit2 有了新的类型 Call
由于Retrofit2 开始依赖于OkHttp，熟悉OkHttp应该知道，OkHttp定义了一个Call接口，官方Api文档说明如下：
> A call is a request that has been prepared for execution. A call can be canceled. As this object represents a single request/response pair (stream), it cannot be executed twice.

现在， Retrofit 2 里也多了一个 Call<T> 接口，其中T为请求成功后返回的数据类型。语法和 OkHttp 基本一模一样，唯一不同是这个函数知道如何做数据的反序列化。它知道如何将 HTTP 响应转换成对象。
另一个大的进步是 2.0 同时支持了在一个类型中的同步和异步。同时，一个请求也可以被真正地终止。终止操作会对底层的 http client 执行 cancel 操作。即便是正在执行的请求，也能立即切断。
> 以下代码来自：https://realm.io/cn/news/droidcon-jake-wharton-simple-http-retrofit-2/
	
	Call<List<Contributor>> call = gitHubService.repoContributors("square", "retrofit");

	call.enqueue( new Callback<List<Contributor>>() {
				  @Override void onResponse(/* ... */) {
				    // ...
				  }

				  @Override void onFailure(Throwable t) {
				    // ...
				  }
				} );//异步请求
	// or...
	call.execute();//同步请求

	// later...
	call.cancel(); //取消请求


注意，以上代码其实是错误的，因为Retrofit2中说明，Call 对象是从你的 API 接口返回的参数化后的对象。调用跟接口名相同的函数名，你就会得到一个实例出来。我们可以直接调用它的 execute 方法，但是得留意一下，这个方法只能调用一次。
如果你尝试第二次调用，会出现错误：
	
> Exception in thread "main" java.lang.IllegalStateException: Already executed.

实际上，可以直接克隆一个实例，代价非常低。当你想要多次请求一个接口的时候，直接用 clone 的方法来生产一个新的相同的可用对象。
修改以上代码：
	
	Call<List<Contributor>> call = gitHubService.repoContributors("square", "retrofit");
	Call<List<Contributor>> cloneCall = call.clone(); //克隆相同请求
	call.enqueue( new Callback<List<Contributor>>() {
				  @Override void onResponse(/* ... */) {
				    // ...
				  }

				  @Override void onFailure(Throwable t) {
				    // ...
				  }
				} );//异步请求
	// or...
	cloneCall.execute();//同步请求

	// later...
	call.cancel(); //取消请求
	cloneCall.cancel(); //取消请求

简单比较一下Retrofit 1.x 和 Retrofit 2.x 的一步请求定义：

**定义User类**
	
	public class User {  
	    private String name;
	    private String action;

	    public User() {}
	    public User(String name, String action) {
	        this.name = name;
	        this.action = action;
	    }
	}

**Retrofit 1.x **

定义 ApiService
	
	public interface UserService {  
	    @POST("/create")
	    void createUser(@Body User user, Callback<User> callback); //定义异步请求
	}

调用：

	User user = new User("Mrljdx", "Write Code ");  
	userService.createUser(user, new Callback<User>)() {});  

**Retrofit 2.x **
	
定义 ApiService
	
	public interface TaskService {  
	    @POST("/create")
	    Call<User> createUser(@Body User task); //定义异步请求,
	}

调用:

	User user = new User("Mrljdx", "Write Code");  
	Call<Task> call = userService.createUser(user);  
	call.enqueue(new Callback<Task>() {});  //Retrofit 2.x 异步请求

**在Retrofit2中，同步请求和异步请求都可以定义为同一个，不需要另外为了一个异步请求另外再写一个异步请求方法。**

关于Retrofit 1.x 和Retrofit2.x的比较不在本文的讨论范围之类，具体请参考：

> https://futurestud.io/blog/retrofit-getting-started-and-android-client

--------------------

### 新的Respone类型
在Retrofit2中对Response做了一点改变，就是你可以根据Response来获取请求返回的状态码、响应信息和头信息。
查询Retrofit中Response源码如下：
	
	/** An HTTP response. */
	public final class Response<T> {
	  /** Create a synthetic successful response with {@code body} as the deserialized body. */
	  public static <T> Response<T> success(T body) {
	    return success(body, new okhttp3.Response.Builder() //
	        .code(200)
	        .message("OK")
	        .protocol(Protocol.HTTP_1_1)
	        .request(new Request.Builder().url("http://localhost").build())
	        .build());
	  }

	  /**
	   * Create a successful response from {@code rawResponse} with {@code body} as the deserialized
	   * body.
	   */
	  public static <T> Response<T> success(T body, okhttp3.Response rawResponse) {
	    if (rawResponse == null) throw new NullPointerException("rawResponse == null");
	    if (!rawResponse.isSuccessful()) {
	      throw new IllegalArgumentException("rawResponse must be successful response");
	    }
	    return new Response<>(rawResponse, body, null);
	  }

	  /**
	   * Create a synthetic error response with an HTTP status code of {@code code} and {@code body}
	   * as the error body.
	   */
	  public static <T> Response<T> error(int code, ResponseBody body) {
	    if (code < 400) throw new IllegalArgumentException("code < 400: " + code);
	    return error(body, new okhttp3.Response.Builder() //
	        .code(code)
	        .protocol(Protocol.HTTP_1_1)
	        .request(new Request.Builder().url("http://localhost").build())
	        .build());
	  }

	  /** Create an error response from {@code rawResponse} with {@code body} as the error body. */
	  public static <T> Response<T> error(ResponseBody body, okhttp3.Response rawResponse) {
	    if (body == null) throw new NullPointerException("body == null");
	    if (rawResponse == null) throw new NullPointerException("rawResponse == null");
	    if (rawResponse.isSuccessful()) {
	      throw new IllegalArgumentException("rawResponse should not be successful response");
	    }
	    return new Response<>(rawResponse, null, body);
	  }

	  private final okhttp3.Response rawResponse;
	  private final T body;
	  private final ResponseBody errorBody;

	  private Response(okhttp3.Response rawResponse, T body, ResponseBody errorBody) {
	    this.rawResponse = rawResponse;
	    this.body = body;
	    this.errorBody = errorBody;
	  }

	  /** The raw response from the HTTP client. */
	  public okhttp3.Response raw() {
	    return rawResponse;
	  }

	  /** HTTP status code. */
	  public int code() {
	    return rawResponse.code();
	  }

	  /** HTTP status message or null if unknown. */
	  public String message() {
	    return rawResponse.message();
	  }

	  /** HTTP headers. */
	  public Headers headers() {
	    return rawResponse.headers();
	  }

	  /** {@code true} if {@link #code()} is in the range [200..300). */
	  public boolean isSuccess() {
	    return rawResponse.isSuccessful();
	  }

	  /** The deserialized response body of a {@linkplain #isSuccess() successful} response. */
	  public T body() {
	    return body;
	  }

	  /** The raw response body of an {@linkplain #isSuccess() unsuccessful} response. */
	  public ResponseBody errorBody() {
	    return errorBody;
	  }
	}

通过源码，可以看到Response对于请求后的返回成功和错误进行了处理，通过公共方法可以获取到``Http status code``、``Http status message``和``Http headers``其中``body()``方法会返回请求的返回信息反序列化的类型对象。
同时还提供了一个很方便的函数``isSuccess()``来帮助你判断请求是否成功完成，其实就是检查了下响应码是不是 200。然后就拿到了响应的 body 部分，另外有一个单独的方法获取 error body。基本上就是出现一个返回码，然后调用相对应的函数的。只有当响应成功以后，我们会去做反序列化操作，然后将反序列化的结果放到 body 回调中去。如果出现了返回了网络成功响应（返回码：200）却最终返回 false 的情况，我们实际上是无法判断返回到底是什么的，只能将 ResponseBody（简单封装的了 content-type，length，以及 raw body部分） 类型交给你去处理。

------------

### 请求的URL可以动态配置

在Retrofit2中使用 ** @Url ** 即可实现URL的动态配置，比如定义一个``LeanCloudApi``
	
	//定义一个LeanCloud的Api 定义leanCloudLogin方法
	public interface LeanCloudApis {

        @POST
        Call<LoginResultBean> leanCloudLogin(
                @Body HashMap<String, Object> map,
                @Url String url
        );
    }
    //在这里使用 ``https://api.leancloud.cn/1.1/login`` 替换默认的URL，通过自定义拦截器可以看到请求的URL已经变成了传入的url地址。
    Call<LoginResultBean> response = leanCloudApis.leanCloudLogin(map, "https://api.leancloud.cn/1.1/login");

拦截的log日志如下：

> Request Url is :https://api.leancloud.cn/1.1/login
Method is : POST
Request Body is :okhttp3.RequestBody$1@4c203ea1

** PS：个人觉得 Retrofit2 在这个动态配置URL的技能上感觉确实很吊，不过这个在``SpringAndroid 1``中就已经有了。当然感兴趣的同学可以考虑尝试一下SpringAndroid，不过SpringAndroid目前暂不支持RxJava如果需要支持，则要自己实现一个请求结果转换类。在这篇文章中就不深入解释了。**


-------------

### 支持多种 Converter 并存

> PS:在SpringAndroid 1.0 中早已实现，目前Spring Android最新版本为2.0.0M3 支持 OkHttp。关于SpringAndroid2 和 Retrofit2的比较以后再写。

在Retrofit 1.x中，Converter序列化默认为Gson解析，当然也可以使用Jackson替换。
在Retrofit 1.x版本中如果你遇到这种情况：一个 API 请求返回的结果需要通过 JSON 反序列化，另一个 API 请求需要通过 proto 反序列化，唯一的解决方案就是将两个接口分离开声明。
	
	interface SomeProtoService {
	  @GET("/some/proto/endpoint")
	  Call<SomeProtoResponse> someProtoEndpoint();
	}

	interface SomeJsonService {
	  @GET("/some/json/endpoint")
	  Call<SomeJsonResponse> someJsonEndpoint();
	}

之所以搞得这么麻烦是因为一个 REST adapter 只能绑定一个 Converter 对象。返回格式的差异不应该成为分组时候的阻碍。所以在Retrofit 2.x中可以将不同返回格式的请求放到同一个RestService中。

	interface SomeService {
	  @GET("/some/proto/endpoint")
	  Call<SomeProtoResponse> someProtoEndpoint();

	  @GET("/some/json/endpoint")
	  Call<SomeJsonResponse> someJsonEndpoint();
	}

原理也很简单，就是当返回数据时，会根据你定义的converter来判断能否处理某种返回类型。比如``Protobuff``类型的数据都是继承自``message`` 或者 ``message lite`` 的类。根据判断这个类是否继承自``message`` 或者 ``message lite`` 即可处理。就会使用Protobuf的converter来处理返回数据。
如果返回数据是JSON类型的，那么会检查``Protobuff``能不能处理，如果不能处理则交给JSON的converter处理（比如GsonConverter或JacksonConverter）。由于JSON 并没有什么继承上的约束。所以我们无法通过什么确切的条件来判断一个对象是否是 JSON 对象。所以JSON converter 一定要放在最后，不然会预期返回不符。（这也是容易出问题的地方）
> Retrofit 2.x 中去掉了默认的Converter，如果你想在工程中使用相应的Converter则需要导入相应的包；

** 在Gradle导入Converter：**

	//Retrofit 2.0.0-beta3
	compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta3'
    compile 'com.squareup.retrofit2:converter-protobuf:2.0.0-beta3'
    compile 'com.squareup.retrofit2:converter-jackson:2.0.0-beta3'
    compile 'com.squareup.retrofit2:converter-moshi:2.0.0-beta3'
    compile 'com.squareup.retrofit2:converter-wire:2.0.0-beta3'
    compile 'com.squareup.retrofit2:converter-simplexml:2.0.0-beta3'
    compile 'com.squareup.retrofit2:converter-scalars:2.0.0-beta3'

查看最新的可以去MavenRepository中查找[Retrofit2](http://mvnrepository.com/artifact/com.squareup.retrofit2)


下面以修改的官方Sample中的一个示例简单说明：(在IDEA中运行)
示例说明：
	
	import java.io.IOException;
	import java.util.List;

	import retrofit2.*;
	import retrofit2.http.GET;
	import retrofit2.http.Path;
	/**
	 * Created by Mrljdx on 16/1/7.
	 * Retrofit 2.0 和 OkHttp的使用
	 */
	public class SimpleService {

	    public static final String API_URL = "https://api.github.com";

	    public static class Contributor {
	        public final String login;
	        public final int contributions;

	        public Contributor(String login, int contributions) {
	            this.login = login;
	            this.contributions = contributions;
	        }
	    }

	    public interface GitHub {
	        @GET("/repos/{owner}/{repo}/contributors")
	        Call<List<Contributor>> contributors(
	                @Path("owner") String owner,
	                @Path("repo") String repo);
	    }

	    public static void main(String... args) throws IOException {
	        // Create a very simple REST adapter which points the GitHub API.
	        Retrofit retrofit = new Retrofit.Builder()
	                .baseUrl(API_URL)
	                //ProtoBuf converter
	                .addConverterFactory(ProtoConverterFactory.create())
	                //XML converter
	                .addConverterFactory(SimpleXmlConverterFactory.create())
	                //Moshi converter
	                .addConverterFactory(MoshiConverterFactory.create())
	                //Scalar converter
	                .addConverterFactory(ScalarsConverterFactory.create())
	                //JSON converter
	                .addConverterFactory(GsonConverterFactory.create())
	                .addConverterFactory(JacksonConverterFactory.create())
	                .build();

	        // Create an instance of our GitHub API interface.
	        GitHub github = retrofit.create(GitHub.class);

	        // Create a call instance for looking up Retrofit contributors.
	        Call<List<Contributor>> call = github.contributors("square", "retrofit");

	        // Fetch and print a list of the contributors to the library.
	        List<Contributor> contributors = call.execute().body();
	        for (Contributor contributor : contributors) {
	            System.out.println(contributor.login + " (" + contributor.contributions + ")");
	        }
	    }

	}

以上的例子比较极端，一次性加入了所有的``Converters``，在实际编码时，其实最多也只会用到``SimpleXmlConverterFactory``、``GsonConverterFactory``或者``JacksonConverterFactory``根据取舍，如果你想一次性加入所有的``Converters``则会在实际的项目中导致一些错误。
根据实际经验发现，Retrofit2发送请求如下：

> <font color="#FF4040">AppClient Request ---> Retrofit 2 --> Conveters --> Retrofit 2 ---> OkHttp3 --> WebService handle Request </font>
> <font color="#90EE90">WebSevice Response --> OkHttp3 --> Retrofit2 --> Conveters --> Retrofit2 --> AppClient handle Response </font>
	


-----------------

### 默认依赖OkHttp

现在当你在Gradle中加入 ``compile 'com.squareup.retrofit2:retrofit:2.0.0-beta3'``的依赖后，会默认下载``OkHttp3`` 和 ``Okio`` 。由于OkHttp库是一款非常优秀的网络请求库有取代Apache HttpClient网络库的趋势，貌似在Android 6.0中Google官方也默认支持OkHttp，可见这款库的优秀之处，另外由于依赖于OkHttp库使得Retrofit不用自己去实现一些网络请求，减少了整个Retrofit的60%体积大小。这个很给力啊！
OkHttp 的背后是一个叫做 Okio 的库，提供的 IO 支持。这是众多 IO 库更好的选择而且极度高效。（PS:谁用谁知道)
前面提到Retrofit中可以自定义拦截器，通过自定义拦截器，可以用于处理在发送请求或接受回复时对数据进行处理和判断。
大致的流程如下：
> AppClient Request --> Retrofit 2 --> Intercepter handle request --> excute()--> WebService handle Request and return Response --> Intercepter handle response --> Retrofit2-->AppClient get Response

在代码中实现如下：
定义自己的拦截器``MyInterceptors``，具体看注释：

	import okhttp3.*;

	import java.io.IOException;

	/**
	 * Created by Mrljdx on 16/1/8.
	 */
	public class MyInterceptors implements Interceptor {
	    @Override
	    public Response intercept(Chain chain) throws IOException {

	        //封装headers
	        Request request = chain.request().newBuilder()
	                .addHeader("Content-Type", "application/json") //添加请求头信息
	                .build();
	        Headers headers = request.headers();
	        //打印
	        System.out.println("Content-Type is : " + headers.get("Content-Type"));
	        String requestUrl = request.url().toString(); //获取请求url地址
	        String methodStr = request.method(); //获取请求方式
	        RequestBody body = request.body(); //获取请求body
	        String bodyStr = (body==null?"":body.toString());
	        //打印Request数据
	        System.out.println("Request Url is :" + requestUrl + "\nMethod is : " + methodStr + "\nRequest Body is :" + bodyStr + "\n");

	        Response response = chain.proceed(request);
	        if (response != null) {
	            System.out.println("Response is not null");
	        } else {
	            System.out.println("Respong is null");
	        }
	        return response;

	    }
	}

在定义拦截器之后使用拦截器，贴出部分代码片段：

	OkHttpClient client = new OkHttpClient().newBuilder()
											.addInterceptor(new MyInterceptors())
											.build();
	Retrofit retrofit = new Retrofit.Builder()
        .baseUrl(API_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .client(client) //设置OkHttp Client
        .build();

在一般的商业应用开发中，对于拦截器的作用在于处理返回信息，对请求数据加密或者存储cookies等。在这种情况下，本文定义的拦截器基本上可以满足一般应用的需求。简单的应用开发只需要封装 access token 便于服务器验证即可。

---------------------

## END

### 小结
新的Retrofit版本已经发布测试版，目前市面上的一些公司也大都在使用Retrofit+OkHttp的网络请求框架，对于一些初学者可以根据市场的需求来针对性的学习Android网络请求方面的内容，基础是一方面，了解和掌握一些比较好的开发框架也是很重要的。
其实开发框架就是让你的代码变得更加高效和简洁，便于维护和开发。要学会接受一些新的框架和别人的设计思想，好了,不扯淡了~ 容易引起不适。
由于Mrljdx能力有限，文中如有错误之处望请斧正。互相交流，开卷有益。^_^

### 参考资料

1. [用 Retrofit 2 简化 HTTP 请求](https://realm.io/cn/news/droidcon-jake-wharton-simple-http-retrofit-2/)
2. [Retrofit 2.x Api文档](http://square.github.io/retrofit/2.x/retrofit/)
3. [OkHttp3.x Apix文档](http://square.github.io/okhttp/3.x/okhttp/okhttp3/Interceptor.Chain.html)
4. [Retrofit 2.0: The biggest update yet on the best HTTP Client Library for Android](http://inthecheesefactory.com/blog/retrofit-2.0/en)
5. [okHttp Change Log](https://github.com/square/okhttp/blob/master/CHANGELOG.md)
6. [Retrofit Change Log](https://github.com/square/retrofit/blob/master/CHANGELOG.md)
7. [Jakewharton Blog](http://jakewharton.com/java-interoperability-policy-for-major-version-updates/)
8. [Retrofit1.x 和 Retrofit 2.x的比较（英文）](https://futurestud.io/blog/retrofit-getting-started-and-android-client)

------------------------



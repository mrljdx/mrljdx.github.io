# AA中@Background实现原理详解

AndroidAnnotations中引入了@Background的注解方法,极大的简化了编写异步代码的工作量.使我们摆脱了麻烦的AsyncTask.
下面我们来看一下官方介绍中,介绍的一个关于获取书签(BookMarks)的请求代码:
<!--more-->

## 使用AsyncTask
    
    public class BookmarksToClipboardActivity extends Activity {
        //do some ...
        
        //点击更新书签
        void updateBookmarksClicked() {
            UpdateBookmarksTask task = new UpdateBookmarksTask();
         
            task.execute(search.getText().toString(), application.getUserId());
        
        }
        //书签URL
        private static final String BOOKMARK_URL = "http://www.bookmarks.com/bookmarks/{userId}?search={search}";
        //异步网络请求,从服务器获取书签
        class UpdateBookmarksTask extends AsyncTask<String, Void, Bookmarks> {
         
            @Override
            protected Bookmarks doInBackground(String... params) {
              String searchString = params[0];
              String userId = params[1];
              //这里使用SpringAndroid作为网络请求框架
              RestTemplate client = new RestTemplate();
              //封装url请求数据
              HashMap<String, Object> args = new HashMap<String, Object>();
              args.put("search", searchString);
              args.put("userId", userId);
             
              HttpHeaders httpHeaders = new HttpHeaders();
              HttpEntity<Bookmarks> request = new HttpEntity<Bookmarks>(httpHeaders);
              //获取网络Get请求后的返回值
              ResponseEntity<Bookmarks> response = client.exchange(BOOKMARK_URL, HttpMethod.GET, request, Bookmarks.class, args);
              Bookmarks bookmarks = response.getBody();
              //异步返回结果值
              return bookmarks;
            }
         
            @Override
            protected void onPostExecute(Bookmarks result) {
              //处理返回结果
              adapter.updateBookmarks(result);
              bookmarkList.startAnimation(fadeIn);
            }
            
        }
        
        // do some ...
    }
     
## 使用@Background
    
** 定义请求客户端 **

    //关于@Rest see @ https://github.com/excilys/androidannotations/wiki/Rest-API#rest
    @Rest("http://www.bookmarks.com")
    public interface BookmarkClient {
      
      @Get("/bookmarks/{userId}?search={search}")
      Bookmarks getBookmarks(String userId,String search);
     
    }
    
** 在@EActivity下使用代码片段 **
    
    @EActivity
    public class BookmarksToClipboardActivity extends Activity {
            
        //do some ...
    
        //点击更新书签
        @Click({R.id.updateBookmarksButton1, R.id.updateBookmarksButton2})
        void updateBookmarksClicked() {
            searchAsync(search.getText().toString(), application.getUserId());
        }
        
        //申明&网络请求实例化
        @RestService
        BookmarkClient restClient;
        
        //异步网络请求
        @Background
        void searchAsync(String userId, String searchString) {
            //获取返回结果
            Bookmarks bookmarks = restClient.getBookmarks(userId, searchString);
            //处理返回结果
            updateBookmarks(bookmarks);
        }
           
        //Ui线程更新操作
        @UiThread
        void updateBookmarks(Bookmarks bookmarks) {
            adapter.updateBookmarks(bookmarks);
            bookmarkList.startAnimation(fadeIn);
        }
        
        //do some ..
    }

## @Background的实现原理分析

### 生成的注解代码
AndroidAnnotations是一个在编译时生成代码的注解框架,以上面的网络请求为例,在使用@Background注解的代码如下:
    
    public final class LoginActivity_ extends LoginActivity
        implements HasViews, OnViewChangedListener{
        
        //do some ..
        
        //生成的代码
        @Override
        public void searchAsync(final String userId, final String searchString){
            //BackgroundExceutor 维护了一个线程池
            BackgroundExecutor.execute(new BackgroundExecutor.Task("", 0, "") {
        
                    @Override
                    public void execute() {
                        try {
                            BookmarksToClipboardActivity_.super.searchAsync(userId,searchString);
                        } catch (Throwable e) {
                            Thread.getDefaultUncaughtExceptionHandler().uncaughtException(Thread.currentThread(), e);
                        }
                    }
        
            }
        }
        
        //do some ...
    }

### BackgroundExecutor 中的关键方法
    
    public class BackgroundExecutor {
    
        private static final String TAG = "BackgroundExecutor";
        
        //默认的Executor执行数量
        public static final Executor DEFAULT_EXECUTOR = Executors.newScheduledThreadPool(2 * Runtime.getRuntime().availableProcessors());
        private static Executor executor = DEFAULT_EXECUTOR;
        
        //执行的任务集合,用于管理后台线程任务集合
        private static final List<Task> TASKS = new ArrayList<Task>();
        private static final ThreadLocal<String> CURRENT_SERIAL = new ThreadLocal<String>();
           
        //BackgroundExceutor不能被实例化
        private BackgroundExecutor() {
        }
        
        public static abstract class Task implements Runnable {
        
        		private String id;
        		private int remainingDelay;
        		private long targetTimeMillis; /* since epoch */
        		private String serial;
        		private boolean executionAsked;
        		private Future<?> future;
        
        		/*
        		 * A task can be cancelled after it has been submitted to the executor
        		 * but before its run() method is called. In that case, run() will never
        		 * be called, hence neither will postExecute(): the tasks with the same
        		 * serial identifier (if any) will never be submitted.
        		 * 
        		 * Therefore, cancelAll() *must* call postExecute() if run() is not
        		 * started.
        		 * 
        		 * This flag guarantees that either cancelAll() or run() manages this
        		 * task post execution, but not both.
        		 */
        		private AtomicBoolean managed = new AtomicBoolean();
                
                //对后台任务赋值,判断是否需要延迟执行,以及任务的id和执行序列
        		public Task(String id, int delay, String serial) {
        			if (!"".equals(id)) {
        				this.id = id;
        			}
        			if (delay > 0) {
        				remainingDelay = delay;
        				targetTimeMillis = System.currentTimeMillis() + delay;
        			}
        			if (!"".equals(serial)) {
        				this.serial = serial;
        			}
        		}
        
        		@Override
        		public void run() {
        			if (managed.getAndSet(true)) {
        				/* cancelled and postExecute() already called */
        				return;
        			}
        
        			try {
        				CURRENT_SERIAL.set(serial);
        				execute();
        			} finally {
        				/* handle next tasks */
        				postExecute();
        			}
        		}
        
        		public abstract void execute();
        
        		private void postExecute() {
        			if (id == null && serial == null) {
        				/* nothing to do */
        				return;
        			}
        			CURRENT_SERIAL.set(null);
        			synchronized (BackgroundExecutor.class) {
        				/* execution complete */
        				TASKS.remove(this);
        
        				if (serial != null) {
        					Task next = take(serial);
        					if (next != null) {
        						if (next.remainingDelay != 0) {
        							/* the delay may not have elapsed yet */
        							next.remainingDelay = Math.max(0, (int) (targetTimeMillis - System.currentTimeMillis()));
        						}
        						/* a task having the same serial was queued, execute it */
        						BackgroundExecutor.execute(next);
        					}
        				}
        			}
        		}
        
        }
        
        //执行后台任务,将任务添加到后台队列
        public static synchronized void execute(Task task) {
        	Future<?> future = null;
        	//判断是否
        	if (task.serial == null || !hasSerialRunning(task.serial)) {
        		task.executionAsked = true;
        		future = directExecute(task, task.remainingDelay);
        	}
        	if (task.id != null || task.serial != null) {
        		/* keep task */
        		task.future = future;
        		TASKS.add(task);
        	}
        }
        
    }
    
通过分析BackgroundExecutor并结合AA的[@Background](https://github.com/excilys/androidannotations/wiki/WorkingWithThreads#background)可以了解到
@Background让我们拜托了AsyncTask的束缚,使用@Background注解的方法并不会立即执行,而是会加入到** BackgroundExecutor**维护的执行线程池中.这也表示如果同时调用
两个被@Background注解的方法,他们会并行执行.

## @Background用法

### 注解方法
如果一个方法被@Background注解,那么表示这个方法会在一个非UI线程的线程中执行.
示例如下:
    
    void myMethod() {
        someBackgroundWork("hello", 42);
    }
    
    @Background
    void someBackgroundWork(String aParam, long anotherParam) {
        [...]
    }
    
### Id
在我们平时使用AyncTask时会面临一个问题,就是如果执行异步任务,那么怎么才能取消这个任务呢?这也是AsyncTask的一个设计缺陷,不过@Background解决了这个问题,就是使用Id.
如果你想取消任何一个后台执行的任务, 可以使用 ** BackgroundExecutor.cancelAll("id");**方法.
示例如下:
    
    void myMethod() {
        //执行一个后台任务
        someCancellableBackground("hello", 42);
        [...]
        boolean mayInterruptIfRunning = true;
        //取消执行id = "cancellable_task" 的任务 
        BackgroundExecutor.cancelAll("cancellable_task", mayInterruptIfRunning);
    }
    
    @Background(id="cancellable_task")
    void someCancellableBackground(String aParam, long anotherParam) {
        [...]
    }

一般取消执行任务的情况有,屏幕横竖屏切换,或者资源突然被回收了,亦或者从某一个加载页面返回需要取消当前页的请求等等.总之有效地控制后台任务能带来更好地用户体验以及提升应用性能.

### Serial
在一般情况下,@Background 注解的方法调用时会并行执行,没有先后顺序.如果你想要这些后台执行的任务有序执行,那么你可以定义 Serial 的值来实现,所有具有相同Serial的任务都会有序执行.
示例如下:
    
    void myMethod() {
        for (int i = 0; i < 10; i++)
            someSequentialBackgroundMethod(i);
    }
    
    @Background(serial = "test")
    void someSequentialBackgroundMethod(int i) {
        SystemClock.sleep(new Random().nextInt(2000)+1000);
        Log.d("AA", "value : " + i);
    }

那么为什么定义相同的 serial 就可以按照实例化的顺序执行呢?回到之前的``BackgroundExecutor`` 我们可以看到同步执行代码片段:

    synchronized (BackgroundExecutor.class) {
            				/* execution complete */
            				TASKS.remove(this);
            
            				if (serial != null) {
            					Task next = take(serial);
            					if (next != null) {
            						if (next.remainingDelay != 0) {
            							/* the delay may not have elapsed yet */
            							next.remainingDelay = Math.max(0, (int) (targetTimeMillis - System.currentTimeMillis()));
            						}
            						/* a task having the same serial was queued, execute it */
            						BackgroundExecutor.execute(next);
            					}
            				}
    }
    
这里会判断当前的serial 是否为空,如果不为空则会一个一个的从``List<Task> TASKS``中取出任务来有序执行.

### Delay
如果你想在执行一个后台任务之前加个延迟执行,你可以使用``delay``参数
示例如下:
    
    @Background(delay=2000)
    void doInBackgroundAfterTwoSeconds() {
    }

具体原理可以查看``BackgroundExecutor`` 中的代码:
    
    private void postExecute() {
    			if (id == null && serial == null) {
    				/* nothing to do */
    				return;
    			}
    			CURRENT_SERIAL.set(null);
    			synchronized (BackgroundExecutor.class) {
    				/* execution complete */
    				TASKS.remove(this);
    
    				if (serial != null) {
    					Task next = take(serial);
    					if (next != null) {
    					    
    						if (next.remainingDelay != 0) {
    							/* the delay may not have elapsed yet */
    							next.remainingDelay = Math.max(0, (int) (targetTimeMillis - System.currentTimeMillis()));
    						}
    						/* a task having the same serial was queued, execute it */
    						BackgroundExecutor.execute(next);
    					}
    				}
    			}
    }
    
## 相关资料
1. [AndroidAnnotations官网](http://androidannotations.org/)
2. [@Background](https://github.com/excilys/androidannotations/wiki/WorkingWithThreads#background)

以上就是@Background的使用相关的问题和原理分析.如果纰漏还请斧正.多多交流,开卷有益.O(∩_∩)O~

------------------
  


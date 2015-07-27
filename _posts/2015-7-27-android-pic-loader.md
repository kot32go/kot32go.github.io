---
layout: post
title: 本地图片加载框架分析
description: "图片加载框架"
tags: [Android]
---

##首先要了解的是Looper、Handler、Message 等之间的关系


Message：消息，其中包含了消息ID，消息处理对象以及处理的数据等，由MessageQueue统一列队，终由Handler处理。

Handler：处理者，负责Message的发送及处理。使用Handler时，需要实现handleMessage(Message msg)方法来对特定的Message进行处理，例如更新UI等。

MessageQueue：消息队列，用来存放Handler发送过来的消息，并按照FIFO规则执行。当然，存放Message并非实际意义的保存，而是将Message以链表的方式串联起来的，等待Looper的抽取。

Looper：消息泵，不断地从MessageQueue中抽取Message执行。因此，一个MessageQueue需要一个Looper。

Thread：线程，负责调度整个消息循环，即消息循环的执行场所。

Android系统的消息队列和消息循环都是针对具体线程的，一个线程可以存在（当然也可以不存在）一个消息队列和一个消 息循环（Looper），特定线程的消息只能分发给本线程，不能进行跨线程，跨进程通讯。但是创建的工作线程默认是没有消息循环和消息队列的，如果想让该 线程具有消息队列和消息循环，需要在线程中首先调用Looper.prepare()来创建消息队列，然后调用Looper.loop()进入消息循环。 如下例所示：
class LooperThread extends Thread {
      public Handler mHandler;

      public void run() {
          Looper.prepare();

          mHandler = new Handler() {
              public void handleMessage(Message msg) {
                  // process incoming messages here
              }
          };

          Looper.loop();
      }
  }

这样你的线程就具有了消息处理机制了，在Handler中进行消息处理。

 几点小结

·      Handler的处理过程运行在创建Handler的线程里

·      一个Looper对应一个MessageQueue

·      一个线程对应一个Looper

·      一个Looper可以对应多个Handler

·      不确定当前线程时，更新UI时尽量调用post方法

##其次，要了解Java 信号量Semaphore 的用法

Java 并发库 的Semaphore 可以很轻松完成信号量控制，Semaphore可以控制某个资源可被同时访问的个数，acquire()获取一个许可，如果没有就等待，而release()释放一个许可。比如在Windows下可以设置共享文件的最大客户端访问个数。 
Semaphore维护了当前访问的个数，提供同步机制，控制同时访问的个数。在数据结构中链表可以保存“无限”的节点，用Semaphore可以实现有限大小的链表。另外重入锁ReentrantLock也可以实现该功能，但实现上要负责些，代码也要复杂些。 

mPoolSemaphore = new Semaphore(5);

表示一次只允许5个线程访问某个资源
使用时，要获取许可和释放许可
获取许可：semaphore.tryAcquire(timeout, TimeUnit.MILLISECONDS);
释放许可：semaphore.release();  

##图片加载核心

刚刚说到了Looper和Thread ，图片加载的核心就是利用
初始化：

	mPoolThread = new Thread() {
    @Override
    public void run() {
        Looper.prepare();

        mPoolThreadHander = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                //在这里得到图片加载任务
                mThreadPool.execute(getTask());
                try {
                    mPoolSemaphore.acquire();
                } catch (InterruptedException e) {
                }
            }
        };

        // 释放一个信号量
        mSemaphore.release();
        Looper.loop();
    }
	};
	mPoolThread.start();
	
	
任务列表：

	mTasks = new LinkedList<Runnable>();
	
缓存：

	mLruCache = new LruCache<String, Bitmap>(cacheSize) {
   		@Override
    	protected int sizeOf(String key, Bitmap value) {
        	return value.getRowBytes() * value.getHeight();
    	}
	};
	
任务获取

	private synchronized Runnable getTask() {
    	if (mType == Type.FIFO) {
        	return mTasks.removeFirst();
    	} else if (mType == Type.LIFO) {
        	return mTasks.removeLast();
    	}
    	return null;
	}
	
任务添加：

	private synchronized void addTask(Runnable runnable) 	{
	    try {
	        // 请求信号量，防止mPoolThreadHander为null
	        if (mPoolThreadHander == null)
	            mSemaphore.acquire();
	    } catch (InterruptedException e) {
	    }
	    mTasks.add(runnable);

	    mPoolThreadHander.sendEmptyMessage(0x110);
	}

下面看加载图片的入口，也是将一切联系起来的关键：

public void loadImage(final String path, final ImageView imageView) {
	    // set tag
	    imageView.setTag(path);
	    // UI线程
	    if (mHandler == null) {
	        mHandler = new Handler() {
	            @Override
	            public void handleMessage(Message msg) {
	                ImgBeanHolder holder = (ImgBeanHolder) msg.obj;
	                ImageView imageView = holder.imageView;
	                Bitmap bm = holder.bitmap;
	                String path = holder.path;
	                if (imageView.getTag().toString().equals(path)) {
	                    imageView.setImageBitmap(bm);
	                }
	            }
	        };
	    }

	    Bitmap bm = getBitmapFromLruCache(path);
	    if (bm != null) {
	        ImgBeanHolder holder = new ImgBeanHolder();
	        holder.bitmap = bm;
	        holder.imageView = imageView;
	        holder.path = path;
	        Message message = Message.obtain();
	        message.obj = holder;
	        mHandler.sendMessage(message);
	    } else {
	        addTask(new Runnable() {
	            @Override
	            public void run() {

	                ImageSize imageSize = getImageViewWidth(imageView);

	                int reqWidth = imageSize.width;
	                int reqHeight = imageSize.height;

	                Bitmap bm = decodeSampledBitmapFromResource(path, reqWidth,
	                        reqHeight);
	                addBitmapToLruCache(path, bm);
	                ImgBeanHolder holder = new ImgBeanHolder();
	                holder.bitmap = getBitmapFromLruCache(path);
	                holder.imageView = imageView;
	                holder.path = path;
	                Message message = Message.obtain();
	                message.obj = holder;
	                // Log.e("TAG", "mHandler.sendMessage(message);");
	                mHandler.sendMessage(message);
	                mPoolSemaphore.release();
	            }
	        });
	    }

	}


可以看到，首先会根据path 去LruCache 中取，如果取不到，那么addTask到任务列表中。又从addTask()的源码中看到，添加后即会调用
mPoolThreadHander.sendEmptyMessage()

来取出任务进行执行，也就是运行run() 中的一系列代码

那么，为什么要使用Semaphore 呢？直接加载不就完了？其实是为了进行多线程的加载
Loader 类中有一个
threadCount
变量，在初始化的时候能够传入并设置
这个变量有三个作用
1.

	mThreadPool = Executors.newFixedThreadPool(threadCount);

在一开始的初始化线程池中,会传入一个线程数，表示这个线程池的容量
在初始化执行加载图片的代码中，

	mThreadPool.execute(getTask());

是将线程放入线程池中进行执行

而观察信号量的定义：

	mPoolSemaphore = new Semaphore(threadCount);

也就是说，最多允许threadCount个线程同时访问，这样加载速度就大大加快了，这里在程序中threadCount设置为3 ，均衡了速度和内存开销
     改进：可以根据手机内存总大小智能设置值

最后，图片的压缩

这里只看本地图片的压缩
我们知道，图片太大，显示的容器过小会造成资源的浪费，压缩的核心思想是把图片的大小控制到和容器一致
得到ImageView的大小：绘制过程不是在onCreate中调用的，因此在onCreate 中直接getWidth 和gitHeight 是获取不到的，在UniversalImageLoader 的源码中，有如下获取ImageView 大小的函数：

	private ImageSize getImageViewWidth(ImageView imageView) {
	    ImageSize imageSize = new ImageSize();
	    final DisplayMetrics displayMetrics = imageView.getContext()
	            .getResources().getDisplayMetrics();
	    final LayoutParams params = imageView.getLayoutParams();

	    int width = params.width == LayoutParams.WRAP_CONTENT ? 0 : imageView
	            .getWidth(); // Get actual image width
	    if (width <= 0)
	        width = params.width; // Get layout width parameter
	    if (width <= 0)
	        width = getImageViewFieldValue(imageView, "mMaxWidth"); // Check
	    // maxWidth
	    // parameter
	    if (width <= 0)
	        width = displayMetrics.widthPixels;
	    int height = params.height == LayoutParams.WRAP_CONTENT ? 0 : imageView
	            .getHeight(); // Get actual image height
	    if (height <= 0)
	        height = params.height; // Get layout height parameter
	    if (height <= 0)
	        height = getImageViewFieldValue(imageView, "mMaxHeight"); // Check
	    // maxHeight
	    // parameter
	    if (height <= 0)
	        height = displayMetrics.heightPixels;
	    imageSize.width = width;
	    imageSize.height = height;
	    return imageSize;

	}


	/**
	 * 反射获得ImageView设置的最大宽度和高度
	 *
	 * @param object
	 * @param fieldName
	 * @return
	 */
	private static int getImageViewFieldValue(Object object, String fieldName) {
	    int value = 0;
	    try {
	        Field field = ImageView.class.getDeclaredField(fieldName);
	        field.setAccessible(true);
	        int fieldValue = (Integer) field.get(object);
	        if (fieldValue > 0 && fieldValue < Integer.MAX_VALUE) {
	            value = fieldValue;

	            Log.e("TAG", value + "");
	        }
	    } catch (Exception e) {
	    }
	    return value;
	}


至于Bitmap的大小压缩过程：

	/**
	 * 计算inSampleSize，用于压缩图片
	 *
	 * @param options
	 * @param reqWidth
	 * @param reqHeight
	 * @return
	 */
	private int calculateInSampleSize(BitmapFactory.Options options,
	                                  int reqWidth, int reqHeight) {
	    // 源图片的宽度
	    int width = options.outWidth;
	    int height = options.outHeight;
	    int inSampleSize = 1;

	    if (width > reqWidth && height > reqHeight) {
	        // 计算出实际宽度和目标宽度的比率
	        int widthRatio = Math.round((float) width / (float) reqWidth);
	        int heightRatio = Math.round((float) width / (float) reqWidth);
	        inSampleSize = Math.max(widthRatio, heightRatio);
	    }
	    return inSampleSize;
	}

	/**
	 * 根据计算的inSampleSize，得到压缩后图片
	 *
	 * @param pathName
	 * @param reqWidth
	 * @param reqHeight
	 * @return
	 */
	private Bitmap decodeSampledBitmapFromResource(String pathName,
	                                               int reqWidth, int reqHeight) {
	    // 第一次解析将inJustDecodeBounds设置为true，来获取图片大小
	    final BitmapFactory.Options options = new BitmapFactory.Options();
	    options.inJustDecodeBounds = true;
	    BitmapFactory.decodeFile(pathName, options);
	    // 调用上面定义的方法计算inSampleSize值
	    options.inSampleSize = calculateInSampleSize(options, reqWidth,
	            reqHeight);
	    // 使用获取到的inSampleSize值再次解析图片
	    options.inJustDecodeBounds = false;
	    Bitmap bitmap = BitmapFactory.decodeFile(pathName, options);

	    return bitmap;
	}


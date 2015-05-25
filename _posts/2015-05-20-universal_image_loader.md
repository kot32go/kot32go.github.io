---
layout: post
title: Universal_Image_Loader 所有配置简介
description: "libiarys"
tags: [libiarys]
---
##Universal_Image_Loader 所有配置简介和避免OOM的配置<!--more-->

  // 内存缓存的设置选项 (最大图片宽度,最大图片高度) 默认当前屏幕分辨率
  // .memoryCacheExtraOptions(480, 800)

  // 硬盘缓存的设置选项 (最大图片宽度,最大图片高度,压缩格式,压缩质量，处理器)
  // .discCacheExtraOptions(480, 800, CompressFormat.JPEG, 75, null)

  // 设置自定义加载和显示图片的线程池
  // .taskExecutor(DefaultConfigurationFactory.createExecutor(3,Thread.NORM_PRIORITY
  // - 1, QueueProcessingType.FIFO))

  // 设置自定义加载和显示内存缓存或者硬盘缓存图片的线程池
  // .taskExecutorForCachedImages(DefaultConfigurationFactory.createExecutor(3,Thread.NORM_PRIORITY
  // - 1, QueueProcessingType.FIFO))

  // 设置显示图片线程池大小，默认为3
  // 注:如果设置了taskExecutor或者taskExecutorForCachedImages 此设置无效
  // .threadPoolSize(3)

  // 设置图片加载线程的优先级,默认为Thread.NORM_PRIORITY-1
  // 注:如果设置了taskExecutor或者taskExecutorForCachedImages 此设置无效
  // .threadPriority(Thread.NORM_PRIORITY - 1)

  // 设置图片加载和显示队列处理的类型 默认为QueueProcessingType.FIFO
  // 注:如果设置了taskExecutor或者taskExecutorForCachedImages 此设置无效
  // .tasksProcessingOrder(QueueProcessingType.FIFO)

  // 设置拒绝缓存在内存中一个图片多个大小 默认为允许,(同一个图片URL)根据不同大小的imageview保存不同大小图片
  // .denyCacheImageMultipleSizesInMemory()

  // 设置内存缓存 默认为一个当前应用可用内存的1/8大小的LruMemoryCache
  // .memoryCache(new LruMemoryCache(2 * 1024 * 1024))

  // 设置内存缓存的最大大小 默认为一个当前应用可用内存的1/8
  // .memoryCacheSize(2 * 1024 * 1024)

  // 设置内存缓存最大大小占当前应用可用内存的百分比 默认为一个当前应用可用内存的1/8
  // .memoryCacheSizePercentage(13)

  // 设置硬盘缓存
  // 默认为StorageUtils.getCacheDirectory(getApplicationContext())
  // 即/mnt/sdcard/android/data/包名/cache/
  // .discCache(new
  // UnlimitedDiscCache(StorageUtils.getCacheDirectory(getApplicationContext())))

  // 设置硬盘缓存的最大大小
  // .discCacheSize(50 * 1024 * 1024)

  // 设置硬盘缓存的文件的最多个数
  // .discCacheFileCount(100)

  // 设置硬盘缓存文件名生成规范
  // 默认为new HashCodeFileNameGenerator()
  // .discCacheFileNameGenerator(new Md5FileNameGenerator())

  // 设置图片下载器
  // 默认为 DefaultConfigurationFactory.createBitmapDisplayer()
  // .imageDownloader(
  // new HttpClientImageDownloader(getApplicationContext(),
  // new DefaultHttpClient()))

  // 设置图片解码器
  // 默认为DefaultConfigurationFactory.createImageDecoder(false)
  // .imageDecoder(DefaultConfigurationFactory.createImageDecoder(false))

  // 设置默认的图片显示选项
  // 默认为DisplayImageOptions.createSimple()
  // .defaultDisplayImageOptions(DisplayImageOptions.createSimple())

  // 打印DebugLogs
  // .writeDebugLogs()

  // 建立
  // .build();
  
  
/**
   * DisplayImageOptions所有配置简介
   */
  // 设置图片加载时的默认图片
  // .showImageOnLoading(R.drawable.ic_chat_def_pic)
  // 设置图片加载失败的默认图片
  // .showImageOnFail(R.drawable.ic_chat_def_pic_failure)
  // 设置图片URI为空时默认图片
  // .showImageForEmptyUri(R.drawable.ic_chat_def_pic)
  // 设置是否将View在加载前复位
  // .resetViewBeforeLoading(false)
  // 设置延迟部分时间才开始加载
  // 默认为0
  // .delayBeforeLoading(100)
  // 设置添加到内存缓存
  // 默认为false
  // .cacheInMemory(true)
  // 设置添加到硬盘缓存
  // 默认为false
  // .cacheOnDisc(true)
  // 设置规模类型的解码图像
  // 默认为ImageScaleType.IN_SAMPLE_POWER_OF_2
  // .imageScaleType(ImageScaleType.IN_SAMPLE_POWER_OF_2)
  // 设置位图图像解码配置
  // 默认为Bitmap.Config.ARGB_8888
  // .bitmapConfig(Bitmap.Config.ARGB_8888)
  // 设置选项的图像解码
  // .decodingOptions(new Options())
  // 设置自定义显示器
  // 默认为DefaultConfigurationFactory.createBitmapDisplayer()
  // .displayer(new FadeInBitmapDisplayer(300))
  // 设置自定义的handler
  // 默认为new Handler()
  // .handler(new Handler())
  // 建立
  // .build();
  
   * 减少配置之中线程池的大小，(.threadPoolSize).推荐1-5；
   
   * 使用.bitmapConfig(Bitmap.config.RGB_565)代替ARGB_8888;
   
   * 使用.imageScaleType(ImageScaleType.IN_SAMPLE_INT)或者        try.imageScaleType(ImageScaleType.EXACTLY)；
   
   * 避免使用RoundedBitmapDisplayer.他会创建新的ARGB_8888格式的Bitmap对象；
   
   * 使用.memoryCache(new WeakMemoryCache())，不要使用.cacheInMemory();
   
   * 一定不要重复初始化
   	
   
   
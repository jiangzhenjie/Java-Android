# 对Android使用Bitmap时出现OOM情况的一些实践与思考 #
----------------------------
## 背景 ##
 最近在做的项目中有一个相册模块需要展示大量的图片。该模块大概是这样的。进入相册后，在底部展示一个类似Gallery的画廊，然后上部展示画廊选中的图片，点击大图进入另外一个界面，该界面类似于图库浏览器，即可以左右滑动浏览图片。由于图片较大，较多，因此必须认真考虑OOM问题。经过几天的实践和摸索，最后较为满意地解决了OOM问题。实践和摸索的过程很痛苦，不过最后也有所收获，就此记下，方便日后查询。：）
## 解决OOM问题的几个有效方法 ##
Android系统对内存特别敏感，加之Bitmap对象就像一头大胖子，非常消耗内存。解决OOM问题，从根本来说，有以下几种方法，总结之：   

    1. 对图片进行压缩显示，按需分配。
    
    一般而言，在大多数情况下图片的实际大小都比需要呈现出来的要大很多。
	例如，在一个40*40的ImageView中显示一张480*480的图片。显然没有必要将480*480的图片全部加载到内存中。因此对图片进行压缩非常必要。
	在Android的官方教程中提供了对图片进行压缩的详细教程，建议认真看一下。(http://developer.android.com/training/displaying-bitmaps/index.html)
	该教程中主要使用下面的方法计算图片的压缩率。

	public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    	// Raw height and width of image
    	final int height = options.outHeight;
    	final int width = options.outWidth;
    	int inSampleSize = 1;

    	if (height > reqHeight || width > reqWidth) {

        	final int halfHeight = height / 2;
        	final int halfWidth = width / 2;

        	// Calculate the largest inSampleSize value that is a power of 2 and keeps both
        	// height and width larger than the requested height and width.
        	while ((halfHeight / inSampleSize) > reqHeight
                && (halfWidth / inSampleSize) > reqWidth) {
            	inSampleSize *= 2;
        	}
    	}
    	return inSampleSize;
	}

	2. 及时释放，保证内存循环利用。
	
	无论对图片进行怎样的压缩，当图片的数量达到一定程序时，OOM也必然出现，正所谓积少成多。因此，及时释放也是避免出现OOM的一个有效的方法。
	及时释放理论上有两种途径：
	一是程序员手工释放，比如调用Bitmao.recycle()方法。这种方式适合图片比较少，切换速度比较慢。比如只展示一个图片，展示完手工释放。
	二是使用图片缓存，使用软引用的方式。这种方式使用图片比较多，切换速度快。这种方式依赖Java的垃圾回收器，在多图片中效果比较好。
	提到图片缓存，这里不得不提一个非常著名的开源项目Android-Universal-Image-Loader（https://github.com/nostra13/Android-Universal-Image-Loader）。
	该项目提供了一个强大的图片加载工具，可以加载本地图片和网络图片，具有缓存功能，具有多线程功能等等，非常强大，建议凡涉及图片加载，优先考虑该项目。
	为避免OOM，对ImageLoader的设置有一些需要注意的地方，如下：

	对于初始化ImageLoader
		必须使用WeakMemoryCache
		限制threadPoolSize的大小
		限制memorySize的大小

	private void initImageLoader() {
        if (ImageLoader.getInstance().isInited()) {
            ImageLoader.getInstance().clearMemoryCache();
            ImageLoader.getInstance().destroy();
        }
        ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(
                this).memoryCache(new WeakMemoryCache()).threadPoolSize(1)
                .build();
        ImageLoader.getInstance().init(config);
    }

	对于显示图片
		使用Bitmap.Config.RGB_565
		使用resetViewBeforeLoading

	public static DisplayImageOptions getDisplayOptions(
            BitmapFactory.Options decodingOptions) {
        DisplayImageOptions.Builder builder = new DisplayImageOptions.Builder()
                .bitmapConfig(Bitmap.Config.RGB_565)
                .resetViewBeforeLoading(true).cacheOnDisk(false);
        if (decodingOptions != null) {
            builder = builder.decodingOptions(decodingOptions);
        }
        return builder.build();
    }

	对于加载图片
		根据具体需要计算压缩率
	public static BitmapFactory.Options getDecodingOptions(String dir,
                                                    String fileName, int reqWidth, int reqHeight) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeFile(dir + File.separator + fileName, options);
        options.inSampleSize = calculateInSampleSize(options, reqWidth,
                reqHeight);
        options.inJustDecodeBounds = false;
        return options;
    }

	总之，ImageLoader非常强大，根据具体情况，选择不同的配置。

	
	以上的两种方式是避免OOM的根本所在，很多优秀的图片开源项目都是基于上述的两种方式。
	然而在实践过程中，自己也发现了一些细小但有用的方法，如下。
	
	
	3. 延迟显示，给GC留一点时间。
	
	当用户操作很快时，比如在ViewPager中显示大图，尽管使用上述的两种方法优化，OOM也可能出现。
	原因在于GC回收内存需要一定的时间，延迟显示可以给GC留一点时间来回收内存。
	
	 new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                ImageLoader.getInstance().displayImage(uri, mPhotoView,
                        ImageUtils.getDisplayOptions(ImageUtils.getDecodingOptions(AlbumActivity.gAlbumDir, mImageUri, width, height)),
                        mImageLoaderListener);
            }
     },200);	

	4. 缺省图片改用进度条，进一步减少内存消耗。

	有一种习惯是当图片没有加载时，显示一张缺省图片。然而，在内存很吃紧的时候，这种方式也是很浪费内存的。
	比如，在图库浏览器浏览图片的时候，当快速滑动时，你会发现系统使用是使用进度条来显示加载的。改用进度条，对优化内存也有一定作用。

# 小结 #

以上就是这几天来对OOM问题的一些实践和思考。也许不完全正确，或者不够完整，但对避免OOM问题有一定参考价值。后续有更多的内容再改进。

	

 
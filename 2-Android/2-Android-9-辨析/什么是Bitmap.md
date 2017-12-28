文档处理控制栏：
* [x] 选题收集：
* [ ] 初稿整理：
* [ ] 补充校对：
* [ ] 入库存档：

---

版权信息
原文链接：[Android Bitmap最全面详解](http://www.jianshu.com/p/ade293a49cd7)

---


## 1.Bitmap类

> Bitmap图像处理的最重要类之一。用它可以获取图像文件信息，进行图像颜色变换、剪切、旋转、缩放等操作，并可以指定格式保存图像文件

#### 1.1Bitmap常用的方法

![](//upload-images.jianshu.io/upload_images/7508328-70e6027e705ce809.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](//upload-images.jianshu.io/upload_images/7508328-f42a7c5238b4ea90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.2Bitmap的颜色配置信息与压缩方式信息

Bitmap中有两个内部枚举类：Config和CompressFormat，
Config是用来设置颜色配置信息的，
CompressFormat是用来设置压缩方式的。

![](//upload-images.jianshu.io/upload_images/7508328-6f3c5d3e622fd0cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Config解析：**

*   Bitmap.Config.ALPHA_8：颜色信息只由透明度组成，占8位。

*   Bitmap.Config.ARGB_4444：颜色信息由透明度与R（Red），G（Green），B（Blue）四部分组成，每个部分都占4位，总共占16位。

*   Bitmap.Config.ARGB_8888：颜色信息由透明度与R（Red），G（Green），B（Blue）四部分组成，每个部分都占8位，总共占32位。是Bitmap默认的颜色配置信息，也是最占空间的一种配置。

*   Bitmap.Config.RGB_565：颜色信息由R（Red），G（Green），B（Blue）三部分组成，R占5位，G占6位，B占5位，总共占16位。

> 通常我们优化Bitmap时，当需要做性能优化或者防止OOM（Out Of Memory），我们通常会使用Bitmap.Config.RGB_565这个配置，因为Bitmap.Config.ALPHA_8只有透明度，显示一般图片没有意义，Bitmap.Config.ARGB_4444显示图片不清楚，Bitmap.Config.ARGB_8888占用内存最多。

**CompressFormat解析：**

*   Bitmap.CompressFormat.JPEG：表示以JPEG压缩算法进行图像压缩，压缩后的格式可以是".jpg"或者".jpeg"，是一种有损压缩。

*   Bitmap.CompressFormat.PNG：表示以PNG压缩算法进行图像压缩，压缩后的格式可以是".png"，是一种无损压缩。

*   Bitmap.CompressFormat.WEBP：表示以WebP压缩算法进行图像压缩，压缩后的格式可以是".webp"，是一种有损压缩，质量相同的情况下，WebP格式图像的体积要比JPEG格式图像小40%。美中不足的是，WebP格式图像的编码时间“比JPEG格式图像长8倍”。

#### 1.3Bitmap对图像进行操作

###### 1）Bitmap裁剪图像

`itmap.createBitmap(Bitmap source, int x, int y, int width, int height)`

根据源Bitmap对象source，创建出source对象裁剪后的图像的Bitmap。x,y分别代表裁剪时，x轴和y轴的第一个像素，width，height分别表示裁剪后的图像的宽度和高度。
注意：x+width要小于等于source的宽度，y+height要小于等于source的高度。

`Bitmap.createBitmap(Bitmap source, int x, int y, int width, int height,Matrix m, boolean filter)`

这个方法只比上面的方法多了m和filter这两个参数，m是一个Matrix（矩阵）对象，可以进行缩放，旋转，移动等动作，filter为true时表示source会被过滤，仅仅当m操作不仅包含移动操作，还包含别的操作时才适用。其实上面的方法本质上就是调用这个方法而已。

~~~
    public static Bitmap createBitmap(Bitmap source, int x, int y, int width, int height) {
        return createBitmap(source, x, y, width, height, null, false);
    }

~~~

###### 2 ）Bitmap缩放，旋转，移动图像

Bitmap缩放，旋转，移动，倾斜图像其实就是通过`Bitmap.createBitmap(Bitmap source, int x, int y, int width, int height,Matrix m, boolean filter)`方法实现的，只是在实现这些功能的同时还可以实现图像的裁剪。

~~~
// 定义矩阵对象  
        Matrix matrix = new Matrix();  
        // 缩放图像  
        matrix.postScale(0.8f, 0.9f);  
        // 向左旋转（逆时针旋转）45度，参数为正则向右旋转（顺时针旋转） 
        matrix.postRotate(-45);  
        //移动图像
        //matrix.postTranslate(100,80);
        Bitmap bitmap = Bitmap.createBitmap(source, 0, 0, source.getWidth(), source.getHeight(),  
                matrix, true);

~~~

Matrix的postScale和postRotate方法还有多带两个参数的重载方法postScale(float sx, float sy, float px, float py)和postRotate(float degrees, float px, float py)，后两个参数px和py都表示以该点为中心进行操作。
**注意：虽然Matrix还可以调用postSkew方法进行倾斜操作，但是却不可以在此时创建Bitmap时使用。**

###### 3） Bitmap保存图像与释放资源

~~~
bitmap=BitmapFactory.decodeResource(getResources(),R.drawable.feng);
        File file=new File(getFilesDir(),"lavor.jpg");
        if(file.exists()){
            file.delete();
        }
        try {
            FileOutputStream outputStream=new FileOutputStream(file);
            bitmap.compress(Bitmap.CompressFormat.JPEG,90,outputStream);
            outputStream.flush();
            outputStream.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        bitmap.recycle();//释放bitmap的资源，这是一个不可逆转的操作

~~~

###### 3）Matrix拓展类

| 方法 | 介绍 |
| --- | --- |
| setRotate(float degrees, float px, float py) | 对图片进行旋转 |
| setScale(float sx, float sy) | 对图片进行缩放 |
| setTranslate(float dx, float dy) | 对图片进行平移 |
| postTranslate(centerX, centerY) | 在上一次修改的基础上进行再次修改 set 每次操作都是最新的 会覆盖上次的操作 |

~~~
    //显示原图 
        ImageView iv_src = (ImageView) findViewById(R.id.iv_src); 

        //显示副本 
        ImageView iv_copy = (ImageView) findViewById(R.id.iv_copy);

        //[1]先把tomcat.png 图片转换成bitmap 显示到iv_src
        Bitmap srcBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.tomcat);

        //[1.1]操作图片 
//      srcBitmap.setPixel(20, 30, Color.RED);
        iv_src.setImageBitmap(srcBitmap);

        //[2]创建原图的副本    

        //[2.1]创建一个模板  相当于 创建了一个大小和原图一样的 空白的白纸 
        Bitmap copybiBitmap = Bitmap.createBitmap(srcBitmap.getWidth(), srcBitmap.getHeight(), srcBitmap.getConfig());
        //[2.2]想作画需要一个画笔 
        Paint paint = new Paint();
        //[2.3]创建一个画布  把白纸铺到画布上 
        Canvas canvas = new Canvas(copybiBitmap);
        //[2.4]开始作画  
        Matrix matrix = new Matrix();

        //[2.5]对图片进行旋转 
        //matrix.setRotate(20, srcBitmap.getWidth()/2, srcBitmap.getHeight()/2);

        //[2.5]对图片进行
//      matrix.setScale(0.5f, 0.5f);

        //[2.6]对图片进行平移
//      matrix.setTranslate(30, 0);

        //[2.7]镜面效果  如果2个方法一起用 
//      matrix.setScale(-1.0f, 1);
        //post是在上一次修改的基础上进行再次修改  set 每次操作都是最新的 会覆盖上次的操作
//      matrix.postTranslate(srcBitmap.getWidth(), 0);

        //[2,7]倒影效果
        matrix.setScale(1.0f, -1);
        //post是在上一次修改的基础上进行再次修改  set 每次操作都是最新的 会覆盖上次的操作
        matrix.postTranslate(0, srcBitmap.getHeight());
        canvas.drawBitmap(srcBitmap,matrix , paint);

        //[3]把copybimap显示到iv_copy上
        iv_copy.setImageBitmap(copybiBitmap);

~~~

## 2.BitmapFactory类

> 创建位图对象从不同的来源,包括文件、流, 和字节数组。

###### 2.1BitmapFactory常用方法

| 方法 | 说明 |
| --- | --- |
| decodeFile(String pathName, Options opts) | 从文件读取图片 |
| decodeFile(String pathName) | 从文件读取图片 |
| decodeFileDescriptor(FileDescriptor fd) | 从文件读取文件 与decodeFile不同的是这个直接调用JNI函数进行读取 效率比较高 |
| decodeFileDescriptor(FileDescriptor fd, Rect outPadding, Options opts) | 同上 |
| decodeStream(InputStream is) | 从输入流读取图片 |
| decodeStream(InputStream is, Rect outPadding, Options opts) | 从输入流读取图片 |
| decodeStream(InputStream is, Rect outPadding, Options opts) | 从资源文件读取图片 |
| decodeResource(Resources res, int id) | 从资源文件读取图片 |
| decodeResource(Resources res, int id, Options opts) | 从资源文件读取图片 |
| decodeByteArray(byte[] data, int offset, int length) | 从数组读取图片 |
| decodeByteArray(byte[] data, int offset, int length, Options opts) | 从数组读取图片 |

> BitmapFactory.decodeResource 加载的图片可能会经过缩放，该缩放目前是放在 java 层做的，效率比较低，而且需要消耗 java 层的内存。因此，如果大量使用该接口加载图片，容易导致OOM错误
> BitmapFactory.decodeStream 不会对所加载的图片进行缩放，相比之下占用内存少，效率更高。
> 这两个接口各有用处，如果对性能要求较高，则应该使用 decodeStream；如果对性能要求不高，且需要 Android 自带的图片自适应缩放功能，则可以使用 decodeResource。

#### 2.2BitmapFactory三种加载图片

*   1）从资源文件读取图片

~~~
Bitmap bitmap= BitmapFactory.decodeResource(getResources(),R.drawable.after19);
iv.setImageBitmap(bitmap);

~~~

*   2）从输入流中读取图片

~~~
FileInputStream fis=new FileInputStream(new File(getFilesDir()+"/psb.jpg"));
            Bitmap bitmap= BitmapFactory.decodeStream(fis);
            iv.setImageBitmap(bitmap);

~~~

*   3）从数组中读取图片

~~~
public static Bitmap readBitmapFromByteArray(byte[] data, int width, int height) {
      BitmapFactory.Options options = new BitmapFactory.Options();
      options.inJustDecodeBounds = true;
      BitmapFactory.decodeByteArray(data, 0, data.length, options);
      float srcWidth = options.outWidth;
      float srcHeight = options.outHeight;
      int inSampleSize = 1;

      if (srcHeight > height || srcWidth > width) {
          if (srcWidth > srcHeight) {
              inSampleSize = Math.round(srcHeight / height);
          } else {
              inSampleSize = Math.round(srcWidth / width);
          }
      }

      options.inJustDecodeBounds = false;
      options.inSampleSize = inSampleSize;

      return BitmapFactory.decodeByteArray(data, 0, data.length, options);
  }

~~~

#### 2.3BitmapFactory常用操作

*   1）保存本地图片

~~~
public static void writeBitmapToFile(String filePath, Bitmap b, int quality) {
      try {
          File desFile = new File(filePath);
          FileOutputStream fos = new FileOutputStream(desFile);
          BufferedOutputStream bos = new BufferedOutputStream(fos);
          b.compress(Bitmap.CompressFormat.JPEG, quality, bos);
          bos.flush();
          bos.close();
      } catch (IOException e) {
          e.printStackTrace();
      }
  }

~~~

*   2）图片缩放

~~~
public static Bitmap bitmapScale(Bitmap bitmap, float scale) {
      Matrix matrix = new Matrix();
      matrix.postScale(scale, scale); // 长和宽放大缩小的比例
      Bitmap resizeBmp = Bitmap.createBitmap(bitmap, 0, 0, bitmap.getWidth(), bitmap.getHeight(), matrix, true);
      return resizeBmp;
  }

~~~

*   3）对 bitmap 进行裁剪

~~~
public Bitmap  bitmapClip(Context context , int id , int x , int y){
   Bitmap map = BitmapFactory.decodeResource(context.getResources(), id);
   map = Bitmap.createBitmap(map, x, y, 120, 120);
   return map;
}

~~~

## 3.BitmapFactory.option常用方法

| Option参数类方法 | 说明 |
| --- | --- |
| public boolean inJustDecodeBounds | 如果设置为true，不获取图片，不分配内存，但会返回图片的高度宽度信息 |
| public int inSampleSize | 图片缩放的倍数 |
| public int outWidth | 获取图片的宽度值 |
| public int outHeight | 获取图片的高度值 |
| public int inDensity | 用于位图的像素压缩比 |
| public int inTargetDensity | 用于目标位图的像素压缩比（要生成的位图） |
| public byte[] inTempStorage | 创建临时文件，将图片存储 |
| public boolean inScaled | 设置为true时进行图片压缩，从inDensity到inTargetDensity |
| public boolean inDither | 如果为true,解码器尝试抖动解码 |
| public Bitmap.Config inPreferredConfig | 设置解码器这个值是设置色彩模式，默认值是ARGB_8888，在这个模式下，一个像素点占用4bytes空间，一般对透明度不做要求的话，一般采用RGB_565模式，这个模式下一个像素点占用2bytes |
| public String outMimeType | 设置解码图像 |
| public boolean inPurgeable | 当存储Pixel的内存空间在系统内存不足时是否可以被回收 |
| public boolean inInputShareable | inPurgeable为true情况下才生效，是否可以共享一个InputStream |
| public boolean inPreferQualityOverSpeed | 为true则优先保证Bitmap质量其次是解码速度 |
| public boolean inMutable | 配置Bitmap是否可以更改，比如：在Bitmap上隔几个像素加一条线段 |
| public int inScreenDensity | 当前屏幕的像素密度 |


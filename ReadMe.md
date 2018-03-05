# Darknet yolo 在 android studio上的移植和实现

## 假期无事 跑了一下darknetyolo 看起来效果不错 经过一系列不懈优化 终于跑到1秒内了

# 主要步骤

## 安装NDK版本的android studio

其实现在 android studio已经非常完善了
NDK也不需要什么特别的设置 直接用即可

## 创建一个NDK过程
这里自己搞定吧 下一步的事情

## 添加darknet对应文件
![](http://www.chenty.com/wp-content/uploads/2018/02/TIM%E5%9B%BE%E7%89%8720180214175700.png)

## 添加Cmake路径和配置
我这边参考 Makefile 添加如下

```c
#C Flag
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-unknown-pragmas -Wfatal-errors -fPIC")
set(CMAKE_LD_FLAGS "${CMAKE_LD_FLAGS} -lm -pthread -fopenmp")

#Cmakefilelist
file(GLOB darknet_files "src/main/cpp/darknet/src/*.c")

set(DARKNET_SRC_LISTS
    ${darknet_files}
    src/main/cpp/darknetlib.c
)

add_library( # Sets the name of the library.
             darknetlib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             ${DARKNET_SRC_LISTS}
             )
```

## 添加JNI接口
添加完成后需要编写中间层JNI调用完成对接
也就是 darknetlib.c这个文件

初期我做的比较简单没有传递 bitmap 先传递了 文件path 然后调用官方程序
代码如下很简单
```c
jdouble
JNICALL
Java_com_example_chenty_demoyolo_Yolo_testyolo(JNIEnv *env, jobject obj, jstring imgfile)
{
    double time;
    const char *imgfile_str = (*env)->GetStringUTFChars(env, imgfile, 0);

    char *datacfg_str = "/sdcard/yolo/cfg/voc.data";
    char *cfgfile_str = "/sdcard/yolo/cfg/tiny-yolo-voc.cfg";
    char *weightfile_str = "/sdcard/yolo/weights/tiny-yolo-voc.weights";
    //char *imgfile_str = "/sdcard/yolo/data/dog.jpg";
    char *outimgfile_str = "/sdcard/yolo/out";

    time = test_detector(datacfg_str, cfgfile_str,
                  weightfile_str, imgfile_str,
                  0.2f, 0.5f, outimgfile_str, 0);

    (*env)->ReleaseStringUTFChars(env, imgfile, imgfile_str);
    return time;
}
```

此处函数传递进来的是需要转换文件的路径 然后执行深度网络 
返回值我修改了下返回的是执行时间

## Java部分
下面对java层进行添加和修改 和普通Jni差不多
```java
public void yoloDetect(){

        new Thread(new Runnable() {
            public void run() {
                double runtime = testyolo(srcimgpath);
                Log.i(TAG, "yolo run time " + runtime);
                Message msg = new Message();
                msg.what = DETECT_FINISH;
                msg.obj = runtime;
                mHandler.sendMessage(msg);
            }
        }).start();

    }

public native double testyolo(String imgfile);
```

## 打包模型和释放
为了更加便于使用打包了模型文件 并将模型直接拷贝到SD卡

这部分是android常规
```java
public void exactresClick(View v){
        view_status.setText("exact model, please wait");
        copyFilesFassets(this, "cfg", "/sdcard/yolo/cfg");
        copyFilesFassets(this, "data", "/sdcard/yolo/data");
        copyFilesFassets(this, "weights", "/sdcard/yolo/weights");
        view_status.setText("exact model finish");

    }
```
## 执行结果还是比较满意的
在 小米6上可以跑到1秒左右一张的速度 因为没有使用neon 应该还有优化的空间

![](http://www.chenty.com/wp-content/uploads/2018/02/Screenshot_20180214-122851-576x1024.png)![](http://www.chenty.com/wp-content/uploads/2018/02/Screenshot_20180214-123105-576x1024.png)![](http://www.chenty.com/wp-content/uploads/2018/02/Screenshot_20180214-123015-576x1024.png)![](http://www.chenty.com/wp-content/uploads/2018/02/Screenshot_20180214-122952-576x1024.png)

apk下载
https://pan.baidu.com/s/1qZbIdXU



- [Android 性能优化之 String篇](http://blog.csdn.net/vfush/article/details/53038437)
- [Android 内存优化总结&实践](https://mp.weixin.qq.com/s/2MsEAR9pQfMr1Sfs7cPdWQ)
- [Android性能优化典范 - 第6季](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653578016&idx=1&sn=d997d1142bac09e3764c075392468ae5&chksm=84b3b127b3c4383197c7d1cf15ecec44d66a1119b033ae383f9e2126bb1be0abc93416622dc0&scene=21#wechat_redirect)
- [阿里巴巴的Android内存优化分享](http://www.infoq.com/cn/presentations/android-memory-optimization)
- [Android 内存暴减的秘密？！](https://mp.weixin.qq.com/s/4YS4QW1lo0LiyuApFznMVA)
- [我这样减少了26.5M Java内存！](https://mp.weixin.qq.com/s?__biz=MzA3NjA3NTI5Mg==&mid=2656330117&idx=1&sn=a304224af107ab97a6dfc8e04e153bef&chksm=84c619f6b3b190e08796ba9448d8e23e92b9977e1c12845457b27b3cb6e824d455ab7759e400&scene=21#wechat_redirect)


--------

前人栽树，后人乘凉

### 1.Android内存分配和回收方式

- [Android 操作系统的内存回收机制](https://www.ibm.com/developerworks/cn/opensource/os-cn-android-mmry-rcycl/)
- [Android进程的内存管理分析](http://blog.csdn.net/gemmem/article/details/8920039)
- [android dalvik heap 浅析](http://blog.csdn.net/cqupt_chen/article/details/11068129)
- [揭秘 ART 细节 —— Garbage collection](http://www.cnblogs.com/jinkeep/p/3818180.html)


### 2.内存优化所对应的内存问题

##### 2.1 内存泄漏

- [Android性能优化之常见的内存泄漏](http://blog.csdn.net/u010687392/article/details/49909477)
- [Android App 内存泄露之Handler](http://blog.csdn.net/zhuanglonghai/article/details/38233069)


##### 2.2 图片

- [Android 开发绕不过的坑：你的 Bitmap 究竟占多大内存？](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=403263974&idx=1&sn=b0315addbc47f3c38e65d9c633a12cd6&scene=21#wechat_redirect)
- [GlideBitmapPool](https://github.com/amitshekhariitbhu/GlideBitmapPool)

##### 2.3 缓存池

##### 2.4 内存抖动

- [Android 性能优化之String篇](http://blog.csdn.net/vfush/article/details/53038437)

##### 2.5 数据结构优化

ArrayMap、SparseArray

- [HashMap，ArrayMap，SparseArray源码分析及性能对比](http://www.jianshu.com/p/7b9a1b386265)


##### 2.6 枚举

##### 2.7 ListView复用

##### 2.8 

慎用多进程、尽量使用系统资源、减少view层级、数据相关、dex优化等


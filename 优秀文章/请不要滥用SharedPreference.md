- [请不要滥用SharedPreference](http://weishu.me/2016/10/13/sharedpreference-advices/)



-----------


>之所以说SharedPreference（下文简称sp）是一种轻量级的存储方式，是它的设计所决定的：sp在创建的时候会把整个文件全部加载进内存，如果你的sp文件比较大，那么会带来两个严重问题：

> - 第一次从sp中获取值的时候，有可能阻塞主线程，使界面卡顿、掉帧。
> - 解析sp的时候会产生大量的临时对象，导致频繁GC，引起界面卡顿。
> - 这些key和value会永远存在于内存之中，占用大量内存。
> 
>
>
>小结
>总价一下，sp是一种轻量级的存储方式，使用方便，但是也有它适用的场景。要优雅滴使用sp，要注意以下几点：
> - 不要存放大的key和value！我就不重复三遍了，会引起界面卡、频繁GC、占用内存等等，好自为之！
> - 毫不相关的配置项就不要丢在一起了！文件越大读取越慢，不知不觉就被猪队友给坑了；蓝后，放进defalut的那个简直就是愚蠢行为！
> - 读取频繁的key和不易变动的key尽量不要放在一起，影响速度。（如果整个文件很小，那么忽略吧，为了这点性能添加维护成本得不偿失）
> - 不要乱edit和apply，尽量批量修改一次提交！
> - 尽量不要存放JSON和HTML，这种场景请直接使用json！
> - 不要指望用这货进行跨进程通信！！！

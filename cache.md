# cache and buffer

[Linux下谁在消耗我们的cache](http://blog.yufeng.info/archives/688)

Linux下对文件的访问和设备的访问通常会被cache起来加快访问速度，这个是系统的默认行为。 而cache需要耗费我们的内存，虽然这个内存最后可以通过echo 3>/proc/sys/vm/drop_caches这样的命令来主动释放。但是有时候我们还是需要理解谁消耗了我们的内存。

我们来先了解下内存的使用情况:
```
[root@my031045 ~]# free
             total       used       free     shared    buffers     cached
Mem:      24676836     626568   24050268          0      30884     508312
-/+ buffers/cache:      87372   24589464
Swap:      8385760
```
有了伟大的systemtap, 我们可以用stap脚本来了解谁在消耗我们的cache了：

这个命令行用来调查谁在加数据入page_cache
```
[root@my031045 ~]# stap -e 'probe vfs.add_to_page_cache {printf("dev=%d, devname=%s, ino=%d, index=%d, nrpages=%d\n", dev, devname, ino, index, nrpages )}'
...
dev=2, devname=N/A, ino=0, index=2975, nrpages=1777
dev=2, devname=N/A, ino=0, index=3399, nrpages=2594
dev=2, devname=N/A, ino=0, index=3034, nrpages=1778
dev=2, devname=N/A, ino=0, index=3618, nrpages=2595
dev=2, devname=N/A, ino=0, index=1694, nrpages=106
dev=2, devname=N/A, ino=0, index=1703, nrpages=107
dev=2, devname=N/A, ino=0, index=1810, nrpages=210
dev=2, devname=N/A, ino=0, index=1812, nrpages=211
...
```
这时候我们拷贝个大文件：

```
[chuba@my031045 ~]$ cp huge_foo.file  bar
``` 
#这时候我们可以看到文件的内容被猛的添加到cache去：
```
...
dev=8388614, devname=sda6, ino=2399271, index=39393, nrpages=39393
dev=8388614, devname=sda6, ino=2399271, index=39394, nrpages=39394
dev=8388614, devname=sda6, ino=2399271, index=39395, nrpages=39395
dev=8388614, devname=sda6, ino=2399271, index=39396, nrpages=39396
dev=8388614, devname=sda6, ino=2399271, index=39397, nrpages=39397
dev=8388614, devname=sda6, ino=2399271, index=39398, nrpages=39398
dev=8388614, devname=sda6, ino=2399271, index=39399, nrpages=39399
dev=8388614, devname=sda6, ino=2399271, index=39400, nrpages=39400
dev=8388614, devname=sda6, ino=2399271, index=39401, nrpages=39401
dev=8388614, devname=sda6, ino=2399271, index=39402, nrpages=39402
dev=8388614, devname=sda6, ino=2399271, index=39403, nrpages=39403
dev=8388614, devname=sda6, ino=2399271, index=39404, nrpages=39404
dev=8388614, devname=sda6, ino=2399271, index=39405, nrpages=39405
dev=8388614, devname=sda6, ino=2399271, index=39406, nrpages=39406
dev=8388614, devname=sda6, ino=2399271, index=39407, nrpages=39407
dev=8388614, devname=sda6, ino=2399271, index=39408, nrpages=39408
dev=8388614, devname=sda6, ino=2399271, index=39409, nrpages=39409
dev=8388614, devname=sda6, ino=2399271, index=39410, nrpages=39410
dev=8388614, devname=sda6, ino=2399271, index=39411, nrpages=39411
...
```
此外加入我们想了解下系统的cache都谁在用呢, 那个文件用到多少页了呢？
我们有个脚本可以做到，这里非常谢谢 子团 让我使用他的代码。

```
[chuba@my031045 ~]# stap -g viewcache.stp
```
 
在另外的shell里面
```
[chuba@my031045 ~]# dmesg
...
inode: 116397109, num: 5
inode: 116397111, num: 2
inode: 116397112, num: 1
inode: 116397149, num: 2
inode: 116397152, num: 1
inode: 116397336, num: 2
inode: 116397343, num: 1
inode: 116397371, num: 4
inode: 116397372, num: 2
...
```
非常清楚的看出来每个inode占用了多少页，用工具转换下就知道哪个文件耗费了多少内存。

点击下载viewcache.stp

另外小TIPS：

从inode到文件名的转换
find / -inum your_inode

从文件名到inode的转换
stat -c “%i” your_filename
或者 ls -i your_filename

我们套用了下就马上知道那个文件占用的cache很多。

[chuba@my031045 ~]$ sudo find / -inum 2399248
/home/chuba/kernel-debuginfo-2.6.18-164.el5.x86_64.rpm
玩的开心。

参考资料:
page cache和buffer cache的区别:
这篇文章总结的最靠谱: http://blog.chinaunix.net/u/1595/showart.php?id=2209511

后记:
linux下有个这样的系统调用可以知道页面的状态:mincore – determine whether pages are resident in memory
同时有人作个脚本fincore更方便大家的使用, 点击下载fincore

后来子团告诉我还有这个工具: https://code.google.com/p/linux-ftools/

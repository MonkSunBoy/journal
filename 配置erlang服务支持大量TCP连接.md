# 单进程支持大量级TCP连接

## 问题

在维护IM系统的时候，在某一天，我们的在线人数突然疯涨，很多客户端一直出现登录不上或者掉线的现象（HTTP客户端），当查看日志的时候，发现这样的错误日志：

	2016-12-21 10:33:16.511 [error] <0.672.0> Supervisor ejabberd_receiver_sup had child undefined started with {ejabberd_receiver,start_link,undefined} at <0.8620.693> exit with reason {system_limit,[{erlang,open_port,[{spawn,"expat_erl"},[binary]],[]},{xml_stream,new,2,[{file,"src/xml_stream.erl"},{line,182}]},{ejabberd_receiver,handle_call,3,[{file,"src/ejabberd_receiver.erl"},{line,207}]},{gen_server,try_handle_call,4,[{file,"gen_server.erl"},{line,607}]},{gen_server,handle_msg,5,[{file,"gen_server.erl"},{line,639}]},{proc_lib,init_p_do_apply,3,[{file,"proc_lib.erl"},{line,237}]}]} in context child_terminated
	
	2016-12-21 10:33:16.592 [error] <0.1332.0>@ejabberd_listener:accept:326 (#Port<0.6339>) Failed TCP accept: system_limit

## 查找相关信息
我们根据erlang和system_limit关键字进行查找，发现了一下信息：
[erlang资源限制](http://erlang.org/doc/efficiency_guide/advanced.html) 
>
Open ports:	The maximum number of simultaneously open Erlang ports is often by default 16,384. This limit can be configured at startup. For more information, see the +Q command-line flag in the erl(1) manual page in ERTS.
Open files and sockets:	The maximum number of simultaneously open files and sockets depends on the maximum number of Erlang ports available, as well as on operating system-specific settings and limits.

根据以上信息，发现问题应该是系统erlang系统的port个数限制和操作系统的可打开文件数的限制引起的资源问题

然后再次查询和这两个相关的信息发现：
[gen_tcp接受链接时enfile的问题分析及解决](http://blog.yufeng.info/archives/1851) 
[gen_tcp:accept](http://erlang.org/doc/man/gen_tcp.html#accept-1) 
>
EMFILE The per-process limit of open file descriptors has been reached.
ENFILE The system limit on the total number of open files has been reached.

和
>
{error, system_limit} if all available ports in the Erlang emulator are in use

## 寻找解决办法

### 增加port限制的数量

根据[ejabberd优化](https://www.ejabberd.im/tuning) 和[ejabberd参数配置](https://docs.ejabberd.im/admin/guide/managing/#erlang-runtime-system) ，我们通过修改配置文件ejabberdctl.cfg的ERL_MAX_PORTS来增加允许的port数量

### 增加最大文件描述符的数量

通过查询解决办法，发现一下信息：
[ulimit](http://blog.yufeng.info/archives/tag/ulimit) 和[老生常谈: ulimit问题及其影响](http://blog.yufeng.info/archives/1380)

通过了解相关的文档，发现一下需要修改的地方：
1. ulimit -n xxx
2. /etc/sysctl.conf
3. /etc/security/limits.conf
4. /etc/security/limits.d/90-nproc.conf
5. /proc/sys/fs/nr_open //nr_open是单个进程可分配的最大文件数
6. /proc/sys/fs/file-max // file-max是内核可分配的最大文件数

自己对以上配置的理解：
>
5和6是整个系统的全局限制，分别限制单个进程可分配的最大文件数和内核可分配的最大文件数
3和4是在用户登录后，pam从limits.conf中设置了上限，ulimit命令只能在低于上限的范围内发挥了。
1和2就是在3和4允许的范围内动态的修改限制

### 备注
```
[root@localhost ~]# vim .bashrc 
[root@localhost ~]# vim /etc/rc.d/rc.local 
```

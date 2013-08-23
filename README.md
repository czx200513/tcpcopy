[TCPCopy](https://github.com/wangbin579/tcpcopy) - A Powerful TCP Stream Replay Tool
====================================================================================

Description
--------------------------------------
>TCPCopy is a TCP stream replay tool which could be used realtime and offline and is mainly developed by NetEase and also contributed by developers in other companies (e.g., Taobao). It has been widely used in companies in China (e.g., NetEase, Taobao, Sina, Sohu). 
>
>As for realtime replay, TCPCopy is an online TCP duplication tool which could copy the TCP live flow on the online server to a target server in realtime, and thus could bring the complexity of online environments to the target server. It is especially useful for live testing and reducing errors before a system being deployed online. For example, when you want to migrate from Apache to Nginx, TCPCopy can help you test it. While Apache is running online, TCPCopy can copy the TCP flows from Apache to Nginx. To Nginx, the TCP flows are just forwarding to it. This will not affect Apache at all except cost a little network bandwidth and CPU load.  
>
>As for offline replay, TCPCopy is a offline tcp stream replay tool.


Scenarios:
--------------------------------------
    1) Distributed stress testing
       Use tcpcopy to copy real-world data to stress test your server software. Bugs that only can be produced in high-stress situations can be found.

    2) Live testing
       Prove the new system is stable and find bugs that only occur in the real world.

    3) Regression testing

    4) Benchmark
       Do performance benchmark. For instance, you can use tcpcopy to compare the performance of Apache and Nginx.
    
    5) ...


How does TCPCopy work
--------------------------------------
    Generally speaking, TCPCopy copies packets on the online server, modifies TCP/IP headers, and sends modified packets to the target server. In this way, TCP applications on the target server will consider the packets from the online server as online requests from end-users. As most applications on the internet are based on TCP protocol, TCPCopy could be widely used for applications based on HTTP, memcached, POP3, mysql, etc. 
    
    There are two kinds of frameworks that TCPCopy could be used:

![tcpcopy](https://raw.github.com/octocat/Spoon-Knife/master/forkit.gif)


Getting TCPCopy installed:
--------------------------------------
    1) Install
       a) git clone http://github.com/wangbin579/tcpcopy
       b) cd tcpcopy
       c) sh autogen.sh
       d) ./configure
       e) make
       f) make install


       Advanced Options: 
		There are quite a few configuration options for TCPCopy which allow you to control a lot of things. 
		They are as follows. 

       	--enable-debug   	compile TCPCopy with debug support (saved in a log file)
		--enable-mysqlsgt   	run TCPCopy at mysql skip-grant-tables mode
		--enable-mysql     	run TCPCopy at mysql mode
		--enable-offline    run TCPCopy at offline mode
		--enable-pcap    	run TCPCopy at pcap mode
		--enable-nfqueue    	run the TCPCopy server (intercept) at nfqueue mode
		--enable-advanced   run TCPCopy at advanced mode (archecture 2) 
		--enable- dlinject  send packets at data link layer instead of IP layer

Traditional usage guide
--------------------------------------
    2) Run:
       a) on the target host (root privilege is required):

          using ip queue (kernel < 3.5):
            modprobe ip_queue # if not running
            iptables -I OUTPUT -p tcp --sport port -j QUEUE # if not set
            ./intercept 

          or

          using nfqueue (kernel >= 3.5):
            iptables -I OUTPUT -p tcp --sport port -j NFQUEUE # if not set
            ./intercept

       b) on the source host (root privilege is required):
          sudo ./tcpcopy -x localServerPort-targetServerIP:targetServerPort


Advanced usage guide:
--------------------------------------
    1) Chinese References:
       http://blog.csdn.net/wangbin579/article/details/8950282
       http://blog.csdn.net/wangbin579/article/details/8994601
       http://blog.csdn.net/wangbin579/article/details/10148247

    2) English References:
       TODO


Example:
--------------------------------------
    Suppose there are two online hosts, 1.2.3.25 and 1.2.3.26. And 1.2.3.161 is the target host. Port 11311 is used as local server port and port 11511 is used as remote target server port. We use tcpcopy to test if 1.2.3.161 can process 2X requests than a host can serve.

    Here we use traditional tcpcopy to perform the above test task.
    
    1) on the target host (1.2.3.161, kernel 2.6.18)
       # modprobe ip_queue 
       # iptables -I OUTPUT -p tcp --sport 11511 -j QUEUE 
       # ./intercept

    2) online host (1.2.3.25)
       # ./tcpcopy -x 11311-1.2.3.161:11511

    3) online host(1.2.3.26)
       # ./tcpcopy -x 11311-1.2.3.161:11511

    CPU load and memory usage is as follows:
       1.2.3.25:
           21158 appuser   15   0  271m 226m  756 S 24.2  0.9  16410:57 asyn_server
           9168  root      15   0 18436  12m  380 S  8.9  0.1  40:59.15 tcpcopy
       1.2.3.26:
           16708 appuser   15   0  268m 225m  756 S 25.8  0.9  17066:19 asyn_server
           11662 root      15   0 17048  10m  372 S  9.3  0.0  53:51.49 tcpcopy
       1.2.3.161:
           27954 root      15   0  284m  57m  828 S 58.6  1.4 409:18.94 asyn_server
           1476  root      15   0 14784  11m  308 S  7.7  0.3  49:36.93 intercept
    Access log analysis:
       1.2.3.25:
           $ wc -l access_1109_09.log
             7867867,  2185 reqs/sec
       1.2.3.26:
           $ wc -l access_1109_09.log
             7843259,  2178 reqs/sec
       1.2.3.161:
           $ wc -l access_1109_09.log
             15705229, 4362 reqs/sec
       request loss ratio:
           (7867867 + 7843259 - 15705229) / (7867867 + 7843259) = 0.0375%

    Clearly, the target host can process 2X of requests a source host can serve.
    How is the CPU load? Well, tcpcopy on online host 1.2.3.25 used 8.9%, host 1.2.3.26 used 9.3%, while intercept on the target host consumed about 7.7%. We can see that the CPU load is low here, and so is the memory usage.


Influential Factors
--------------------------------------

Release History
--------------------------------------
	2011.09  version 0.1, TCPCopy released
	2011.11  version 0.2, fix some bugs
	2011.12  version 0.3, support mysql copy 
	2012.04  version 0.3.5, add support for multiple copies of the source request
	2012.05  version 0.4, fix some bugs 
	2012.07  version 0.5, support large packets (>MTU)
	2012.08  version 0.6, support offline replaying from pcap files to the target server
	2012.10  version 0.6.1, support intercept at multi-threading mode
	2012.11  version 0.6.3, fix fast retransmitting problem
	2012.11  version 0.6.5, support nfqueue
	2013.03  version 0.7.0, support lvs
	2013.06  version 0.8.0, support new configure option with configure --enable-advanced for advanced 
                            users and optimize intercept
	2013.08  version 0.9.0, pcap injection is supported and GPLv2 code has been removed for mysql replay,etc.

Note:
--------------------------------------
    1) It is tested on Linux only (kernal 2.6 or above).
    2) Tcpcopy may lose packets hence lose requests.
    3) Root privilege is required.
    4) To know more about tcpcopy, refer to the documents in the docs directory.
    5) Please open a new issue on http://github.com/wangbin579/tcpcopy if you encounter some problems. 


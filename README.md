# RemoteFileManager
-接下来的改动思路：
 -1.模拟RPC的远程方法调用，即geter每次都像调用本地方法(只实现接口)一样调用sender端的方法
 -	(其中涉及到方法名、参数个数、参数类型的传递，需要重新设计协议)
 -2.sender端通过反射来实现方法调用
 -（以上二条已基本完成）
 -3.geter端和sender端的方法同步及注册
 -11.27
 -文件传输(小文件可直接传输，大文件尚无思路)
 -命令和数据缓存不再只存储一次，建立一个队列存储多个，可能遇到的问题有：
 -	1.命令顺序和数据缓存的对应问题
 
 12.6改动思路(基本完成)：
 NIO+线程池的方式
 	主线程负责接收连接和请求，并阻塞，由线程池中的线程进行业务逻辑处理
 中继器的作用：
  	中继器只负责最开始的连接
 	中继器负责维护被控端的信息，控制端和被控端首次都先连接中继器，而后被控端作为服务器，控制端作为客户端进行点对点连接(nio+线程池)，且都断开和中继器的连接，
 	一旦控制端断开以后，被控端重新连接上中继器(未完成)
 
 -
 -11.26当前还存在几个问题：
 -1、sendler端连接时，会好server端疯狂通信数次到数十次。猜测是因为获取IP处时间过长(已解决)
 -2.read方法碰到 showProgram Files 会出现异常(是因为没有加上根目录，此问题待解决)
 -
 -11.27遇到的问题
 -1.geter端除了初始命令以外，其他的命令都需要发送两次才能执行，暂不清楚原因
 -	PS:原因可能是geter端发送的命令sender端来不及处理(此时server端的sender线程正在睡眠)，所以serve端发回了no data 结果geter端再次发送同样的命令
 -	解决方法：取消server端sender线程的睡眠（已解决）
 -11.28遇到问题:
 -server端接收到数据后，将表明长度的前是个字节的数据删除了，导致数据发送给geter/sender端时数据头消失，无法知道数据长度
 -	原因：服务端在写出数据的判断数据长度时，因数据中包含0，长度判断错误
 -	解决办法：新增写出object的方法
 
 -12.6遇到问题：
 	在被控端的iterator.hasNext()方法中加入线程池处理业务逻辑，会导致空指针异常
 	原因：
 		虽然该key已经被remove，但是该方法未处理完，select()中仍然有感兴趣的事件
 	解决办法：select()方法不再只阻塞在主线程上。应区分连接线程和处理线程，分别注册事件，分别阻塞
	修改思路：
		1.启动一个Boss线程和若干个worker线程，其中Boss线程负责接收请求并绑定接收事件，而后添加一个任务，并唤醒一个worker线程，让worker线程绑定读事件

12月15日：
	测试功能和修改BUG
12月15日出现BUG：
1.client端连接中继器时，出现了两个geter线程
	原因：主线程进入geter线程后返回，没加break，导致执行了register线程
2.传输数据时，head大小无法确定，接收数据时只有等head写满以后才往body写入数据，导致head中出现奇怪字符，body中缺少字符
	解决办法：
		将head的大小置为10，多余部分用0填充。接收时大小也为10
3.break函数跳出循环，会导致中继器的主线程结束，无法接受后续请求
	解决办法：
		将break替换为continue；
4.server端回发数据时，head和body的内容都正常，但是client的channel的read时，head内容变为body内容，原因未知
	原因：head未flip,当前的position仍然为limit或者capicity,client读取数据时，就直接从body开始读取了
	解决方法：channel.write()之前先将head flip一次
	
12月18日：
	待完成事项：
		rpc中的反射
	
	
 	
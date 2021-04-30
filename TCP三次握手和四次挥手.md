# TCP三次握手和四次挥手

## 三次握手四次挥手时序图

![https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X2pwZy9qM2dmaWNpY3lPdmFzRjZwek1SVmxPbmJwTU5pYzNYcUhCUEJRSW5tZkVCZ3ZkVmQ4V0RQTG9kQ2lhQ0lIeTFpYjJtU0xDTVVHcmYzcFRyaWM2ME1KZUl6SGxiZy82NDA?x-oss-process=image/format,png](http://image.huawei.com/tiny-lts/v1/images/fa6e60dc2d010a8509632732948a8e5d_875x976.jpg@900-0-90-f.jpg)

## 状态机 

![https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9qM2dmaWNpY3lPdmFzRjZwek1SVmxPbmJwTU5pYzNYcUhCUHRCV1AzWWliSzh4NUZZUHhOeFpPNzJMa09rQnZndVA2MlhWaWN6MXo0bmFmNkxnaWFGSlN3NjVpYkEvNjQw?x-oss-process=image/format,png](http://image.huawei.com/tiny-lts/v1/images/c0f349200ecf3356fe628f092e11853b_562x801.png@900-0-90-f.png) 

## 连接目标

- TCP 进行握手初始化一个连接的目标是：==分配资源、初始化序列号==(通知 peer 对端我的初始序列号是多少)，握手过程可以简化为下面的四次交互：

  1. client 端首先发送一个 SYN 包告诉 Server 端我的初始序列号是 X；
  2. Server 端收到 SYN 包后回复给 client 一个 ACK 确认包，告诉 client 说我收到了；
  3. 接着 Server 端也需要告诉 client 端自己的初始序列号，于是 Server 也发送一个 SYN 包告诉 client 我的初始序列号是 Y；
  4. Client 收到后，回复 Server 一个 ACK 确认包说我知道了。

- 整个过程 4 次交互即可完成初始化，但是，细心的同学会发现两个问题：

  1. SERVER发送的SYN包是作为响应发起者的SYN包还是作为发起连接的SYN包？怎么区分？
     - Server 的ACK确认包和接下来的SYN包合成一个SYN ACK包一起发送的，就节省了一次交互。于是三次握手在进行最少次交互的情况下完成了Peer两端的==资源分配和初始序号==的交换

- 大部分情况下建立连接需要三次握手，也不一定都是三次，==有可能出现四次握手来建立连接的==。如下图，当 Peer 两端==同时==发起 SYN 来建立连接的时候，就出现了四次握手来建立连接(对于有些 TCP/IP 的实现，可能不支持这种同时打开的情况)。 

  ![https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9qM2dmaWNpY3lPdmFzRjZwek1SVmxPbmJwTU5pYzNYcUhCUHlWRWliNUprNHJHZlM4V1E4Q3JoUkxpYkNNRHpWREFjVWFKR3dVTVBua1JaVFZtekU0QzBTaGpnLzY0MA?x-oss-process=image/format,png](http://image.huawei.com/tiny-lts/v1/images/be0e6b58ebfee3d572ae215cf39dddac_601x199.png@900-0-90-f.png) 

## 初始化序列号

>   初始化序列号X、Y可以固定吗？

假设 ISN 固定是 1，Client 和 Server 建立好一条 TCP 连接后，Client 连续给 Server 发了 10 个包，这 10 个包不知怎么被链路上的路由器缓存了(*路由器会毫无先兆地缓存或者丢弃任何的数据包*)，这个时候碰巧 Client 挂掉了，然后 Client 用同样的端口号重新连上 Server， ISN 又从 1开始，Client 又连续给 Server 发了几个包，假设这个时候 Client 的序列号变成了 5。

接着，之前被路由器缓存的 10 个数据包全部被路由到 Server 端了，Server 给 Client 回复确认号 10，这个时候，Client 整个都不好了，这是什么情况？我的序列号才到 5，你怎么给我的确认号是 10 了，整个都乱了。

RFC793 中，建议 ISN 和一个假的时钟绑在一起，这个时钟会在每 4 微秒对 ISN 做加一操作，直到超过 2^32，又从 0 开始，这需要 4 小时才会产生 ISN 的==回绕问题==，这几乎可以保证每个新连接的 ISN 不会和旧的连接的 ISN 产生冲突。这种递增方式的 ISN，很容易让攻击者猜测到 TCP 连接的 ISN，现在的实现大多是在一个基准值的基础上进行随机的。

## 初始化连接SYN超时

> Client 发送 SYN 包给 Server 后挂了，Server 回给 Client 的 SYN-ACK 一直没收到 Client 的 ACK 确认，这个时候这个连接既没建立起来，也不能算失败。
>
> 这就需要一个超时时间让 Server 将这个连接断开，否则这个连接就会一直占用 Server 的 SYN 连接队列中的一个位置，大量这样的连接就会将 Server 的 SYN 连接队列耗尽，让正常的连接无法得到处理。

目前，Linux 下默认会进行 5 次重发 SYN-ACK 包，重试的间隔时间从 1s 开始，下次的重试间隔时间是前一次的双倍，5 次的重试时间间隔为 1s,2s, 4s, 8s,16s，总共 31s，第 5 次发出后还要等 32s 都知道第 5 次也超时了，所以，总共需要 1s + 2s +4s+ 8s+ 16s + 32s =63s，TCP 才会把断开这个连接。

由于，SYN 超时需要 63 秒，那么就给攻击者一个攻击服务器的机会，攻击者在短时间内发送大量的 SYN 包给 Server(俗称 SYN flood 攻击)，用于耗尽 Server 的 SYN 队列。对于应对 SYN 过多的问题，linux 提供了几个 TCP 参数：tcp_syncookies、tcp_synack_retries、tcp_max_syn_backlog、tcp_abort_on_overflow 来调整应对。

## 断开连接目标

- TCP 进行断开连接的目标是：==回收资源、终止数据传输==。

- 由于 TCP 是全双工的，需要 Peer 两端分别各自拆除自己通向 Peer 对端的方向的通信信道。这样需要四次挥手来分别拆除通信信道，就比较清晰明了了。

  1. Client 发送一个 FIN 包来告诉 Server 我已经没数据需要发给 Server 了；
  2. Server 收到后回复一个 ACK 确认包说我知道了；
  3. 然后 server 在自己也没数据发送给 client 后，Server 也发送一个 FIN 包给 Client 告诉 Client 我也已经没数据发给 client 了；
  4. Client 收到后，就会回复一个 ACK 确认包说我知道了。

- 到此，四次挥手，这个 TCP 连接就可以完全拆除了。在四次挥手的过程中，细心的同学可能会有以下疑问：

   1. Clint 和Server同时发起断开连接的FIN包会怎么样的？TCP状态是怎么转移的？

      ```python
      如果 Peer 在 FIN_WAIT1 状态下首先收到对端 Peer 的 FIN 包的话，那么该 Peer 在确认已经收到了对端 Peer 全部的 Data 数据包后，就响应一个 ACK 给对端 Peer，然后自己进入 CLOSEING 状态，Peer 在 CLOSEING 状态下收到自己的 FIN 包的 ACK 包的话，那么就进入 TIME WAIT 状态。于是，TCP 的 Peer 两端同时发起 FIN 包进行断开连接，那么两端 Peer 可能出现完全一样的状态转移 FIN_WAIT1——>CLOSEING——->TIME_WAIT，也就会 Client 和 Server 最后同时进入 TIME_WAIT 状态。同时关闭连接的状态转移如下图所示：
      ```

      ![https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9qM2dmaWNpY3lPdmFzRjZwek1SVmxPbmJwTU5pYzNYcUhCUFFRY2lhV2dMOHpBb05JVlQxZDJLQTBpYndVV0tJZkdpYzZrRGZwcWE5TVZYS1Y0a2VoNzY1cGZuQS82NDA?x-oss-process=image/format,png](http://image.huawei.com/tiny-lts/v1/images/08cbd992f33ba7327b1b76e36d1fca0e_666x526.png@900-0-90-f.png) 

  2. Server端的ACK确认包能不能和接下来的FIN包合并成一个包？变成三次挥手呢？

     ```python
     可能的。TCP 是全双工通信，Cliet 在自己已经不会在有新的数据要发送给 Server 后，可以发送 FIN 信号告知 Server，这边已经终止 Client 到对端 Server 那边的数据传输。
     但是，这个时候对端 Server 可以继续往 Client 这边发送数据包。
     于是，两端数据传输的终止在时序上是独立并且可能会相隔比较长的时间，这个时候就必须最少需要 2+2= 4 次挥手来完全终止这个连接。
     但是，如果 Server 在收到 Client 的 FIN 包后，再也没数据需要发送给 Client 了，那么对 Client 的 ACK 包和 Server 自己的 FIN 包就可以合并成为一个包发送过去，这样四次挥手就可以变成三次了(似乎 linux 协议栈就是这样实现的)
     ```

  3. 为什么要TIME_WAIT呢 ? 能不能省？为什么？

     > 见TIME_WAIT篇

## TIME_WAIT 会带来哪些问题？

- 作为服务器，短时间内关闭了大量的 Client 连接，就会造成服务器上出现大量的 TIME_WAIT 连接，占据大量的 tuple，严重消耗着服务器的资源。
- 作为客户端，短时间内大量的短连接，会大量消耗的 Client 机器的端口，毕竟端口只有 65535 个，端口被耗尽了，后续就无法在发起新的连接了。

由于上面两个问题，作为客户端需要连本机的一个服务的时候，首选==UNIX 域套接字==而不是 TCP。怎么来解决或缓解 TIME_WAIT 问题呢？可以进行 TIME_WAIT 的快速回收和重用来缓解 TIME_WAIT 的问题。有没一些清掉 TIME_WAIT 的技巧呢？

## TIME_WAIT的快速回收和重用

### **TIME_WAIT** 快速回收 	

linux 下开启 TIME_WAIT 快速回收需要同时打开==tcp_tw_recycle==和 ==tcp_timestamps==(默认打开)两选项。Linux 下快速回收的时间为 3.5* RTO（Retransmission Timeout），而一个 RTO 时间为 200ms -120s。开启快速回收 TIME_WAIT，可能会带来(问题一)中说的三点危险，为了避免这些危险，要求同时满足以下三种情况的新连接要被拒绝掉：

- 来自同一个对端 Peer 的 TCP 包携带了时间戳；
- 之前同一台 peer 机器(仅仅识别 IP 地址，因为连接被快速释放了，没了端口信息)的某个 TCP 数据在 MSL 秒之内到过本 Server；
- Peer 机器新连接的时间戳小于 peer 机器上次 TCP 到来时的时间戳，且差值大于重放窗口戳(TCP_PAWS_WINDOW)。

初看起来正常的数据包同时满足下面 3 条几乎不可能，因为机器的时间戳不可能倒流的，出现上述的 3 点均满足时，一定是老的重复数据包又回来了，丢弃老的 SYN 包是正常的。到此，似乎启用快速回收就能很大程度缓解 TIME_WAIT 带来的问题。但是，这里忽略了一个东西就是 NAT。

在一个 NAT 后面的所有 Peer 机器在 Server 看来都是一个机器，NAT 后面的那么多 Peer 机器的==系统时间戳很可能不一致==，有些快，有些慢。这样，在 Server 关闭了与系统时间戳快的 Client 的连接后，在这个连接进入快速回收的时候，同一 NAT 后面的系统时间戳慢的 Client 向 Server 发起连接，这就很有可能同时满足上面的三种情况，造成该连接被 Server 拒绝掉。所以，在是否开启 tcp_tw_recycle 需要慎重考虑了

### **TIME_WAIT** 重用

linux 上比较完美的实现了 TIME_WAIT 重用问题。只要满足下面两点中的一点，一个 TW 状态的四元组(即一个 socket 连接)可以重新被新到来的 SYN 连接使用。

1. 新连接 SYN 告知的初始序列号比 TIME_WAIT 老连接的末序列号大；
2. 如果开启了 tcp_timestamps，并且新到来的连接的时间戳比老连接的时间戳大。

要同时开启 tcp_tw_reuse 选项和 tcp_timestamps 选项才可以开启 TIME_WAIT 重用，还有一个条件是：重用 TIME_WAIT 的条件是收到最后一个包后超过 1s。细心的同学可能发现 TIME_WAIT 重用对 Server 端来说并没解决大量 TIME_WAIT 造成的资源消耗的问题，因为不管 TIME_WAIT 连接是否被重用，它依旧占用着系统资源。即便如此，TIME_WAIT 重用还是有些用处的，它解决了整机范围拒绝接入的问题，虽然一般一个单独的 Client 是不可能在 MSL 内用同一个端口连接同一个服务的，但是如果 Client 做了 bind 端口那就是同个端口了。时间戳重用 TIME_WAIT 连接的机制的前提是 IP 地址唯一性，得出新请求发起自同一台机器，但是如果是 NAT 环境下就不能这样保证了，于是在 NAT 环境下，TIME_WAIT 重用还是有风险的。

有些同学可能会混淆 tcp_tw_reuse 和 SO_REUSEADDR 选项，认为是相关的一个东西，其实他们是两个完全不同的东西，可以说两个半毛钱关系都没。tcp_tw_reuse 是==内核选项==，而 SO_REUSEADDR ==用户态的选项==，使用 SO_REUSEADDR 是告诉内核，如果端口忙，但 TCP 状态位于 TIME_WAIT，可以重用端口。如果端口忙，而 TCP 状态位于其他状态，重用端口时依旧得到一个错误信息，指明 Address already in use”。如果你的服务程序停止后想立即重启，而新套接字依旧使用同一端口，此时 SO_REUSEADDR 选项非常有用。但是，使用这个选项就会有(问题二、)中说的三点危险，虽然发生的概率不大。

## TIME_WAIT清除技巧

1. 修改==tcp_max_tw_buckets==

   tcp_max_tw_buckets 控制并发的 TIME_WAIT 的数量，默认值是 180000。如果超过默认值，内核会把多的 TIME_WAIT 连接清掉，然后在日志里打一个警告。官网文档说这个选项只是为了阻止一些简单的 DoS 攻击，平常不要人为的降低它。 

2. 利用==RST包==从外部清掉TIME_WAIT链接

   根据 TCP 规范，收到任何的发送到未侦听端口、已经关闭的连接的数据包、连接处于任何非同步状态（LISTEN,SYS-SENT,SYN-RECEIVED）并且收到的包的 ACK 在窗口外，或者安全层不匹配，都要回执以 RST 响应(而收到滑动窗口外的序列号的数据包，都要丢弃这个数据包，并回复一个 ACK 包)，内核收到 RST 将会产生一个错误并终止该连接。我们可以利用 RST 包来终止掉处于 TIME_WAIT 状态的连接，其实这就是所谓的 ==RST 攻击==了。

   为了描述方便：假设 Client 和 Server 有个连接 Connect1，Server 主动关闭连接并进入了 TIME_WAIT 状态，我们来描述一下怎么从外部使得 Server 的处于 TIME_WAIT 状态的连接 Connect1 提前终止掉。要实现这个 RST 攻击，首先我们要知道 Client 在 Connect1 中的端口 port1(一般这个端口是随机的，比较难猜到，这也是 RST 攻击较难的一个点)，利用 IP_TRANSPARENT 这个 socket 选项，它可以 bind 不属于本地的地址，因此可以从任意机器绑定 Client 地址以及端口 port1，然后向 Server 发起一个连接，Server 收到了窗口外的包于是响应一个 ACK，这个 ACK 包会路由到 Client 处。

   这个时候 99%的可能 Client 已经释放连接 connect1 了，这个时候 Client 收到这个 ACK 包，会发送一个 RST 包，server 收到 RST 包然后就释放连接 connect1 提前终止 TIME_WAIT 状态了。提前终止 TIME_WAIT 状态是可能会带来(问题二)中说的三点危害，具体的危害情况可以看下 RFC1337。RFC1337 中建议，不要用 RST 过早的结束 TIME_WAIT 状态。

   至此，上面的疑症都解析完毕，然而细心的同学会有下面的疑问：    







  

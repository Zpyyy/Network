# TCP概述



> TCP作为==面向连接==的提供==全双工可靠==的==基于字节流==的运输层服务协议，具有==差错控制==、==拥塞控制==和==流量控制==等功能。

## TCP报文格式

![（图1）](https://img-blog.csdnimg.cn/20181113003420540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyNTY4MTY=,size_16,color_FFFFFF,t_70) 

- 滑动窗口大小（Window）

  该字段长度位 16 位，即 TCP 数据包长度位 64KB。可以通过 **Options**字段的 **WSOPT** 选项扩展到 1GB。

- 选项和填充

  该字段最大长度为40字节，一般数据结构为TLV格式。填充是为了使TCP首部为4字节（32bit）的整数倍。 

  | KIND (1字节) | LENGTH(1字节) | INFO(N字节) |
  | :----------: | :-----------: | :---------: |

  

  - kind说明选项的类型。有的TCP选项没有后面两个字段，仅包含1字节的kind字段。

  - length（如果有的话）指定该选项的总长度，该长度包括kind字段和length字段占据的2字节

  - info（如果有的话）是选项的具体信息。常见的TCP选项有7种

    

  ![img](https://img-blog.csdnimg.cn/20190416110630622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hvbGxha2U=,size_16,color_FFFFFF,t_70) 

  

  1. EOL和NOP Option（Kind 0、Kind 1）只占1 Byte，没有Length和Value字段；

  2. NOP用于 将TCP Header的长度补齐至32bit的倍数（由于Header Length字段以32bit为单位，因此TCP Header的长度一定是32bit的倍数）；

  3. SACK-Premitted Option占2 Byte，没有Value字段；

  4. 其余Option都以1 Byte的“Kind”开头，指明Option的类型；Length指明Option的总长度（包括Kind和Length）

  5. 对于收到“不能理解”的Option，TCP会无视掉，并不影响该TCP Segment的其它内容；

  6. Maximum Segment Size (MSS) Option

    一般情况下，通信双方在建立连接时，SYN Segment中会携带MSS Option，MSS指明本端可以接受的最大长度的TCP Segment（Payload，不含TCP Header），也就是说，对端发送数据的长度不应该大于MSS（单位Byte）。

    ```js
    1. 首先要明确一点，MSS并非和对端协商的值，而是对对端发送数据长度的“限制”，表明在整个TCP连接期间，都不会接收长度大于MSS的TCP Segment。
    2. 如果收到的SYN中没有MSS，将使用默认值536。MSS Option的Value字段长度固定为16bit，所以MSS最大值为65535（单位Byte）。因此，网络中所有设备都被要求，必须能够处理大小小于576Byte的数据包（IP Header + TCP Header + Default MSS 最小值为 576 Byte）
    3. IPv4网络中，MSS的典型取值为1460 ，1460Byte + 20Byte IP Header + 20Byte TCP Header = 1500Byte = 以太网典型MTU；
    4. IPv6网络中，典型MSS取值为1440；另外，如果MSS=65535，表示MSS = PMTU - 60
    ```

  7. Selective Acknowledgment (SACK) Options 

     在标准的TCP实现中，使用的是累加式的ACK，例如“ACK Num = n”代表对序列号n以前的Bytes进行确认。但是，显然，这样将无法对不连续的Segment进行确认。此外，当出现不连续Segment时，还会导致TCP的接收队列出现一个“坑”，不将这个坑填上，坑后的数据就无法交付给应用程序。

    为解决上述问题，TCP定义了SACK Option，可以使TCP接收者将这个“坑”的位置通告给发送者，让其对这一段数据进行重传。

    ==注意==：若要使用SACK特性，必须在建立连接时，在SYN Segment中附加上SACK-Permitted Option，以此告知对方自己支持SACK。  

    SACK格式

  | KIND | LENGTH | Left edge of 1st block | Right edge of 1st block | ...  | Left edge of n-st block(n<=4) | Right edge of n-st block(n<=4) |
  | :--: | :----: | :--------------------: | :---------------------: | :--: | :---------------------------: | :----------------------------: |
  |  5   |  可变  |         32bit          |          32bit          | ...  |             32bit             |             32bit              |

  - *SACK Option通过“Left Edge ~ Right Edge”，指定了一个或多个范围的Seq Num，称为SACK Block，指明了处于“坑”后面（或坑之间）的、已成功接收的Bytes。* 

  - *由于TCP Header最长为60 Byte，因此SACK Option中最多只能包含4个SACK Block* 

    

  **实例**：（终端A - 终端B）

  1.    终端A收到了TCP数据流中的Seq Num为0 ~ 1452、2905~4096的字节，但缺少了1453-2904；

  2.    终端A向B发送ACK Segment，其中ACK Num=1453、SACK Option=2905~4097，表明它已经收到了数据流中的Seq Num为2905 ~ 4096的字节，但没有还没收到1453~2904；

  3.    终端B收到这个SACK后，重传包含Byte 1453~2904的TCP Segment；

  4.    终端A向B发送ACK Segment，其中ACK Num=4097，表明它已经收到Seq Num 4097之前的所有字节

  5.    之后，数据通信恢复正常。
    

  SACK 的工作原理如下图所示， 接收方收到 500-699 的数据包，但没有收到 300-499 的数据包就会回 SACK(500-700) 给发送端，表示收到 500-699 的数据。

  ![https://p1-tt.byteimg.com/large/pgc-image/352de140635649e08a5c4cc019c9db53](http://image.huawei.com/tiny-lts/v1/images/a54cf26ddb9dfbe75f56_640x360.jpg@900-0-90-f.jpg)

8. Window Scale (WSCALE or WSopt) Option 

   TCP Header的Window Size字段长度为16bit，因而正常情况下，Window Advertisement最大只能是65535 Bytes；

    Window Scale Option用于将TCP Header的Window Size字段从16bit扩展至最多30bit，格式如下

   | KIND (3) | LENGTH(3) | shift.cnt(0-14) |
   | :------: | :-------: | :-------------: |

   1. shift.cnt的取值范围为0~14，表示将Window Advertisement的值扩展至“WindowSize × 2s

   2.    WSopt只能出现在SYN Segment或SYN+ACK Segment中，因此shift.cnt在三次握手之后就会固定下来。

   3.    另外，WSopt是双向独立的，因此连接的两个方向可以有不同的Shift.cnt。但是，WSopt必须双向同时启用，也就是说，如果SYN中不带有WSopt，SYN+ACK中也不能出现WSopt；同样，如果SYN+ACK中不带有WSopt，那么发起SYN的一端就会当作自己也不曾发送过WSopt。

   4.    shift.cnt根据接收Buffer的大小，由TCP自动选取。接收Buffer由系统或程序设定。

9. Timestamps Option and PAWS（Protection against Wrapped Sequence Numbers，防止序列号回绕） 

   启用Timestamp Option后，每个TCP Segment中都会带有Timestamp Option，其中包含了两个32bit的Timestamp（TSval和TSecr） 

   | KIND(8) | LENGTH(10) | TimeStamp Value(TSval) | TimeStamp Echo Reply(TSecr) |
   | ------- | ---------- | ---------------------- | --------------------------- |

   

   1. TSval指明了发送端在发送TCP Segment时的Timestamp；接收端在对该TCP Segment做ACK时，将TSval值回显在TSecr字段中。

      - 注意：由于TCP连接是双向的，接收端在ACK中回显TSecr时，也会把自己当前的Timestamp放入TSval字段。

   2. Timestamp是一个随时间单调递增的值，由于TCP接收端只需要在ACK中将TSval简单地回显，因此通信双方并不需要进行时间同步等操作。

   3. 通过Timestamp Option，发送端再也不需要在内存中保存发送Segment的时间了，只需要将其放入TSval，然后接收端将其回显在ACK Segment即可。当发送端收到ACK Segment后，取出TSscr，和当前时间做算术差，即可完成一次RTT的测量。

   4. 若非通过Timestamp Option来计算RTT，大部分TCP实现只会以“每个Window采样一次”的频率来测算RTT。因此通过Timestamp Option，可以实现更密集的RTT采样，使RTT的测算更精确。

   5. Timestamp Option还有PAWS（Protection Against Wrapped Sequence Numbers，防止序列号回绕）功能，详见以下示例 

      **示例：**

      假设TCP Window Size为1GB（使用Window Scale），发送者每发送一个Window的数据Timestamp值加100，数据的发送情况如下所示： 

      | 时间点 | 发送数据量 | Seq Num | TimeStamp | 接收                      |
      | ------ | ---------- | ------- | --------- | ------------------------- |
      | 1      | 0G：1G     | 0G：1G  | 0-100     | OK                        |
      | 2      | 1G:2G      | 1G:2G   | 100-200   | 其中某些Segment丢包后重传 |
      | 3      | 2G:3G      | 2G:3G   | 200-300   | OK                        |
      | 4      | 3G:4G      | 3G:4G   | 300-400   | OK                        |
      | 5      | 4G:5G      | 0G:1G   | 400-500   | OK                        |
      | 6      | 5G:6G      | 1G:2G   | 500-600   | 此前丢包的Segment出现了   |

      

      2.    在时间点2的时候，发生了丢包；在时间点4和5之间，序列号发生了回绕；在时间点6，已经被认为“丢包”的Segment延迟到达了。
      3.    由于延时到达的Segment的timestamp为1xx，小于时间点6的有效timestamp（5xx），因此这个Segment会根据PAWS机制丢弃，从而不会对TCP造成影响。、

10. User Timeout (UTO) Option 

   - UserTimeout值表明了TCP发送者等待ACK的时间，如果在指定时间内没收到ACK，就会认为对端挂掉。对于传统TCP（RFC 793）而言，UserTimeout是本地配置的。RFC 1122建议，当TCP重传3次后，应该通知应用程序，100s后，应该拆除连接。

     通过UTO，可以让TCP将UserTimeout值“告知”给对端，UTO格式如下所示：

     | KIND(28) | LENGTH(4) | G bit ( Granularity bit ) | UserTimeout |
     | -------- | --------- | ------------------------- | ----------- |

     

     1. G bit = 1，表示UserTimeout的单位为分钟；G bit = 0，表示UserTimeout的单位为秒

     2. 通过UTO，TCP接收者可以根据“对端的UserTimeout”来调整自己的行为。UserTimeout建议取值为：min（U_Limit，max（Adv_UTO，Remot_UTO，L_Limit））

        o U_Limit是本地UserTimeout的最高限制；Adv_UTO是通告出去的UserTimeout；Remot_UTO是对端的UserTimeout；L_Limit是本地UserTimeout的最低限制；

     3. 要注意的是，UTO只是用于“告知”，TCP接收者却不一定要根据对端的UTO值来调整自己的行为。

     4. 此外，NAT设备也可以根据UTO来调整连接保活计时器

   11. Authentication Option （TCP-AO）and TCP MD5 Signature Option（TCP-MD5） 

       TCP-MD5和TCP-AO主要用于防止TCP欺骗攻击（TCP Spoofing Attacks）。TCP-MD5是旧标准（RFC 2385），例如BGP、LDP等协议就是以TCP-MD5作为认证手段的。2010年后，IETF建议使用TCP-AO去取代TCP-MD5，然而TCP-AO当前的普及率还很低。

       - TCP-MD5 Option的MD5 Hash根据以下信息计算：
         1. TCP伪头部
         2. TCP头部（包括Option，checksum设为0）
         3. TCP Segment Data
         4. 密钥
       - 相对于TCP-MD5，TCP-AO的主要改进之处在于：
         1. 支持多种MAC算法
         2. 支持带内的密钥变更操作

       <u>注意：TCP-AO与TCP-MD5一样，都不包含密钥分发机制。因此在密钥分发方面都存在一定风险。</u>

   ------

   

## 可靠性

1. 三次握手
   - 三次握手过程？状态？
   - 为什么要三次握手？改成两次行不行？
   - 三次握手携带数据吗？为什么？
   - 第三次握手失败怎么办？
   - RST报文什么时候发送？
   - <u>SYN攻击是什么？如何检测？如何防范？</u>
   - 握手过程会同步什么信息？
   - 如果已经建立了连接，但是客户端突然出现故障了怎么办？ 
2. 四次挥手
   - 四次挥手过程？状态？
   - 为什么建立连接是三次握手，而关闭是四次挥手？
   - 为什么不能把服务器发送的ACK和FIN合并起来编程三次挥手？（CLOSE_WAIT状态意义是什么？）
   - 如果第二次挥手时服务器的ACK没有送达客户端，会怎样？
   - ==第四次挥手释放连接时，为什么要等待2MSL==？
   - 过多的TIME_WAIT状态对业务有什么影响？<u>如何避免？</u>
3. 差错控制
   - 差错控制包括哪些机制？
   - TCP通过什么来完成差错控制？
4. 超时重传
   - TCP非超时的情况下会重传吗？
   - 
5. 流量控制
6. 拥塞控制

## 三次握手



## 流量控制

> 流量控制是避免【发送方】的数据填满【接收方】的缓存

1. 为什么需要流量控制？
2. 如何控制？
3. 发送方何时再继续发送数据？
4. 窗口关闭是什么？怎么解决？
5. 糊涂窗口综合症是什么？怎么解决？

## 拥塞控制

> 避免【发送方】的数据填满整个网络

1. 已经有流量控制为什么还需要拥塞控制？两者有什么区别？
2. 什么是拥塞窗口？和发送窗口有什么关系？
3. 如何判断当前网络是否发生了拥塞？
4. 慢启动
5. 拥塞避免
6. 拥塞发生
7. 快速重传和快速恢复




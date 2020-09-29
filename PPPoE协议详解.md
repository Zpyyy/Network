# PPPoE协议
## 概述
  1998年PPPoE问世，它主要是解决modem存在的问题而产生的：既要通过同一个用户前置接入设备连接远程的多个用户主机，又要提供类似拨号一样的接入控制，计费等功能，而且要尽可能地减少用户的配置操作。
  它利用以太网将大量主机组成网络，通过一个远端接入设备连入因特网，并对接入的每个主机实现控制、计费功能。
  PPPoE协议采用Client / Server方式，它将PPP报文封装在以太网帧之内，在以太网上提供点对点的连接。
  根据PPP会话的起止点所在位置的不同，分为两种部署方式：
  * 第一种方式在路由器间建立PPP会话，同一共网络上的主机通过同一个PPP会话传送数据，用户PC不用装任何软件，一般是一个企业、公司用一个账号。
  * 第二种部署方式，PPP会话建立在PC和运营商的路由器间，对每一个PC建立一个PPP会话。

## PPPoE的消息交互过程和报文格式
###  PPPoE的消息交换过程
PPPoE有两个阶段：Discovery阶段和PPP Session阶段，具体如下：
1. Discovery阶段
    * 当一个主机想开始PPPoE进程的时候，它必须先识别接入端的以太网MAC地址，建立PPPoE的SESSION ID。这就是Discovery阶段的目的。  
    * 为了提供以太网上的点到点连接，每一个PPP会话必须知道远程通信对方的以太网地址，并建立一个唯一的会话标识符。PPPoE包含一个（以太网地址）发现协议来提供这个功能。
    * discovery阶段分为四步，当一台主机希望初始化一个PPPoE会话时，它必须通过Discovery阶段识别对端的MAC、得到Session-ID。
    * 先发送PADI，多个接入服务器发回PADO报文回复，主机选择一个接入服务器，发出单播PADR报文，接入服务器发出确认报文PADS，至此双方得到session ID和对方的MAC地址。
2. PPP Session阶段
    * 当PPPoE进入Session阶段后PPP报文就可以作为PPPoE帧的净荷封装在以太网帧发到对端，SESSION ID必须是Discovery阶段确定的ID，MAC地址必须是对端的MAC地址，PPP报文从Protocol ID开始。
    * 在Session阶段，主机或服务器任何一方都可发PADT（PPPoE Active Discovery Terminate）报文通知对方结束本Session。
	
### PPPoE报文格式
PPPOE的以太帧格式报文如下：
* DESTINATION_ADDR域，PADI报文目的MAC为广播MAC 0xffffffff,PADO、PADR、PADS中目的MAC是单播MAC，在PPP session阶段和报文转发时，目的MAC必须是Discovery阶段得到的单播MAC。 
* SOURCE_ADDR必须是设备发送报文接口的MAC地址。
* ETHER_TYPE 在Discovery 阶段是0x8863，在PPP  session阶段是0x8864，数据报文传送也是0x8864。
* Payload报文格式: 
    * VER 4 bit,必须是0x1 
    * TYPE 4 bit,必须是0x1
    * CODE 8 bit，在Discovery和PPP Session阶段有不同的意义
    * SESSION_ID 16 bit 在Discovery阶段产生后就确定了，0xffff为保留值，预留将来使用，目前不能被使用。   
    * LENGTH 16 bit，表示PPPoE payload的长度，它不包括以太头和pppoe头长度。
    * PPPoE payload 容纳了0个或多个TLV,一个TLV的结构如下
       * TAG_TYPE 16 bit ，其值见附录A，当收到的PADI报文中有未知TAG_TYPE必须被忽略。为引入新的TAG提供向后兼容性。
       * TAG_LENGTH 16 bit  表示TAG_VALUE.的长度，当新的强制TAG引入，版本号会增加。
       * TAG_VALUE 

### PPPoE Discovery 阶段
Discovery阶段由4个步骤组成。完成之后通信双方都知道了PPPoE SESSION_ID以及对方以太网地址，它们共同定义了唯一的PPPoE会话。
  1. 主机广播一个（会话）发起数据包（以请求建立链路）
  2. 一个或多个访问集中器发送提供（服务）数据包
  3. 主机发送单播会话请求数据包
  4. 选中的访问集中器发送确认数据包
  * 当主机接收到该确认数据包后，它就可以进入PPP会话阶段。访问集中器发送确认数据包后，它就可以进入到PPP会话阶段。
  * Discovery阶段所有的以太网帧的ETHER_TYPE域都设置为0x8863。
  
####  1.PADI报文（PPPoE Active Discovery Initiation）
  * 主机发送DESTINATION_ADDR 为广播地址的PADI数据包，CODE域设置为0x09,SESSION_ID域必须设置为0x0000。
  * PADI数据包必须包含且仅包含一个TAG_TYPE为Service-Name的TAG，以表明主机请求的服务，以及任意数目的其它类型的TAG。
      * 如：0x0101, 表示业务名，TAG_LENGTH为0时，表示任何业务都是可以接受的。
      * 如：0x0103, Host-Uniq
       * 主机如何知道一个PADO或PADS是对自己发出的PADI 或 PADR的应答？通过Host-Uniq唯一识别。TAG_VALUE 是主机任选的一个二进制数值。
       * 在PADI 或PADR中，主机可以包含一个Host-Uniq TAG 。如果接入服务器收到此TAG，它必须不加修改地包含此TAG在相应的PADO 或PADS 响应中。
       * 首先主机是知道自己发送的Host-Uniq值的，接入服务器应答时会在应答报文中不加修改地再携带此TAG。

#### 2.PADO报文（PPPoE Active Discovery Offer）
   * 当接入服务器收到一个PADI并且自己能提供服务，它就发出PADO报文来应答。DESTINATION_ADDR为发送PADI的主机的单播地址，CODE 域为0x07 并且SESSION_ID必须是0x0000。
   * PADO必须包含一个AC-Name（access Concentrator） TLV，其中容纳了接入服务器的名字（意义等同于PADI中的Service-Name）和接入服务器能够提供的其它服务的名字。
      * 如：0x0102：AC-Name
        * 唯一标示了一个特定的接入服务器区别于其他的接入服务器，它可以是商标，模块和一系列的ID信息。
   * 如果接入服务器不能提供PADI中的服务，则不发送PADO报文。
   
#### 3.PADR报文 (PPPoE Active Discovery Request)
  * 由于PADI是广播的，主机可能收到多个PADO，主机根据AC-Name或接入服务器能够提供的Services选择一个接入服务器，主机向选中的接入服务器发送PADR。 
  * 其中，DESTINATION_ADDR域设置为发送PADO的访问集中器的单播地址，CODE域设置为0x19，SESSION_ID必须设置为0x0000。
  * 它必须包含一个TAG_TYPE为Service-Name的TLV表明主机请求的服务。还可以包含任意个其它TLV。

#### 4.PADS报文 (PPPoE Active Discovery Session-confirmation)
  * 接入服务器为PPPOE 会话生成唯一的SESSION_ID 并且会应以一个PADS给主机。
  * DESTINATION_ADDR域为发送PADR数据包的主机的单播以太网地址，CODE域设置为0x65,SESSION_ID必须设置为所创建好的PPPoE会话标识符。
  * PADS包含一个TAG_TYPE为Service-Name的TLV。表明接入服务器接受的服务。还可以包含任意个其它TLV。
  * 当接入服务器不感兴趣PADR中的Service-Name，它必须应答一个TAG_TYPE  为Service-Name-Error的PADS报文。同时SESSION_ID必须为0x0000。
    * 该TAG(典型的有一个长度为零的数据部分)表明了由于某种原因，没有理睬所请求的Service-Name。如果有数据部分,并且数据部分的头一个字节非0，那么它必须是一个可打印的UTF-8字符串，解释请求被拒绝的原因。
    * 该字符串可以不以NULL结束。

#### 5.PADT (PPPoE Active Discovery Terminate)
 * 这种数据包可以在会话建立以后的任意时刻发送，表明PPPoE会话已经终止。
 * 它可以由主机或访问集中器发送，DESTINATION_ADDR域为单播以太网地址，CODE域设置为0xa7,SESSION_ID必须表明终止的会话，这种数据包不需要任何TAG。
 * 当收到PADT以后，就不允许再使用该会话发送PPP流量了。在发送或接收到PADT后，即使是常规的PPP结束数据包也不允许发送。
 * PPP通信双方应该使用PPP协议自身来结束PPPoE会话，但在无法使用PPP时可以使用PADT。
 
#### 6.PPP会话阶段
  * 一旦PPPoE会话开始，PPP数据就像其它PPP封装一样发送。所有的以太网数据包都是单播的。
  * ETHER_TYPE域设置为0x8864。PPPoE的CODE必须设置为0x00。PPPoE会话的SESSION_ID不允许发生改变，必须是Discovery阶段所指定的值。
  * PPPoE的payload包含一个PPP帧，帧始于PPP Protocol-ID。

## 附录
### TAG类型和TAG值
 * 0x0000 End-Of-List
	 * 表示没有更多的TAG，TAG_LENGTH必须总是0，此TAG不是必需的，它保持向后兼容。   
	 
 * 0x0101 Service-Name
	 * 表示业务名，TAG_LENGTH为0时表示任何业务都是可以接受的。 
	 
 * 0x0102 AC-Name
	 * 唯一标示了一个特定的接入服务器区别于其他的接入服务器，它可以是商标，模块和一系列的ID信息。
	 
 * 0x0103 Host-Uniq
	 * 主机如何知道一个PADO或PADS是对自己发出的PADI 或 PADR的应答？
	 * 通过Host-Uniq唯一识别。TAG_VALUE 是主机任选的一个二进制数值。在PADI 或PADR中，主机可以包含一个Host-Uniq TAG 。
	 * 如果接入服务器收到此TAG，它必须不加修改地包含此TAG在相应的PADO 或PADS 响应中。
	 * 首先主机是知道自己发送的Host-Uniq值的，接入服务器应答时会在应答报文中不加修改地再携带此TAG。
	 
 * 0x0104 AC-Cookie
	 * 此TAG是接入服务器用来防止DOS攻击。接入服务器可以在PADO报文中包含此TAG。
	 * 如果主机收到此TAG，它必须在接下来的PADR报文中不加修改地携带此TAG。TAG_VALUE 是任意的二进制数值，并且不能被主机解释。
	 
 * 0x0105 Vendor-Specific
	 * 此TAG用于传递供应商所有权信息。TAG_VALUE的前四字节是供应商ID,剩下的字节为指定。
	 * 高1字节是0，低3字节是提供商SMI网络管理私用企业代码。
	 
 * 0x0110 Relay-Session-Id
	 * 此TAG可以被一个中间代理加到任何discovery报文中，TAG_VALUE对接入服务器和主机不透明，Relay-Session-Id TAG with a TAG_VALUE 长度为12字节。
	 * Relay-Session-Id TAG不应该加到已经包含了一个Relay-Session-Id TAG的discovery报文中。在这种情况下，中继代理应该已经存在的Relay-Session-Id TAG。
	 * 如果不能使用存在的 TAG 或者没有充足的空间加一Relay- Session-Id TAG，中继代理应该返回Generic-Error TAG 给发送者。
	 
 * 0x0201 Service-Name-Error
	 * 当被请求的服务不支持，此TAG（特别数据部分是0长度）指出原因 。    
	 * 如果有数据，并且数据的第一字节非0，必须用UTF-8字符串解释为什么请求被拒绝。 
	 
 * 0x0202 AC-System-Error
	 * 此TAG显示接入服务器在处理主机请求时遇到错误，如没有足够资源创建一个虚链路，它可以被包含在PADS报文中。 
	 
 * 0x0203 Generic-Error
	 * 当一个不可恢复的错误产生并且没有其他错误适合时，此TAG显示了错误。它能被加到PADO, PADR 或PADS报文中。





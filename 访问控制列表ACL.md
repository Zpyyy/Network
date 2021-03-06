# 访问控制列表
  * 访问控制列表(Access Control List，简称ACL)是根据报文字段对报文进行过滤的一种安全技术。
  * 访问控制列表通过过滤报文达到流量控制、攻击防范以及用户接入控制等功能，在现实中应用广泛。
  * ACL根据功能的不同分为标准ACL和扩展ACL。标准ACL只能过滤报文的源IP地址；扩展ACL可以过滤源IP、目的IP、协议类型、端口号等。
  * ACL的配置主要分为两个步骤：（1）根据需求编写ACL；(2)将ACL应用到路由器接口的某个方向。
  
## 什么是访问控制列表
  1. 访问控制列表（Access Control List）是一种路由器配置脚本，它根据从数据包报头中发现的条件来控制路由器应该允许还是拒绝数据包通过。
  2. 通过接入控制列表可以在路由器、三层交换机上进行网络安全属性配置，可以实现对进入路由器、三层交换机的输入数据流进行过滤，但ACL 对路由器自身产生的数据包不起作用。
  3. 当每个数据包经过关联有 ACL 的接口时，都会与 ACL 中的语句从上到下一行一行进行比对，以便发现符合该传入数据包的模式。
  4. ACL 使用允许或拒绝规则来决定数据包的命运，通过此方式来贯彻一条或多条公司安全策略。还可以配置 ACL 来控制对网络或子网的访问。也可以在VTY线路接口上使用访问控制列表，来保证telnet的连接的安全性。
  5. 默认情况下，路由器上没有配置任何 ACL，不会过滤流量。进入路由器的流量根据路由表进行路由。如果路由器上没有使用 ACL，所有可以被路由器路由的数据包都会经过路由器到达下一个网段。ACL 主要执行以下任务：
      * 限制网络流量以提高网络性能。例如，如果公司政策不允许在网络中传输视频流量，那么就应该配置和应用 ACL 以阻止视频流量。这可以显著降低网络负载并提高网络性能。
      * 提供流量控制。ACL 可以限制路由更新的传输。如果网络状况不需要更新，便可从中节约带宽。
      * 提供基本的网络访问安全性。ACL 可以允许一台主机访问部分网络，同时阻止其它主机访问同一区域。例如，“人力资源”网络仅限选定用户进行访问。
      * 决定在路由器接口上转发或阻止哪些类型的流量。例如，ACL 可以允许电子邮件流量，但阻止所有 Telnet 流量。
      * 控制客户端可以访问网络中的哪些区域。
      * 屏蔽主机以允许或拒绝对网络服务的访问。ACL 可以允许或拒绝用户访问特定文件类型，例如 FTP 或 HTTP。
 ### 基本特点
  * ACL是由一系列permit和deny语句组成的、有序规则的列表，通过匹配报文的相关信息实现对报文的分类；
  * ACL本身只能用于对报文的匹配和分类，而无法实现对报文的过滤功能，针对ACL所匹配的报文的过滤功能，需要特定的机制来实现（如在交换机的接口上使用traffic-filter命令调用ACL来进行报文过滤），它仅仅是一个匹配用的工具；
  * ACL除了能够对报文进行匹配之外，还可以用于匹配路由；
  * ACL是一个使用非常广泛的基础性工具，能够被各种应用或命令调用。
      
## 访问控制列表的分类
  * ACL的类型主要分为IP标准访问控制列表（Standard IP ACL）和IP扩展访问控制列表（Extended IP ACL）两大类：
    * IP标准访问控制列表（Standard IP ACL）。标准 ACL 根据源 IP 地址允许或拒绝流量。数据包中包含的目的地址和端口无关紧要。   
    * IP扩展访问控制列表（Extended IP ACL ）。扩展 ACL 根据多种属性（例如，协议类型、源和 IP 地址、目的 IP 地址、源 TCP 或 UDP 端口、目的 TCP 或 UDP 端口）过滤 IP 数据包，并可依据协议类型信息（可选）进行更为精确的控制。
  * 此外还包括一些复杂的ACL,例如命名ACL，基于时间的ACL,动态ACL,自反ACL等。
  
  * 其他分类标准
    ACL分类| 规则定义描述 |编号范围
    :---:|:---:|:---:
    基本ACL    |仅使用报文的源IP地址、分片标记和时间段信息来定义规则|2000~2999
    高级ACL    |既可使用报文的源IP地址、也可使用目的地址/IP优先级/ToS/DSCP/IP协议类型/ICMP类型/TCP源端口/目的端口/UDP源端口/目的端口号等来定义规则|3000~3999
    二层ACL    |可根据报文的以太帧头信息来定义规则，如根据源MAC地址、目的MAC地址、以太帧协议类型等|4000~4999
    用户自定义ACL|可根据报文偏移位置和偏移量来定义规则|5000~5999
    用户ACL|既可使用IPv4报文的源IP地址或源UCL（USER CONTROL LIST)组，也可使用目的地址/目的UCL组/IP协议类型/ICMP协议类型/TCP源端口/目的端口/UDP源端口/目的端口号等来定义规则|6000~6999
  
## ACL工作原理
  * ACL 要么配置用于入站流量，要么用于出站流量。
    * 入站 ACL 传入数据包经过处理之后才会被路由到出站接口。
    * 入站 ACL 非常高效，如果数据包被丢弃，则节省了执行路由查找的开销。当测试表明应允许该数据包后，路由器才会处理路由工作。
        * 如果数据包报头与某条 ACL 语句匹配，则会跳过列表中的其它语句，由匹配的语句决定是允许还是拒绝该数据包。
        * 如果数据包报头与 ACL 语句不匹配，那么将使用列表中的下一条语句测试数据包。此匹配过程会一直继续，直到抵达列表末尾。
        * 最后一条隐含的语句适用于不满足之前任何条件的所有数据包。这条最后的测试条件与这些数据包匹配，并会发出“拒绝”指令。此时路由器不会让这些数据进入或送出接口，而是直接丢弃它们。
        * 最后这条语句通常称为“隐式 deny any 语句”或“拒绝所有流量”语句。由于该语句的存在，所以 ACL 中应该至少包含一条 permit 语句，否则 ACL 将阻止所有流量。
    * 出站 ACL 传入数据包路由到出站接口后，由出站 ACL 进行处理。
        * 在数据包转发到出站接口之前，路由器检查路由表以查看是否可以路由该数据包。
        * 如果该数据包不可路由，则丢弃它。接下来，路由器检查出站接口是否配置有 ACL。如果出站接口没有配置 ACL，那么数据包可以发送到输出缓冲区。
        
## ACL匹配实现原理
 ### ACL规则
   1. ACL规则的编号范围是0～4294967294，所有规则均按照规则编号从小到大进行排序。所以，rule 5排在首位，而规则编号最大的rule 4294967294排在末位。
   2. 系统按照规则编号从小到大的顺序，将规则依次与报文匹配，一旦匹配上一条规则即停止匹配。
   3. 除了包含ACL动作和规则编号，ACL规则中还定义了源地址、生效时间段这样的字段。这些字段，称作匹配选项，它是ACL规则的重要组成部分。
   4. 可以选择二层以太网帧头信息（如源MAC、目的MAC、以太帧协议类型）作为匹配选项，也可以选择三层报文信息（如源地址、目的地址、协议类型）作为匹配选项，还可以选择四层报文信息（如TCP/UDP端口号）。
 ### ACL通配符
   * 通配符是一个32比特长度的数值，用于指示IP地址中，哪些比特位需要严格匹配，哪些比特位则无所谓
   * 通配符通常采用类似网络掩码的点分十进制形式表示，但是含义却与网络掩码完全不同
     网络掩码|通配符
     :----|:----
     255.255.255.0|0.0.0.255
     用于指示IP地址中，哪些比特是网络部分，那些比特是主机部分|用于指示IP地址中，哪些比特位需要严格匹配，哪些比特位则无所谓
     网络掩码中为1的比特表示网络部分|通配符中为1的比特表示无需匹配
     网络掩码中为0的比特表示主机部分|通配符中为0的比特表示严格匹配
  * 实例：匹配192.168.1.0/24这个子网中后8位组成奇数的IP地址，如192.168.1.1、192.168.1.3、192.168.1.5等
         * 规则：192.168.1.1 0.0.0.255.0
 ### ACL步长
   1. rule 5 deny source 1.1.1.1 0  //表示禁止源IP地址为1.1.1.1的报文通过                  
   2. rule 10 deny source 1.1.1.2 0 //表示禁止源IP地址为1.1.1.2的报文通过                   
   3. rule 15 permit source 1.1.1.0 0.0.0.255 //表示允许源IP地址为1.1.1.0网段的报文通过
   * 由于ACL匹配报文时遵循“一旦命中即停止匹配”的原则，所以源IP地址为1.1.1.1和1.1.1.2的报文，会在匹配上编号较小的rule 5和rule 10后停止匹配，从而被系统禁止通过；而源IP地址为1.1.1.3的报文，则只会命中rule 15，从而得到系统允许通过。要想让源IP地址为1.1.1.3的报文也被禁止通过，必须为该报文配置一条新的deny规则。
 
 ### 规则匹配顺序
   * 配置顺序，即系统按照ACL规则编号从小到大的顺序进行报文匹配，规则编号越小越容易被匹配。后插入的规则，如果你指定的规则编号更小，那么这条规则可能会被先匹配上。ACL规则的生效前提，是要在业务模块中应用ACL。当ACL被业务模块引用时，你可以随时修改ACL规则，并且规则修改后会立即生效。
   * 自动排序，是指系统使用“深度优先”的原则，将规则按照精确度从高到底进行排序，系统按照精确度从高到低的顺序进行报文匹配。规则中定义的匹配项限制越严格，规则的精确度就越高，即优先级越高，那么该规则的编号就越小，系统越先匹配。例如，有一条规则的目的IP地址匹配项是一台主机地址2.2.2.2/32，而另一条规则的目的IP地址匹配项是一个网段2.2.2.0/24，前一条规则指定的地址范围更小，所以其精确度更高，系统会优先将报文与前一条规则进行匹配。
   *在自动排序的ACL中配置规则，不允许自行指定规则编号。系统能自动识别出该规则在这条ACL中对应的优先级，并为其分配一个适当的规则编号。*
    
## 3P原则
   * 在路由器上应用 ACL 的一般规则我们简称为3P原则。即我们可以为每种协议 (per protocol)、每个方向 (per direction)、每个接口 (per interface) 配置一个 ACL：
      * 每种协议一个 ACL： 要控制接口上的流量，必须为接口上启用的每种协议定义相应的 ACL。
      * 每个方向一个 ACL：一个 ACL 只能控制接口上一个方向的流量。要控制入站流量和出站流量，必须分别定义两个 ACL。
      * 每个接口一个 ACL：一个 ACL 只能控制一个接口（例如快速以太网 0/0）上的流量。
      
## ACL的放置位置
  * 每一个路由器接口的每一个方向，每一种协议只能创建一个ACL；在适当的位置放置 ACL 可以过滤掉不必要的流量，使网络更加高效。
  * ACL 可以充当防火墙来过滤数据包并去除不必要的流量。ACL 的放置位置决定了是否能有效减少不必要的流量。例如，会被远程目的地拒绝的流量不应该消耗通往该目的地的路径上的网络资源。
  * 每个 ACL 都应该放置在最能发挥作用的位置。基本的规则是：
      1. 将扩展 ACL 尽可能靠近要拒绝流量的源。这样，才能在不需要的流量流经网络之前将其过滤掉。
      2. 因为标准 ACL 不会指定目的地址，所以其位置应该尽可能靠近目的地。
      
## ACL应用范围
   业务分类|应用场景|涉及业务模块
   :---|:---|:---
   登录控制|对交换机的登录权限进行控制，允许合法用户登录，拒绝非法用户登录，从而有效防止未经授权的用户的非法接入。<br>例如，一般情况下交换机只允许管理员登录，非管理员用户不允许登录。这事就可以在Telnet中应用ACL，并在ACL中定义哪些主机可以登录哪些不能|Telnet/SNMP/FTP/TFTP/SFTP/HTTP
  对转发报文进行过滤|对转发的报文进行过滤，从而使交换机能够进一步对过滤出的报文进行丢弃、修改优先级、重定向、IPSEC保护等处理。<br>例如可以利用ACL,降低P2P下载、网络视频消耗大量带宽的数据流的服务等级，在网络拥塞时有限丢弃这类流量，减少他们对其他重要流量的影响|Qos流策略、NAT、IPSEC
  对上送CPU处理的报文进行过滤|对上送CPU的报文进行必要的限制，可以避免CPU处理过多的协议报文造成占用率过高、性能下降<br>例如，发现某用户向交换机发送大量的ARP攻击报文，造成交换机CPU繁忙，引发系统中断。这是就可以在本机防攻击策略的黑名单中利用ACL，将该用户加入黑名单，使CPU丢弃该用户发送的报文|黑名单、白名单、用户自定义流
  路由过滤|ACL可以应用在各种动态路由协议中，对路由协议发布和接受的路由信息进行过滤。<br>例如，可以将ACL和路由策略配合使用，禁止交换机将某段路由发给邻居路由器。|BGP、IS-IS、OSPF、OSPFv3、RIP、RIPng、组播协议
    

1. FTTx
fiber to the ...

FTTx是光纤接入网，一种接入网应用类型

FTTx基本组网：是根据光纤离最终消费者的远近的一种统称，而由远到近（铜线距离由长到短）可以分为以下几种：

FTTN             node 指的是一个街边柜，离用户几公里，大于300m（1000ft）

FTTC             curb or cabinet 指交接箱 到街边柜

FTTB             building or basement 到楼道或弱电井

FTTH             home

FTTP             premise 场所

FTTD             desk

FTTM             machine/还有一种说法是光纤到移动基站 挂墙/挂杆


 2. PON网络

形成FTTx应用的网络称为PON（无源光网络passive optical network）                    

PON是光纤接入技术，P2MP（点对多点）的主要技术是PON（在ODN中引入splitter完成一对多的分光）

典型的PON网络分光以1:64为主，覆盖距离5到10km



PON网络由三部分组成，分别为：

OLT（光线路终端 optical line terminal）提供面向用户的无源光纤网络的光纤接口

ODN（光分配网optical distribution network）提供光传输通道

ONT/ONU（光网络终端/单元optical network terminal or unit）用户端设备


其中ONU根据应用场景不同又可分为：

MTU（multi-tenant unit多租户单元），MDU（multi-dwelling unit多住户单元），统称为MxU

区别主要是MTU支持E1业务，MDU可以用于商业客户。
MTU 功能上可以取代FR（Frame Relay 帧中继）、TDM（time division multiplexing时分复用）/ATM（asynchronous transfer mode异步传输模式）。
SBU（单商户单元single business unit）用于FTTO（office），SFU（单住户单元single family unit）用于FTTH。 

SBU与SFU的主要区别在于SBU支持E1业务，当商业客户不需要E1业务时，SFU可以用于商业客户。
HGU（家庭网关单元home gateway unit）用于单独家庭用户，具有家庭网关功能，相当于带PON上联接口的家庭网关。



3. OLT组网结构

接入网总架构：

ONU后，跟FTTx组合的几种接入模式：

CMC（Cable Media Converter 有线电缆媒介转换设备，例如华为的MA5633）
CPE（customer-premises equipment 用户驻地设备，位于终端用户驻地的设备） 
CM（Cable Modem 电缆调制解调器）      
PON到CMC完成转换，最终经过CM调制，最终供用户使用。
CPE包括CM（FTTx+另一种接入网），或ONT（FTTx直接到户）。
光纤接入网加铜线接入网、光纤接入网加同轴接入网、光纤接入网加以太网以及直接光纤接入网。
前三种都是PON无源光网络跟另一种介质的组合，PON在经过分光器后会进入ONU（ONT或一种CMC有线电缆媒介转换设备的）。PON一旦到达ONU，不管是不是最终用户端，FTTx的光纤部分就算结束了。

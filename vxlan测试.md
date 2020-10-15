ifc规则配置
    cli home/cli/func/acl/ifc_item_clr begin 10 end 10
    cli home/cli/func/acl/rule_add field 5 key 101
    cli home/cli/func/acl/ifc_item_set id 10 valid 1 igrs 0x200 up_tag.stag.vlan_source 1 up_tag.stag.vlan 100 trans_act 4 trans_egr 0x400
一定需要配两条vlan （不懂为什么要环回）
(环回路径：入口匹配ifc10 -> ifc11 -> ifc13 -> ofc100 -> ude -> prbs -> ofc100(end))

上行（封装）：eth1 -> DP -> PRBS -> DP -> eth2
下行（解封装）：eth2 -> DP -> CPU(解vxlan) -> DP -> eth1
up_tag.stag.vlan_source 值配1 表示point指向 由vlan101指向vlan100


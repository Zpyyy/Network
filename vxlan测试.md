# ifc规则配置

    #入口流规则配置前先删除该条规则
    cli home/cli/func/acl/ifc_item_clr begin 10 end 10 
    
    # 添加acl规则 匹配vlan头 vlanid = 101
    cli home/cli/func/acl/rule_add field 5 key 101
    
    # ifc规则设置（一般用up参数，不用down）valid:使能 igrs:入口 trans_egr：出口 trans_act：point命中 即：eth1入口vlan101流命中vlan100流至eth2流出口
    cli home/cli/func/acl/ifc_item_set id 10 valid 1 igrs 0x200 up_tag.stag.vlan_source 1 up_tag.stag.vlan 100 trans_act 4 trans_egr 0x400
    
    

一定需要配两条vlan （不懂为什么要环回）
(环回路径：入口匹配ifc10 -> ifc11 -> ifc13 -> ofc100 -> ude -> prbs -> ofc100(end))

上行（封装）：eth1 -> DP -> PRBS -> DP -> eth2
下行（解封装）：eth2 -> DP -> CPU(解vxlan) -> DP -> eth1
up_tag.stag.vlan_source 值配1 表示point指向 由vlan101指向vlan100

# vxlan流生成
  1. 流规则属性（filter开头参数） 只有匹配了刘规则才会进入该隧道进行vxlan封装
   * ingress &emsp;&emsp;&emsp;&emsp;&emsp; ---入口
   * level   &emsp;&emsp;&emsp;&emsp;&emsp; ---流优先级，当报文同时命中多条pdu流时，取优先级高的
   * smac_mode &emsp;&emsp;&emsp; ---源mac匹配模式，支持不参与匹配、等于匹配（ignore/equal）
   * smac      &emsp;&emsp;&emsp;&emsp;&emsp;  ---源mac，注意含掩码bit数量[0,15]
   * tag_type_mode  &emsp;&emsp;---tag类型匹配模式，支持不参与匹配，等于匹配
   * tag_type       &emsp;&emsp;&emsp;&emsp;---tag类型 untag/tag
   * svlan_mode     &emsp;&emsp;&emsp;---外层vlan匹配模式，支持不参与、等于匹配
   
  2. 隧道动作（tunnel开头参数） 流规则匹配后进入隧道的封装动作
    
    cli home/cli/api/flow/vxlan/vxlanflow_create flow_id 1 filter_ingress.type eth filter_ingress.mask 0x1 filter_level 6 filter_smac_mode 1 filter_smac.mac 00:00:00:00:00:22 filter_tag_type_mode 1 filter_tag_type 1 filter_svlan_mode 1 filter_svlan 100 tunnel_egress.type 4 tunnel_egress.mask 0x2 tunnel_mtu 500 tunnel_smac.mac 00:00:00:11:11:11 tunnel_dmac.mac 00:00:00:22:22:22 tunnel_svlan_act 1 tunnel_svlan 1 tunnel_pppoe_act 0 tunnel_pppoe_encap_version 1 tunnel_pppoe_encap_type 1 tunnel_pppoe_encap_protol_type 0x21 tunnel_sip.version 1 tunnel_sip.addr 6.3.175.136 tunnel_dip.version 1 tunnel_dip.addr 192.168.0.1 tunnel_dscp 0 tunnel_sport 100 tunnel_dport 4789 tunnel_vni 5060
    
    


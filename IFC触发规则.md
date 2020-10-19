# 1、IFC简介

  ## IFC是什么
   * Initial Filter Criteria:初始过滤规则，它用以描述sip消息经过SCSCF的时候，根据设计方案需要出发到何种设备上去的规则。注意，此处说的是何种设备，并非只能触发到某个AS上。
  ## 为什么要有IFC
   * IFC是在编写用户开户脚本时，需要反复研究、调整的一组数据配置，他是IMS域从整网角度来实现网络规划方案的一种重要手段，特别是在配有多个AS，且每个AS下行有多个呼叫腿的时候。
    
    
# 2、IFC的结构和数据配置

  ## IFC结构
   * 一条IFC涉及到以下数据项：
      IFC = IMPU+PRIORITY+SERVER+DEFHND+SERVINFO+TRIGPT
     * 重点：
       1. IMPU,关注是那个号码的触发
       2. SERVER，出发到那个设备上
       3. 在什么情况下触发
       
  ## IFC的数据配置
   * IMPU
    * 需要进行配置的IMS用户，包括sip uri和tel uri。例如：<BR>
    ```
        A开户配置了IFC 1、2、3，B配置了IFC 4、5
        在A呼叫B的呼叫过程中，主叫测触发要去匹配A上的触发规则，而被叫测则要去匹配B上的触发规则，这就是说
        单次呼叫重点只观察主叫测或者被叫测用户的开户脚本，是无法确定一个呼叫的触发流程的。
    ```
  * PUSI
  
  
<font color=red>你是什么色</font>

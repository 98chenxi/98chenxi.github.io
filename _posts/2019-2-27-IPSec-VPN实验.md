---
layout: post
title:  "IPsec vpn实验"
date:   2019-2-27 21:55:00
categories: [ipsec, vpn]
---


### 实验拓扑

![@拓扑图 | center ](https://i.imgur.com/8caH2gc.png)
### 实验需求
​	1、 根据图示配置ip地址。
​	2、 在R1和R3上配置默认路由，确保公网连通性。
​	3、 在R1和R3上配置IPSec vpn，使pc4和pc5可以互通。

### 实验配置
&emsp; **1、 配置各个设备的ip地址**
　　　R1
　　　[r1]int g0/0
　　　[r1-GigabitEthernet0/0]ip add 192.168.1.254 24
　　　[r1-GigabitEthernet0/0]int g0/1
　　　[r1-GigabitEthernet0/1]ip add 100.1.1.1 24
　　　
　　　R2
　　　[r2]int g0/0
　　　[r2-GigabitEthernet0/0]ip add 100.1.1.2 24
　　　[r2-GigabitEthernet0/0]int g0/1
　　　[r2-GigabitEthernet0/1]ip add 200.1.1.2 24
　　　
　　　R3
　　　[r3]int g0/0
　　　[r3-GigabitEthernet0/0]ip add 200.1.1.3 24
　　　[r3-GigabitEthernet0/0]int g0/1
　　　[r3-GigabitEthernet0/1]ip add 102.168.2.254 24
　　　
　![@pc1与pc2的直连测试 | center ](./1551238315117.png)

------

&emsp; **2、 在R1和R3上配置默认路由**
　　　R1
　　　[r1]ip route-static 0.0.0.0 0.0.0.0 100.1.1.2

　　　R3
　　　[r3]ip route-static 0.0.0.0 0.0.0.0 200.1.1.2
　　　![@公网可通 | center ](./1551238784719.png)

------
&emsp; **3、 在R1上配置IKE和IPSec 策略**
　　　*3.1 使用ACL抓取需要加密的流量（感兴趣流）*
　　　&emsp; [r1]acl advanced 3000
　　　&emsp; [r1-acl-ipv4-adv-3000]rule permit ip source 192.168.1.0 0.0.0.255 des 192.168.2.0 0.0.0.255
　　　
　　　*3.2 配置IKE的提议（proposal）*
　　　&emsp; [r1]ike proposal 1　　　　　`/创建并进入Ike提议视图`
　　　&emsp; [r1-ike-proposal-1]authentication-method pre-share　　　`/指定ike提议使用的认证方法`
　　　&emsp; [r1-ike-proposal-1]encryption-algorithm aes-cbc-128　　　　　`/指定ike加密的算法`
　　　
　　　*3.3 配置预共享密钥内容*
　　　&emsp; [r1]ike keychain ccc 　　　`/密钥名称为ccc`
　　　&emsp; [r1-ike-keychain-ccc]pre-shared-key address 200.1.1.3 key simple ccc　　　`/对端地址和验证密码`
　　　
　　　*3.4 配置第一阶段协商的内容*
　　　&emsp; [r1]ike profile ccc
　　　&emsp; [r1-ike-profile-ccc]keychain ccc　　　`/指定预共享密钥`
　　　&emsp; [r1-ike-profile-ccc]local-identity address 100.1.1.1　　　`/本地身份信息`
　　　&emsp; [r1-ike-profile-ccc]match remote identity address 200.1.1.3　　　`/远端身份信息`
　　　&emsp; [r1-ike-profile-ccc]proposal 1　　　`/指定提议`
　　　
　　　到此为止，IKE第一阶段的协商参数已经配置完毕。接下来配置第二阶段的协商参数
　　　
　　　*3.5 配置转换集*
　　　&emsp; [r1]ipsec transform-set ccc
　　　&emsp; [r1-ipsec-transform-set-ccc]esp authentication-algorithm sha1　　`/指定认证方式`
　　　&emsp; [r1-ipsec-transform-set-ccc]esp encryption-algorithm aes-cbc-128　　`/指定加密方式`
　　　
　　　*3.6 配置IPSec策略*
　　　&emsp; [r1]ipsec policy ccc 1 isakmp　　
　　　&emsp; [r1-ipsec-policy-isakmp-ccc-1]transform-set ccc
　　　&emsp; [r1-ipsec-policy-isakmp-ccc-1]security acl 3000
　　　&emsp; [r1-ipsec-policy-isakmp-ccc-1]remote-address 200.1.1.3
　　　&emsp; [r1-ipsec-policy-isakmp-ccc-1]ike-profile ccc
　　　
　　　*3.7 在公网接口使用这个策略*
　　　&emsp; [r1-GigabitEthernet0/1]ipsec apply policy ccc

------
&emsp; **4、 在R3上配置IKE和IPSec 策略**
　　　*4.1 使用ACL抓取需要加密的流量（感兴趣流）*
　　　&emsp; [r3]acl advanced 3000
　　　&emsp; [r3-acl-ipv4-adv-3000]rule per ip sour 192.168.2.0 0.0.0.255 des 192.168.1.0 0.0
.0.255

　　　*4.2 配置IKE的提议（proposal）*
　　　&emsp; [r3]ike proposal 1
　　　&emsp; [r3-ike-proposal-1]authentication-method pre-share
　　　&emsp; [r3-ike-proposal-1]encryption-algorithm aes-cbc-128
　　　
　　　*4.3 配置预共享密钥内容*
　　　&emsp; [r3]ike keychain ccc 
　　　&emsp; [r3-ike-keychain-ccc]pre-shared-key address 100.1.1.1 key simple ccc　　

　　　*4.4 配置第一阶段协商的内容*
　　　&emsp; [r3]ike profile ccc
　　　&emsp; [r3-ike-profile-ccc]keychain ccc　
　　　&emsp; [r3-ike-profile-ccc]local-identity address 200.1.1.3　
　　　&emsp; [r3-ike-profile-ccc]match remote identity address 100.1.1.1　
　　　&emsp; [r3-ike-profile-ccc]proposal 1　
　　　
　　　*4.5 配置第二阶段加密方式*
　　　&emsp; [r3]ipsec transform-set ccc
　　　&emsp; [r3-ipsec-transform-set-ccc]esp authentication-algorithm sha1　　
　　　&emsp; [r3-ipsec-transform-set-ccc]esp encryption-algorithm aes-cbc-128　

　　　*4.6 配置IPSec策略*
　　　&emsp; [r3]ipsec policy ccc 1 isakmp　　
　　　&emsp; [r3-ipsec-policy-isakmp-ccc-1]transform-set ccc
　　　&emsp; [r3-ipsec-policy-isakmp-ccc-1]security acl 3000
　　　&emsp; [r3-ipsec-policy-isakmp-ccc-1]remote-address 100.1.1.1
　　　&emsp; [r3-ipsec-policy-isakmp-ccc-1]ike-profile ccc
　　　
　　　*4.7 开启抓包，并在公网接口调用策略*
　　　&emsp; [R3]int g0/0
　　　&emsp; [R3-GigabitEthernet0/0]ipsec apply policy ccc

------
### 实验结果
先是什么都没有抓到，然后ping也一直ping不同，只抓到了快速包。
![@失败图 | center ](./1551249336620.png)
![@我发现网关配置错了](./1551249400866.png)
修改之后还是不行
![@这是说明第一阶段没问题？？？](./1551249636672.png)
![@可能是第二阶段不行](./1551249801513.png)
![@这里转换协议应该是MD5](./1551250046154.png)
![@经过修改网关和认证方式为MD5，成功了！！](./1551250135290.png)
![Alt text](./1551250319089.png)
![Alt text](./1551250371556.png)
![Alt text](./1551250466481.png)

------
### 总结
IPSec的协商和建立，传输数据可以分为两个阶段。
第一阶段主要负责生成加密密钥，协商身份认证参数等。
如下图所示
（数据包）
红色标识的部分为协商的参数。
ISAKMP Main Mode 的6个包 Quick Mode的3个包

后续的数据包经过加密，无法解析出其具体内容，参考理论部分的IPSec的协商过程。

最后 ICMP会经过加密，封装成ESP格式。如下所示

作为操作者，其实是两个内网的设备在互访。（192.168.1.1访问192.168.2.1）
但是实际上抓包中只能看到 路由器的两个公网IP互访。

其内容应该为ICMP 但是经过加密以后，是无法窥探其内容的。

这便是IPSec加密的效果。




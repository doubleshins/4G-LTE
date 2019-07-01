2019/05/14 - 防火牆
第一個是 Server 上面的防火牆

#!/bin/bash

# 0. user define
dicips="120.114.140.0/24,120.114.141.0/24,120.114.142.0/24"
dicports="80,443,5911,5931,5951"

# 1. clean rule
iptables -F
iptables -X
iptables -Z

# 2. create policy
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

# 3. create rules
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
# port 22 only for DIC class room.
iptables -A INPUT -s ${dicips} -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -s ${dicips} -p tcp -m state --state NEW -m tcp -m multiport --dports ${dicports} -j ACCEPT
iptables -A INPUT -i mybr0 -j ACCEPT

# 4. NAT
iptables -t nat -F
iptables -t nat -X
iptables -t nat -Z
iptables -t nat -A PREROUTING -p tcp --dport 5050 -j REDIRECT --to-port 22
iptables -t nat -A POSTROUTING -s 192.168.19.0/24 -o eno1 -j MASQUERADE
iptables -t nat -A PREROUTING -i eno1 -p tcp -m tcp --dport 8080 -j DNAT --to-destination 192.168.19.9:443
iptables -t nat -A PREROUTING -i eno1 -p tcp -m tcp --dport 8888 -j DNAT --to-destination 192.168.19.1:443

# final. save rule
iptables-save > /etc/sysconfig/iptables
第二個是 ovirt engine 的防火牆

#!/bin/bash

# 0. user define
ovirttcp="80,5432,443,6641,6642,54323,6100,80,2222,9696,35357"
ovirtudp="7410"

iptables -F
iptables -X
iptables -Z

iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -s 192.168.19.0/24 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp -m multiport --dports ${ovirttcp} -j ACCEPT
iptables -A INPUT -p udp                             -m multiport --dports ${ovirtudp} -j ACCEPT
iptables -A INPUT -j REJECT

iptables-save > /etc/sysconfig/iptables

## 參考資料
- 4G/LTE/公司/社區網路虛擬IP轉實體IP方案 : https://www.enchose.com/index.php/network-solution/vpn-port-forwarding



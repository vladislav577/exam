# Инструкция по демоэкзамену (Модуль 1)

## Таблица адресации

| Устройство | Назначение | IP | Маска | Шлюз |
|---|---|---|---|---|
| HQ-RTR | HQ-SRV (VLAN100) | 172.16.10.1 | /27 | |
| HQ-RTR | HQ-CLI (VLAN200) | 172.16.20.1 | /28 | |
| HQ-RTR | Управление (VLAN999) | 172.16.30.1 | /29 | |
| BR-RTR | BR-SRV | 192.168.30.1 | /27 | |
| ISP | BR-RTR | 172.16.2.1 | /28 | |
| ISP | HQ-RTR | 172.16.1.1 | /28 | |
| HQ-RTR | GRE1 | 172.16.100.2 | /29 | |
| BR-RTR | GRE1 | 172.16.100.1 | /29 | |
| BR-SRV | BR-RTR | 192.168.30.2 | /27 | 192.168.30.1 |
| HQ-SRV | VLAN999 | 172.16.30.2 | /29 | |
| HQ-SRV | VLAN100 | 172.16.10.2 | /27 | 172.16.10.1 |
| HQ-CLI | VLAN200 | 172.16.20.2 | /28 | 172.16.20.1 |

---

## ISP

```bash
hostnamectl set-hostname isp.au-team.irpo && exec bash
mkdir /etc/net/ifaces/ens19
nano /etc/net/ifaces/ens19/options
TYPE=eth
BOOTPROTO=dhcp
DISABLED=no
CONFIG_IPV4=yes
systemctl restart network
apt-get update && apt-get install mc iptables -y
nano /etc/net/sysctl.conf
net.ipv4.ip_forward = 1
systemctl restart network
iptables -t nat -A POSTROUTING -o ens19 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
HQ-RTR
hostnamectl set-hostname hq-rtr.au-team.irpo && exec bash
mkdir -p /etc/net/ifaces/ens19
nano /etc/net/ifaces/ens19/ipv4address
172.16.1.2/28
nano /etc/net/ifaces/ens19/ipv4route
default via 172.16.1.1
nano /etc/net/sysctl.conf
net.ipv4.ip_forward = 1
systemctl restart network
iptables -t nat -A POSTROUTING -o ens19 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
BR-RTR
hostnamectl set-hostname br-rtr.au-team.irpo && exec bash
mkdir -p /etc/net/ifaces/ens19
nano /etc/net/ifaces/ens19/options
TYPE=eth
BOOTPROTO=static
DISABLED=no
CONFIG_IPV4=yes
nano /etc/net/ifaces/ens19/ipv4address
172.16.2.2/28
nano /etc/net/ifaces/ens19/ipv4route
default via 172.16.2.1
iptables -t nat -A POSTROUTING -o ens19 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
BR-SRV
hostnamectl set-hostname br-srv.au-team.irpo && exec bash
nano /etc/net/ifaces/ens192/ipv4address
192.168.30.2/28
nano /etc/net/ifaces/ens192/ipv4route
default via 192.168.30.1
Пользователи
useradd net_admin
passwd net_admin
usermod -a -G wheel net_admin
Пароль:

P@ssw0rd
VLAN (HQ-RTR)
172.16.10.1/27  (ens20.100)
172.16.20.1/28  (ens20.200)
172.16.30.1/29  (ens20.999)
GRE + OSPF (HQ-RTR)
mkdir /etc/net/ifaces/gre1
nano /etc/net/ifaces/gre1/options
TUNLOCAL=172.16.1.2
TUNREMOTE=172.16.2.2
TUNTYPE=gre
TYPE=iptun
TUNTTL=64
TUNMTU=1476
nano /etc/net/ifaces/gre1/ipv4address
172.16.100.2/29
apt-get install frr -y
systemctl enable --now frr
nano /etc/frr/demons
ospfd=yes
vtysh
router ospf
network 172.16.100.0/29 area 0
network 172.16.20.0/28 area 0
network 172.16.10.0/27 area 0
SSH (HQ-SRV / BR-SRV)
Port 2026
Match User sshuser
Banner /etc/openssh/Banner.txt
MaxAuthTries 2
DNS (HQ-SRV)
apt-get install dnsmasq
nano /etc/dnsmasq.d/name.conf
systemctl enable --now dnsmasq
Проверка
ping
nslookup hq-srv.au-team.irpo

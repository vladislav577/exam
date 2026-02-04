# DemoExam Network Setup (Module 1)

Полная реализация инструкции из файла **«Инструкция по демоэкзамену (Модуль 1)»** в виде GitHub‑репозитория.

Репозиторий предназначен для автоматизированной и полуавтоматической настройки всех узлов:

* ISP
* HQ-RTR
* BR-RTR
* HQ-SRV
* BR-SRV
* HQ-CLI

---

## 📁 Структура репозитория

```text
.
├── README.md
├── common/
│   ├── sysctl.sh
│   ├── users.sh
│   └── nat.sh
├── isp/
│   └── setup.sh
├── hq-rtr/
│   ├── network.sh
│   ├── vlan.sh
│   ├── gre_ospf.sh
│   └── dhcp.sh
├── br-rtr/
│   ├── network.sh
│   └── gre_ospf.sh
├── hq-srv/
│   ├── network.sh
│   ├── ssh.sh
│   └── dns.sh
├── br-srv/
│   ├── network.sh
│   └── ssh.sh
└── hq-cli/
    └── network.sh
```

---

## 🚀 Использование

На каждой машине:

```bash
apt update && apt install git -y
git clone https://github.com/USERNAME/demoexam-network-setup.git
cd demoexam-network-setup
bash <путь_к_нужному_скрипту>
```

---

## 🔧 common/sysctl.sh

```bash
#!/bin/bash
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/' /etc/net/sysctl.conf
systemctl restart network
```

---

## 🔧 common/users.sh

```bash
#!/bin/bash
USER=$1
UID_VAL=$2

useradd -u ${UID_VAL} ${USER}
echo "${USER}:P@ssw0rd" | chpasswd
usermod -aG wheel ${USER}
apt-get install sudo nano -y
sed -i 's/#WHEEL_USERS/WHEEL_USERS/' /etc/sudoers
```

---

## 🔧 common/nat.sh

```bash
#!/bin/bash
apt-get install iptables -y
iptables -t nat -A POSTROUTING -o ens19 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
```

---

## 🌐 isp/setup.sh

```bash
#!/bin/bash
hostnamectl set-hostname isp.au-team.irpo
mkdir -p /etc/net/ifaces/ens19
cat > /etc/net/ifaces/ens19/options <<EOF
TYPE=eth
BOOTPROTO=dhcp
DISABLED=no
CONFIG_IPV4=yes
EOF
systemctl restart network
bash ../common/sysctl.sh
bash ../common/nat.sh
```

---

## 🌐 hq-rtr/network.sh

```bash
#!/bin/bash
hostnamectl set-hostname hq-rtr.au-team.irpo
mkdir -p /etc/net/ifaces/ens19
cat > /etc/net/ifaces/ens19/ipv4address <<EOF
172.16.1.2/28
EOF
cat > /etc/net/ifaces/ens19/ipv4route <<EOF
default via 172.16.1.1
EOF
systemctl restart network
bash ../common/sysctl.sh
bash ../common/nat.sh
```

---

## 🌐 hq-rtr/vlan.sh

```bash
#!/bin/bash
for vlan in 100 200 999; do
  mkdir -p /etc/net/ifaces/ens20.${vlan}
  cat > /etc/net/ifaces/ens20.${vlan}/options <<EOF
TYPE=vlan
HOST=ens20
VID=${vlan}
EOF
done

echo "172.16.10.1/27" > /etc/net/ifaces/ens20.100/ipv4address
echo "172.16.20.1/28" > /etc/net/ifaces/ens20.200/ipv4address
echo "172.16.30.1/29" > /etc/net/ifaces/ens20.999/ipv4address
systemctl restart network
```

---

## 🌐 hq-rtr/gre_ospf.sh

```bash
#!/bin/bash
apt-get install frr -y
sed -i 's/ospfd=no/ospfd=yes/' /etc/frr/demons
systemctl enable --now frr
mkdir -p /etc/net/ifaces/gre1
cat > /etc/net/ifaces/gre1/options <<EOF
TYPE=iptun
TUNTYPE=gre
TUNLOCAL=172.16.1.2
TUNREMOTE=172.16.2.2
TUNTTL=64
TUNMTU=1476
EOF
echo "172.16.100.2/29" > /etc/net/ifaces/gre1/ipv4address
systemctl restart network
```

---

## 🌐 br-rtr/network.sh

```bash
#!/bin/bash
hostnamectl set-hostname br-rtr.au-team.irpo
mkdir -p /etc/net/ifaces/ens19
echo "172.16.2.2/28" > /etc/net/ifaces/ens19/ipv4address
echo "default via 172.16.2.1" > /etc/net/ifaces/ens19/ipv4route
systemctl restart network
bash ../common/nat.sh
```

---

## 🖥️ hq-srv/ssh.sh

```bash
#!/bin/bash
apt-get install openssh-server -y
sed -i 's/#Port 22/Port 2026/' /etc/openssh/sshd_config
echo "Authorized access only" > /etc/openssh/Banner.txt
systemctl restart sshd
```

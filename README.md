ИНСТРУКЦИЯ

устройство	Назначение	IP-адрес	Маска	Шлюз по умолчанию
HQ-RTR	HQ-SRV (VLAN100)	172.16.10.1	27	
HQ-RTR	 HQ-CLI (VLAN200)	172.16.20.1	28	
HQ-RTR	Управление (VLAN999)	172.16.30.1	29	
BR-RTR	BR-SRV	192.168.30.1	27	
ISP	BR-RTR	172.16.2.1	28	
ISP	HQ-RTR	172.16.1.1	28	
HQ-RTR	GRE1	172.16.100.2	29	
BR-RTR	GRE1	172.16.100.1	29	
BR-RTR	ISP	172.16.2.2	28	172.16.2.1
HQ-RTR	ISP	172.16.1.2	28	172.16.1.1
BR-SVR	BR-RTR	192.168.30.2	27	192.168.30.1
HQ-SRV	HQ-RTR
(VLAN999)	172.16.30.2	29	
HQ-SRV	HQ-RTR
(VLAN100)	172.16.10.2	27	172.16.10.1
HQ-CLI	HQ-RTR
(VLAN200)	172.16.20.2	28	172.16.20.1

НАСТРОЙКА НА ISP
Задаем имя isp
hostnamectl set-hostname isp.au-team.irpo && exec bash
Создайте каталог для интерфейса:
mkdir  /etc/net/ifaces/ens19
vim /etc/net/ifaces/ens19/options
Cодержимое:
TYPE=eth
BOOTPROTO=dhcp
DISABLED=no
CONFIG_IPV4=yes
:wq
Systemctl restart network

apt-get update
apt-get install mc

Отредактируйте файл настроек:
/etc/net/ifaces/ens20/options
 
/etc/net/ifaces/ens20/ipv4address
172.16.1.1/28
Создайте файл для задания IP-адреса:
/etc/net/ifaces/ens21/options
 
touch /etc/net/ifaces/ens256/ipv4address
172.16.1.1/28
Включение пересылки пакетов:
Отредактируйте файл /etc/net/sysctl.conf:
Измените строку:
net.ipv4.ip_forward = 0
на:
net.ipv4.ip_forward = 1
systemctl restart network


Далее настройка iptables:
apt-get update 
 apt-get install nano iptables -y
iptables -t nat -A POSTROUTING -o ens19 -j MASQUERADE
Сохраните правила:
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
После всей настройки делаем синхронизацию времени:
 
 
 
Настройка на HQ-RTR
Задаем имя HQ-RTR:
hostnamectl set-hostname hq-rtr.au-team.irpo && exec bash
Конфигурация интерфейса
mkdir -p /etc/net/ifaces/ens19
vim /etc/net/ifaces/ens19/options
 
:wq
vim /etc/net/ifaces/ens19/ipv4address
172.16.1.2/28
:wq
vim /etc/net/ifaces/ens19/ipv4route
default via 172.16.1.1
Systemctl restart network
Отредактируйте файл /etc/net/sysctl.conf:
/etc/net/sysctl.conf
nameserver 8.8.8.8
Измените строку:
net.ipv4.ip_forward = 0
на:
net.ipv4.ip_forward = 1
Примените изменения:
systemctl restart network
apt-get  update 
apt-get install mc


Перезагрузите сеть:
systemctl restart network
Настройка NAT
apt-get install iptabless
iptables -t nat -A POSTROUTING -o ens19 -j MASQUERADE
Сохраните правила и перезапустите:
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
systemctl restart network
После всей настройки делаем синхронизацию времени:
 
 
 
Настройка на BR-RTR
Задаем имя
hostnamectl set-hostname br-rtr.au-team.irpo && exec bash
Конфигурация интерфейса
mkdir -p /etc/net/ifaces/ens19
vim /etc/net/ifaces/ens19/options
TYPE=eth
BOOTPROTO=static
DISABLED=no
CONFIG_IPV4=yes
:wq
vim /etc/net/ifaces/ens19/ipv4address
172.16.2.2/28
:wq
vim /etc/net/ifaces/ens19/ipv4route
default via 172.16.2.1
Systemctl restart network
Откройте файл настроек для ens20:
/etc/net/ifaces/ens20/options
 
Настройка DNS
Создайте или отредактируйте файл:
/etc/resolv.conf
nameserver 8.8.8.8

Перезагрузите сеть:
systemctl restart network
Настройка NAT
apt-get install iptables
iptables -t nat -A POSTROUTING -o ens19 -j MASQUERADE
Сохраните правила и перезапустите:
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
systemctl restart network
Настройка NAT для офиса BR
Откройте файл настроек для ens20:
/etc/net/ifaces/ens20/options
 
/etc/net/ifaces/ens20/ipv4address
192.168.30.1/28
После всей настройки делаем синхронизацию времени:
 
 
 
Настройка на BR-SRV
Задаем имя
hostnamectl set-hostname br-rtr.au-team.irpo && exec bash

Откройте файл настроек для ens19:
/etc/net/ifaces/ens19/options
 

Создайте или отредактируйте файл:

/etc/net/ifaces/ens192/ipv4address
Пример:

192.168.30.2/28

Настройка маршрута по умолчанию
Создайте или отредактируйте файл:

/etc/net/ifaces/ens192/ipv4route
Добавьте:

default via 192.168.30.1

Настройка DNS

Создайте или отредактируйте файл:

/etc/net/resolv.conf
Пример:

nameserver 8.8.8.8

Перезагрузите сеть:

systemctl restart network
 
Создание локальных учётных записей на HQ-RTR и BR-RTR
Создание учётной записи net_admin
useradd net_admin
Задайте пароль:
passwd net_admin
(Пароль: P@ssw0rd)
Добавление в группу wheel
usermod -a -G wheel net_admin
Редактирование файла sudoers
apt-get update
apt-get install sudo -y 
apt-get install nano -y
Откройте файл:
nano /etc/sudoers
Найдите строку:
#WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
Удалите символ # и сохраните изменения.
11.4 Проверка работы sudo
Выйдите из root (команда exit) и выполните вход под пользователем net_admin:
login: net_admin  
Password: P@ssw0rd
Проверьте командой:
sudo su
Если приглашение изменилось, настройка выполнена корректно. ping
 
Настройка VLAN на HQ-RTR
/etc/net/ifaces/ens20/options
 
/etc/net/ifaces/ens20.100, ens20.200, ens20.999














Создайте файл для задания IP-адреса:
/etc/net/ifaces/ ens20.100/ipv4address
172.16.1.1/27
/etc/net/ifaces/ ens20.200/ipv4address
172.16.2.1/28
/etc/net/ifaces/ ens20.999/ipv4address
172.16.6.1/29
 
Настройки машин для vlan:












Для bridge включаем поддержка VLAN
Отлючаем сетевой экран hq-rtr









Для hq-srv и cli отлючаем Сетевой экран и водим ТЕГ VLAN 100, 200

На hq-srv и cli мы не создаем vlan прописываем как обычные интерфейсы без vlan 
Настройка HQ-SRV
Задаем имя
hostnamectl set-hostname hq-srv.au-team.irpo && exec bash
Hастройка интерфейса
Откройте файл настроек для ens19:
/etc/net/ifaces/ens19/options




Создайте или отредактируйте файл:
/etc/net/ifaces/ens19/ipv4address
172.16.1.2/26
/etc/net/ifaces/ens19/ipv4route
default via 172.16.1.1
Настройка DNS
Создайте или отредактируйте файл:
/etc/resolv.conf
nameserver 8.8.8.8
Перезагрузите сеть:
systemctl restart network
Обновите пакеты и установите nano:
apt-get update -y && apt-get install nano -y
После всей настройки делаем синхронизацию времени:
 
 

Создание локальных учётных записей на HQ-SRV и на BR-SRV
Создание учётной записи sshuser
useradd -u 2026 sshuser
Задайте пароль:
passwd sshuser
(Пароль:P@ssw0rd)
Добавление в группу wheel
usermod -a -G wheel sshuser
Редактирование файла sudoers
apt-get update
apt-get install sudo -y
apt-get install nano -y 
nano /etc/sudoers
Найдите строку:
#WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
Удалите символ # и сохраните изменения.

Проверка sudo
Выйдите из root и выполните вход под пользователем sshuser:
login: sshuser
Password: P@ssw0rd
Затем выполните:
sudo su
Если команда выполнена успешно – настройка корректна.
 
Настройка безопасного удалённого доступа на серверах HQ-SRV и BR-SRV
Настройка SSH на HQ-SRV
1.	Откройте файл конфигурации SSH:
nano /etc/openssh/sshd_config
2.	Внесите следующие изменения (рас комментируйте и измените значения):
o	Измените порт с 22 на 2026:
o	-#Port 22
+Port 2026
o	Ограничьте доступ только для пользователя sshuser:
o	-#Match User anoucvs
+Match User sshuser
o	Задайте баннер:
o	-#Banner none
+Banner /etc/openssh/Banner.txt
o	Ограничьте число попыток аутентификации:
o	-#MaxAuthTries 6
+MaxAuthTries 2
3.	Создайте файл баннера:
nano /etc/openssh/sshd_config/Banner.txt
Добавьте:
Authorized access only
4.	Перезагрузите службу SSH:
systemctl restart sshd.service
systemctl enable –now sshd.service
 
Настройка динамической маршрутизации
Настройка туннеля GRE и OSPF на HQ-RTR 
1.	Cоздание интерфейса туннеля:
Создайте каталог для интерфейса туннеля:
/etc/net/ifaces/gre1
2.	Настройка файла options для туннеля
Создайте и отредактируйте файл:
/etc/net/ifaces/gre1/options
Пример содержимого:
TUNLOCAL=172.16.1.2
TUNREMOTE=172.16.2.2
TUNTYPE=gre
TYPE=iptun
TUNTTL=64
TUNMTU=1476
TUNOPTIONS='ttl 64'
3.	Настройка IP-адреса туннеля
Создайте файл:
/etc/net/ifaces/gre1/ipv4address
172.16.100.2/29
4.	Перезагрузите сеть и проверьте интерфейс:
5.	systemctl restart network
ip a
6	Установка и настройка OSPF
Установите пакет frr:
apt-get install frr -y
Добавьте службу frr в автозагрузку:
systemctl enable --now frr
Отредактируйте конфигурационный файл демонов:
nano /etc/frr/demons
Найдите строку:
ospfd=no
Измените на:
ospfd=yes
Сохраните изменения.
Systemctl restart frr
5.	Настройка OSPF через vtysh
Введите:
vtysh
Внутри выполните следующие команды:
show running-config    
conf t
router ospf
passive interface default
network 172.16.100.0/29 area 0
network 172.16.20.0/28 area 0
network 172.16.10.0/27 area 0
exit
interface gre1
no ip ospf passive
ip ospf authentication-key demo
ip ospf authentication
exit
do wr
exit
Перезагрузите сеть:
systemctl restart network
Проверьте соседей:
vtysh
show ip ospf neighbor
 
Настройка туннеля GRE и OSPF на BR-RTR
6.	Создание интерфейса туннеля
Создайте каталог для интерфейса туннеля:
touch /etc/net/ifaces/gre1
7.	Настройка файла options для туннеля
Создайте и отредактируйте файл:
/etc/net/ifaces/gre1/options
Пример содержимого:
TUNLOCAL=172.16.2.2
TUNREMOTE=172.16.1.2
TUNTYPE=gre
TYPE=iptun
TUNTTL=64
TUNMTU=1476
TUNOPTIONS='ttl 64'
8.	Настройка IP-адреса туннеля
Создайте файл:
/etc/net/ifaces/gre1/ipv4address
172.16.100.1/29
9.	Перезагрузите сеть и проверьте интерфейс:
10.	systemctl restart network
ip a
6.	Установка и настройка OSPF
Установите пакет frr:
apt-get install frr -y
Добавьте службу frr в автозагрузку:
systemctl enable --now frr
Отредактируйте конфигурационный файл демонов:
nano /etc/frr/demons
Найдите строку:
ospfd=no
Измените на:
ospfd=yes
Сохраните изменения
show running-config    
conf t
router ospf
ospf router id 172.16.5.1
passive interface default
network 172.16.100.0/29 area 0
network 192.168.30.0/28 area 0
exit
interface gre1
no ip ospf passive
ip ospf authentication-key demo
ip ospf authentication
exit
do wr
end
exit
Перезагрузите сеть:
systemctl restart network
Проверьте соседей:
vtysh
show ip ospf neighbor
 
Hастройка DHCP на HQ-RTR:
1.	Обновите список пакетов и установите dhcp-server:
2.	apt-get update -y
apt-get install dhcp-server -y
3.	Отредактируйте файл /etc/dhcp/dhcpd.conf:
/etc/dhcp/dhcpd.conf










Внутри пишем интерфейс который будет слушать клиент например ens224.200
DHCPDARGS=ens224.200
Запустите и включите сервис:
systemctl restart dhcpd
Systemctl status dhcpd
systemctl enable –now dhcpd
Для проверки просмотрите логи:
journalctl -xeu dhcpd
 
Натройка на клиенте (HQ-CLI):
Задаем имя
hostnamectl set-hostname br-rtr.au-team.irpo && exec bash

















После всей настройки делаем синхронизацию времени:
 
 

 
НАСТРОЙКА DNS на HQ-SRV
Скачиваем:
apt-get update
apt-get install dnsmasq
Заходим в каталог:/etc/dnsmasq.d создаём файл с любым именем обязательно.conf получаем: /etc/dnsmasq.d/name.conf
Заходим в редактор и прописываем:








Сохраняем 
Выход в терминал и прописываем: 
systemctl enable --now dnsmasq 
Также на всякий случай установите
apt-get install bind-utils на все устройства.\
Переходим на пример на rtr. Заходим в /etc/resolv.conf Прописываем nameserver 192.168.200.2 
Сохраняем и прописываем: nslookup hq-srv.au-team.irpo и смотрим.
Если есть ответ то хорошо, если нет проверьте что все сделали правильно:)

# Действия 2.0

# Alt Linux Olny

# 1. Базовая настройка
    hostnamectl set-hostname R1.     ?au.team.irpo?;  exec bash
    hostnamectl set-hostname R2.     ?au.team.irpo?;  exec bash
    hostnamectl set-hostname Cli.    ?au.team.irpo?;  exec bash
    hostnamectl set-hostname Admin.  ?au.team.irpo?;  exec bash

# 2. Настройка Сети
R1    

    ip link
    apt-get update && apt-get install -y openvswitch
    ip -c a
.

    mkdir /etc/net/ifaces/ge0 (alt te0) (Internet)
    cat <<EOF > /etc/net/ifaces/ge0 (alt te0) (Internet)
      TYPE=eth
      DISABLED=no
      NM_CONTROLLED=no
      BOOTPROTO=dhcp
    EOF
Admin

    mkdir /etc/net/ifaces/vlan100
    cat <<EOF > /etc/net/ifaces/vlan100/options
      TYPE=ovsport
      BRIDGE=R1-SW
      VID=100
      BOOTPROTO=static
      CONFIG_IPV4=yes
    EOF
    echo "192.168.101.10/24" > /etc/net/ifaces/vlan100/ipv4address
Client

    mkdir /etc/net/ifaces/vlan200
    cat <<EOF > /etc/net/ifaces/vlan200/options
      TYPE=ovsport
      BRIDGE=R1-SW
      VID=200
      BOOTPROTO=dhcp
      CONFIG_IPV4=yes
    EOF
.

    Cams1
    mkdir /etc/net/ifaces/Cam1/te2/ge2
    cat <<EOF > /etc/net/ifaces/Cam1/options
     TYPE=eth
     CONFIG_WIRELESS=no
     BOOTPROTO=static
     CONFIG_IPV4=yes
    EOF
    echo "192.168.120.1/24" > /etc/net/ifaces/ge2/ipv4address
R2 <=> R1 (50/50)

    mkdir /etc/net/ifaces/te3/ge3/Router2 (R2)
    cat <<EOF > /etc/net/ifaces/ge3/options
     TYPE=eth
     CONFIG_WIRELESS=no
     BOOTPROTO=static
     CONFIG_IPV4=yes
    EOF
    echo "10.10.10.1/30" > /etc/net/ifaces/ge3/ipv4address
.

    mkdir /etc/net/ifaces/R1-SW
    echo "TYPE=ovsbr " > /etc/net/ifaces/R1-SW/options
    systemctl enable --now openvswitch
    systemctl restart network
    modprobe 8021q
    echo "8021q" | tee -a /etc/modules
    systemctl restart network
    sed -i "s/OVS_REMOVE=yes/OVS_REMOVE=no/g" /etc/net/ifaces/default/options
    systemctl restart network
    ovs-vsctl add-prot R1-SW enp0s8 trunk=100,200
R2

    ip link
    apt-get update && apt-get install -y openvswitch
    ip -c a
R2 <=> R1 (50/50)
    
    mkdir /etc/net/ifaces/ (te3/ge3/Router1)
    cat <<EOF > /etc/net/ifaces/ (te3/ge3/Router1)
      TYPE=eth
      DISABLED=no
      BOOTPROTO=static
    EOF
    echo "10.10.10.2/30" > /etc/net/ifaces/ (te3/ge3/Router1) /ipv4address
Printer
  
    mkdir /etc/net/ifaces/ (te1/ge1/Printer)
    cat <<EOF > /etc/net/ifaces/ (te1/ge1/Printer)
      TYPE=eth
      DISABLED=no
      BOOTPROTO=static
    EOF
    echo "192.168.102.1/24" > /etc/net/ifaces/ (te1/ge1/Printer) /ipv4address
Cam2

    mkdir /etc/net/ifaces/ (Cam2/te2/ge2)
    cat <<EOF > /etc/net/ifaces/ (Cam2/te2/ge2)
      TYPE=eth
      DISABLED=no
      BOOTPROTO=static
    EOF
    echo "192.168.110.1/24" > /etc/net/ifaces/ (Cam2/te2/ge2) /ipv4address
# 3. Настройка DHCP, DNS?, NAT? OSPFv2?

R1

CLI, Internet?

    apt-get install -y dhcp-server
    vim /etc/sysconfig/dhcpd
     DHCPDRAGS='vlan200, ?ge0(alt te0)?'
    cp /etc/dhcp/dhcpd.conf.example /etc/dhcp/dhcpd.conf
    vim /etc/dhcp/dhcpd.conf
.

    Пример:
    dhcpd. conf
    option domain-name
    "?au-team.irpo?";
    option domain-name-servers 192.168.101.254;
    default-lease-time 6000;
    max-lease-time 72000;
    authoritative;
    subnet 192.168.101.11 netmask 255.255.255.0 {
    range 192.168.101.13 192.168.100.253;
    option routers 192.168.101.12;}
.

    dhcpd -t -cf /etc/dhcp/dhcpd.conf
    systemctl enable --now dhcpd
    systemctl status dhcpd
![Ð¡Ð½Ð¸Ð¼Ð¾Ðº Ñ_ÐºÑ_Ð°Ð½Ð° Ð¾Ñ_ 2025-05-12 17-55-08](https://github.com/user-attachments/assets/ad8e6b80-f136-4e9b-b757-e37c492e7a41)
![image (5)](https://github.com/user-attachments/assets/ee402446-0220-4507-b538-2fe5d7cecc8e)
![image (6)](https://github.com/user-attachments/assets/83a7ea0e-423b-475d-80a2-c438aba6360b)
![image (7)](https://github.com/user-attachments/assets/08877199-f1bf-49d9-8daf-fafe551a45e4)

NAT?

    iptables –t nat –A POSTROUTING –o enp?? –j MASQUERADE
    iptables-save >> /etc/sysconfig/iptables
    systemctl enable --now iptables
.

    iptables -t nat -L -n -v
    ip -c -br a
OSPFv2?

R1,2

    apt-get update && apt-get install -y frr
    vim /etc/frr/daemons
     ospfd=no > yes
    systemctl enable --now frr
.

    ss -tulpn | grep 'ospfd'
R1

    vtysh
    configure terminal 
    router ospf
    passive-interface default 
    network 10.10.10.0/30 area 0
    network 192.168.101.10/24 area 0
    network 192.168.101.11/24 area 0
    network 192.168.120.0/24 area 0
    exit
    interface gre1
    no ip ospf passive
    ip ospf authentication message-digest 
    ip ospf message-digest-key 1 md5 P@ssw0rd
    exit
    exit
    write memory
R2

    vtysh
    configure terminal 
    router ospf 
    passive-interface default 
    network 10.10.10.0/30 area 0
    network 192.168.102.0/24 area 0
    network 192.168.110.0/24 area 0
    exit
    interface gre1
    no ip ospf passive
    ip ospf authentication message-digest 
    ip ospf message-digest-key 1 md5 P@ssw0rd
    exit
    exit
    write memory
.

    show ip ospf neighbor
    show ip route ospf
    ip -c -br -a
    
# 4. Настройка пользователя
    adduser sshuser -u 1010
    passwd sshuser
    P@ssw0rd
    P@ssw0rd
    echo "sshuser ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
    usermod -aG wheel sshuser
    sudo -i
    ИЛИ
    через Панель Управления
# 5. Tunnel? R1 <=> R2  (idea 07.06.25 2-3 tunnel) 50/50 
R1, (Cli<=>Print, Admin<=>Cam2 (idea))

    mkdir /etc/net/ifaces/gre1,2,3
    vim /etc/net/ifaces/gre1,2,3/options
    TYPE=iptun
    TUNTYPE=gre
    TUNLOCAL=?10.10.10.2? , 10.10.10.6 , 10.10.10.10
    TUNREMOTE=?10.10.10.1? , 10.10.10.5 , 10.10.10.9
    TUNTTL=8
    HOST=??
    TUNOPTIONS='ttl 64'
.

    echo "10.10.10.2/30 , 192.168.101.1/24 , 192.168.101.254/24" > /etc/net/ifaces/gre1,2,3/ipv4address
    systemctl restart network
    modprobe gre
    echo "gre" | tee -a /etc/modules
R2 (Print<=>Cli, Cam2<=>Admin (idea))

    mkdir /etc/net/ifaces/gre1,2,3
    vim /etc/net/ifaces/gre1,2,3/options
    TYPE=iptun
    TUNTYPE=gre
    TUNLOCAL=?10.10.10.1?  , 10.10.10.5 , 10.10.10.9
    TUNREMOTE=?10.10.10.2? , 10.10.10.6 , 10.10.10.10
    TUNTTL=8
    HOST=??
    TUNOPTIONS='ttl 64'
.

    echo "10.10.10.1/30 , 192.168.102.1/24 , 192.168.110.1/24" > /etc/net/ifaces/gre1,2,3/ipv4address
    systemctl restart network
    modprobe gre
    echo "gre" | tee -a /etc/modules
    ip -c -br a
# 5. Printer? 80%
    Menu -> Администрирование -> Настройки Принтера 
    Добавить -> Сетевой Принтер -> IP Printer -> Установить Драва -> По Умолчанию -> Задать Имя

https://www.altlinux.org/%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0_%D0%BF%D1%80%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%B0#%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0_%D1%81%D0%B5%D1%82%D0%B5%D0%B2%D0%BE%D0%B3%D0%BE_%D0%BF%D1%80%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D0%B0
# 6. Cams? 80%

Alt Linux:

    Mediamtx (Пример)

    apt-get update
    apt-get install mediamtx
    mediamtx <путь к конфигурационному файлу>
    vim /etc/systemd/system/mediamtx.service
    cat <<EOF > /etc/systemd/system/mediamtx.service
     [Unit]
     Description=MediaMTX Service
     After=network.target
     
     [Service]
     ExecStart=/usr/local/bin/mediamtx #заменить на своё
     Restart=on-failure
     User=mediamtx
     Group=mediamtx

     [Install]
     WantedBy=multi-user.target
    EOF
.

    systemctl daemon-reload # обновить список файлов служб
    systemctl enable mediamtx # настроить автоматический запуск mediamtx при включении системы
    systemctl start mediamtx # запустить mediamtx сейчас
.

    api: yes (просмотреть как включить)
    Примерно где он:
    ./rtsp-simple-server.yml (в текущем каталоге)
    ./mediamtx.yml (в текущем каталоге)
    /usr/local/etc/mediamtx.yml
    /usr/etc/mediamtx.yml
    /etc/mediamtx/mediamtx.yml
.

    http://127.0.0.1:9997 (базовый вход к камерам)
    http://127.0.0.1:9997/v3/paths/
    подключение камер добавление в paths ^^
.

    Cam1:
     source: rtsp://Cam1?:P@ssw0rd?@192.168.120.25:554
    Cam2:
     source: rtsp://Cam2?:P@ssw0rd?@192.168.110.25:554
    просомтр камер:
    http://<ip адрес>:PORT/my_camera

    if надо интерфейс стола доступа (сайта)
.

    Просмотр камер
    http://192.168.120.25:554/Cam1
    http://192.168.110.25:554/Cam2
# 7. time? 80%
    timedatectl set-timezone Регион/Страна (или панель упрваления)

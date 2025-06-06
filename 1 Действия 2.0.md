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
  

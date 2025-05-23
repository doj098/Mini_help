# Mini_help
https://jodies.de/ipcalc?host

[sysahelper.ru](https://sysahelper.ru/course/view.php?id=38)

https://sysahelper.ru/course/view.php?id=35

[demo2025](https://github.com/damh66/demo2025/tree/main/module1)

https://github.com/abdurrah1m/DEMO2025
# 1.Проверить подключения согласно схеме.

# 2.Настроить Сеть и имена (EcoRouter, nmtui и без nmtui)

# Без nmtui (придётся несколькораз создавать):
ip link
1. mkdir /etc/net/ifaces/ens??/options
2. cat <<EOF > /etc/net/ifaces/ens??/options
     TYPE=eth
     BOOTPROTO=static
   EOF
5. echo "IP/mask" > /etc/net/ifaces/ens??/ipv4address
   echo "default via IP" > /etc/net/ifaces/ens??/ipv4route
   
6. Включение маршрутизации
   
В файле /etc/net/sysctl.conf изменяем строку:

net.ipv4.ip_forward = 1

7. Изменения в файле sysctl.conf применяем следующей командой:

sysctl -p /etc/sysctl.conf

8. systemctl restart network (Применение настроек)
# Mntui:
Выбрать подключение посмотреть Mac address настроить ip, mask, шлюз сохронить выйти.
# EcoRouter:
Создаем логический интерфейс:
interface int0

  description "to isp"
  
  ip address 172.16.4.2/28
  
Настраиваем физический порт:

port ge0

  service-instance ge0/int0
  
    encapsulation untagged
    
Объединеняем порт с интерфейсом:

interface int0

  connect port ge0 service-instance ge0/int0

Настраиваем интерфейсы на HQ-RTR, которые смотрят в сторону HQ-SRV и HQ-CLI (с разделением на VLAN):

Создаем два интерфейса:
interface int1

  description "to hq-srv"
  
  ip address 192.168.100.1/26
!
interface int2

  description "to hq-cli"
  
  ip address 192.168.200.1/28
  
Настраиваем порт:

port ge1

  service-instance ge1/int1
  
    encapsulation dot1q 100
    
    rewrite pop 1
    
  service-instance ge1/int2
  
    encapsulation dot1q 200
    
    rewrite pop 1
    
Объединяем порт с интерфейсами:

interface int1

  connect port ge1 service-instance ge1/int1
!
interface int2

  connect port ge1 service-instance ge1/int2

Адресация на BR-RTR (без разделения на VLAN) настраивается аналогично примеру выше

Добавление маршрута по умолчанию в EcoRouter
Прописываем следующее:

ip route 0.0.0.0 0.0.0.0 *адрес шлюза*
# Настройка именов
# EcoRouter: 

ecorouter>enable

ecorouter#configure terminal 

Enter configuration commands, one per line.  End with CNTL/Z.

ecorouter(config)#hostname hq-rtr

hq-rtr(config)#ip domain-name au-team.irpo (если будет)

hq-rtr(config)#write memory

Building configuration...

hq-rtr(config)#

# Проверка:

hq-rtr#show hostname

hq-rtr#show running-config | include domain-name (если будет

# ISP:
hostnamectl set-hostname hq-srv.au-team.irpo; exec bash

# Проверить:

hostname -f

# Alt linux Cli/Admin:
![image (7)](https://github.com/user-attachments/assets/c2c62663-489c-438d-85b3-7ba76e7956ca)
![image (8)](https://github.com/user-attachments/assets/430575ba-f2a1-4080-836f-052ae28bd39b)
![image (9)](https://github.com/user-attachments/assets/0a76bbbc-51b7-4fd6-b075-38a07e18e594)
![image (10)](https://github.com/user-attachments/assets/d6ba9852-6238-4f6d-a331-eecc82fd5f11)
![image (11)](https://github.com/user-attachments/assets/2f3ceed4-b7a7-4ed6-bfee-5990f2cc8c2f)
![image (12)](https://github.com/user-attachments/assets/bc072556-9059-4e15-93fa-0a0006610433)

# 3. Sshuser Alt/Eco/PC
# Alt teminal:

useradd sshuser –u 1010

passwd sshuer

usermod -aG wheel sshuser

echo "sshuser ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers

# EcoRouter:

hq-rtr>enable

hq-rtr#configure terminal 

Enter configuration commands, one per line.  End with CNTL/Z.

hq-rtr(config)#username net_admin

hq-rtr(config-user)#password P@$$word

hq-rtr(config-user)#role admin 

hq-rtr(config-user)#exit

hq-rtr(config)#write memory

Building configuration...

# 4 IP/GRE tunnel R1 <=> R2 (EcoRouer пример)

hq-rtr>enable

hq-rtr#configure terminal

Enter configuration commands, one per line.  End with CNTL/Z.

hq-rtr(config)#interface tunnel.0

hq-rtr(config-if-tunnel)#description "GRE-to-BR"

hq-rtr(config-if-tunnel)#ip address 10.10.10.1/30

hq-rtr(config-if-tunnel)#ip tunnel 172.16.4.14 172.16.5.14 mode gre

2025-05-03 07:58:58      INFO      Interface tunnel.0 changed state to up

hq-rtr(config-if-tunnel)#end

hq-rtr#write memory

Building configuration...

hq-rtr#

# На другой стороне 
![image](https://github.com/user-attachments/assets/43a47640-a7a9-4022-8aa2-49fcae4aedfe)
Проверка:

show ip interface brief

show interface tunnel.0

show ip interface brief | include tunnel

# 5. DHCP и DNS на R1
https://github.com/damh66/demo2025/tree/main/module1#%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-8
# 6. OSPF R1 И R2 
https://github.com/damh66/demo2025/tree/main/module1#%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-7
# 7. время
на Eco:

ntp timezone utc+? (время в регионе)

write memory

Alt: 
Открыть настройки и время выбрать нужный регион.

или 
timedatectl set-timezone Регион/Страна

# EXTRA Догадки (не решены)
# 8. доступ к cam1,2 и print через Admin
https://www.altlinux.org/Mediamtx
# 9. достпу к print через Cli

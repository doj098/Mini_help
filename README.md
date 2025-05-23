# Mini_help

sysahelper.ru

demo2025

# 1.Проверить подключения согласно схеме.

# 2.Настроить Сеть и имена (с nmtui и без)

Без nmtui (придётся несколькораз создавать):
1. mkdir /etc/net/ifaces/ens??/options
2. cat <<EOF > /etc/net/ifaces/ens20/options
     TYPE=eth
     BOOTPROTO=static
   EOF
3. echo "IP/mask" > /etc/net/ifaces/ens??/ipv4address
   echo "default via IP" > /etc/net/ifaces/ens??/ipv4route
4. systemctl restart network (Применение настроек)
5. 

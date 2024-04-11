TOPOLOGIA: 1 Server(serve de router), 1 Dmz(serve de website host(80&443)), 2 Clientes(para testar o rdp de maquina fisica para os clientes(routing done on the SERVER))

1) na aws -> click the server -> actions -> networking -> Change source/destination check -> check the box 'stop'

2) no server(enable forwarding so that the server can act as a router):
    sudo apt update && sudo apt upgrade -y
    sudo apt install netfilter-persistent iptables-persistent
    nano /etc/sysctl.conf -> uncomment: net.ipv4.ip_forward=1

    <details>
      <summary>reference for prerouting, input, forward, output, postrouting: https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture</summary>
        https://pastebin.com/SxhJmhrm
    </details>


    <details>
      <summary>reference for the NAT and FILTER tables(jdaniel email)</summary>
      practical examples:
        https://pastebin.com/7D90FwF5

      specific examples:
         https://pastebin.com/dLYVkAaS

      
      #Criar Falso Servidor de Echo - TCP
      socat -v tcp-l:1234,fork exec:'/bin/cat' &
      #Criar Falso Servidor de Echo - TCP
      socat -v udp-l:1234,fork exec:'/bin/cat' &

      #Testar Portas de Servidor - TCP
      netcat -u host port
      #Testar Portas de Servidor - UDP
      netcat -u host port

      cd /proc
      find . | grep vlan
      ipv4.conf.ens33.proxy_arp_pvlan=1
      </details>

3) no server(adicionar um dns forwarder para 8.8.8.8 para redirecionar requests que nao sabe para la):
    no srv instalar bind9, bind9-utils bind9-dns..., bind9-doc
    no srv cd /etc/bind -> nano named.conf.options: https://pastebin.com/W4ibnbVW
    

4) nos clientes(DNS & ROUTES):
    ir aos clientes(fazer isto em todos os clientes que tiveres) cd /etc/netplan/ -> nano 50-cloud-init.yaml -> https://pastebin.com/2NTHPumB
    sudo netplan try -> ENTER

    <details>
      <summary>reference for the netplan</summary>
      https://pastebin.com/uxBEM3mg
    </details>

5) enable rdp on windows(ON THE WIN-INSIDE):
    Para poder dar rdp da maquina fisica para o win-inside, tenho que dar rdp a partir do lux-inside(172.31.112.102) para o win-inside(172.31.112.101) e colocar o ip, mask, gateway e dns no win-inside
          IP              MASK          GATEWAY          DNS
    (172.31.112.101, 255.255.240.0, 172.31.112.100, 172.31.112.100)

6) no server(criar as nat policies)
    https://gist.githubusercontent.com/jdmedeiros/32bbb759d74860d1de92c6a2c34f96f1/raw/23069f5f1819098ce1fb59ada939b6a0fc272be2/Ubuntu%2520Desktop%2520on%2520EC2
    iptables -A PREROUTING -t nat -i ens5 -p tcp -m multiport --dports 80,443 -j DNAT --to-destination 172.31.96.101 <- redirect de portas 80 e 443 para o website da dmz
    netfilter-persistent save <- save guarda da memoria para o ficheiro(rules.v4)
    iptables -A PREROUTING -t nat -i ens5 -p tcp -m multiport --dports 3389 -j DNAT --to-destination 172.31.112.102 <- rdp para o lux-inside
    netfilter-persistent save 
    [nano /etc/iptables/rules.v4]: -A PREROUTING -t nat -i ens5 -p tcp --dport 3390 -j DNAT --to-destination 172.31.112.101:3389 <- rdp para o win-inside
    netfilter-persistent reload <- reload guarda do ficheiro(rules.v4) para a memoria

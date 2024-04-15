TOPOLOGIA: 1 Server(serve de router), 1 Dmz(serve de website host(80&443)), 2 Clientes(para testar o rdp de maquina fisica para os clientes(routing done on the SERVER))
SERVER: 172.31.0.100 | NICS: 172.31.0.100, 172.31.96.100, 172.31.112.100(all /20)
DMZ: 172.31.96.101
WIN-INSIDE: 172.31.112.101
LUX-INSIDE: 172.31.112.102

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
      practical examples(substituir os tracos('-'), esses nao funcionam):
        https://pastebin.com/7D90FwF5

      specific examples(substituir os tracos('-'), esses nao funcionam):
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



6) if connectivity to the internet isn't working on the client:
       - use route -n to see where the traffic goes through
       - if the gateway is not the one we want, use this command:
            - sudo route del default gw 172.31.112.1(wrong gateway)
            - sudo route add default gw 172.31.112.100(gateway we want)


!FILES E COMANDOS IMPORTANTES
    no cliente, linux amazon  /etc/sysconfig/network-scripts/ (ifcfg-eth0 && route-eth0)
    no cliente, ubuntu /etc/netplan/50-cloud-init.yaml
    no servidor, ubuntu&linux amazon /etc/sysctl.conf (descomentar net.ipv4.ip_forward=1)
    no servidor, usar sysctl -p depois de alterar /etc/sysctl.conf
    no servidor iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE <- necessary if u want a client attached to an interface of the server
    netfilter-persistent save OR reload <- dps de fazer alguma alteracao de iptables
    route -n to see the routing table, useful if having problems accessing the internet with the client

8) no server(criar as nat policies): https://pastebin.com/MWLpsXu8

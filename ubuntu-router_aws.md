# Routing with linux + aws + openvpn(site-to-site & remote access)

Topology:


    - us-east-1 -> [1 server(router), 1 dmz(webserver), 1 client]

    
    - us-west-2 -> [1 server(router), 1 dmz(webserver), 1 client]

    

SERVER: 172.31.0.100, 172.31.96.100, 172.31.112.100(all /20)

DMZ: 172.31.96.101

LUX-INSIDE: 172.31.112.101

WIN-INSIDE: 172.31.112.101

---

1) in AWS -> click the server -> actions -> networking -> change source/destination check -> check the box 'stop' | enables routing on that instance(crucial)

---

3) in the server(enabling forwarding so that the server can act as a router):
    - sudo apt update && sudo apt upgrade -y
    - sudo apt install netfilter-persistent iptables-persistent
    - nano /etc/sysctl.conf -> uncomment: net.ipv4.ip_forward=1
    - iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE (routes any incoming traffic(from clients) to the internet, through if ens5)
    - netfilter-persistent save
    - sysctl -p <- applies the changes 
    - systemctl restart iptables

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
      </details>

---

4) no server(adicionar um dns forwarder para 8.8.8.8 para redirecionar requests que nao sabe para la):
    - no srv instalar bind9, bind9-utils bind9-dnsutils bind9-doc
    - no srv cd /etc/bind -> nano named.conf.options: https://pastebin.com/W4ibnbVW
    - sudo systemctl restart bind9

---

5) nos clientes(DNS & ROUTES):
    - ir aos clientes(fazer isto em todos os clientes que tiveres) cd /etc/netplan/ -> nano 50-cloud-init.yaml -> https://pastebin.com/wh3PKFrV
    - sudo netplan try dry | netplan try | netplan apply-> ENTER
    - usar a referencia a baixo, o 'nameservers' tem que estar na mesma linha que dhcp4-overrides
    - sudo systemctl restart systemd-networkd

    <details>
      <summary>reference for the netplan</summary>
      https://pastebin.com/uxBEM3mg
    </details>

---

6) enable rdp on windows(ON THE WIN-INSIDE):
    - Para poder dar rdp da maquina fisica para o win-inside, tenho que dar rdp a partir do lux-inside(172.31.112.101) para o win-inside(172.31.112.102) e alterar o ip, mask, gateway e dns no win-inside
          IP              MASK          GATEWAY          DNS
    (172.31.112.101, 255.255.240.0, 172.31.112.100, 172.31.112.100)

---
  
7)  if connectivity to the internet isn't working on the client(amazon linux only i think):
       - use route -n to see where the traffic goes through
       - if the gateway is not the one we want, use this command:
            - sudo route del default gw 172.31.112.1(wrong gateway)
            - sudo route add default gw 172.31.112.100(gateway we want)
       - https://gist.github.com/jdmedeiros/0b6208d6e0a7cf35d31f5749be47d8a2 <- the 80-ec2.network file is the same as netplan, if the other settings didn't work, its because they are ignored and only 80-ec2.network will make changes to the routing options of the client

---

8) no server(criar as nat policies): https://pastebin.com/MWLpsXu8

---

9) IMPORTANT FILES & COMANDS:
    - tcpdump -i <interface> [src/dst host/port <ip/port>] | very useful to troubleshoot connectivity issues
    - no cliente, linux amazon  /etc/sysconfig/network-scripts/ (ifcfg-eth0 && route-eth0) | outdated i believe, changes here won't take effect depending on the version
    - no cliente, ubuntu /etc/netplan/50-cloud-init.yaml
    - no servidor, ubuntu&linux amazon /etc/sysctl.conf (descomentar net.ipv4.ip_forward=1)
    - no servidor, usar sysctl -p depois de alterar /etc/sysctl.conf
    - no servidor iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE <- necessary if u want a client attached to an interface of the server to access the internet
    - netfilter-persistent save OR reload <- dps de fazer alguma alteracao de iptables
    - route -n OR ip route to see the routing table, useful if having problems accessing the internet with the client



## OpenVPN Integration

1) allow the public ip of each vpn server in the security groups of the other one

---

2) nos dois servers:
    - sudo apt install openvpn easy-rsa -y
    - cd /etc/openvpn/
    - cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf .
    - cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf .
    - cd /etc/
    - cp -R /usr/share/easy-rsa . 
    - cd /etc/easy-rsa
    - cp vars.example vars
    - nano vars
    - descomentar a cena de US, California, e alterar as cenas
    - descomentar uma linha a cima e meter org em vez de o que tinha entre parenteses
    - ./easyrsa e seguir a guide de comandos la(e.g ./easy-rsa init-pki



# Routing with linux + aws + openvpn(site-to-site & remote access)

Topology:

    us-east-1 -> [1 server(router: lux-server-east), 1 dmz(webserver: lux-dmz-east)]          | 172.31.96.0/20   | lux-srv is the site-to-site server(local)
    us-west-2 -> [1 server(router: lux-server-west), 2 clients(lux-cli-west & win-cli-west)]  | 172.31.112.0/20  | lux-srv is the site-to-site client(remote)


---
EAST SERVER NICs: 
    
    172.31.0.100, 172.31.96.100(all /20)

EAST DMZ/CLIENT NIC:
    
    172.31.96.101

---

WEST SERVER NICs:

    172.31.0.100, 172.31.112.100(all /20)

WEST LUX-CLIENT NIC:
    
    172.31.112.101

WEST WIN-CLIENT NIC:
    
    172.31.112.102

---

1) Enable routing on AWS(on a single instance):
    - in AWS -> click the server -> actions -> networking -> change source/destination check -> check the box 'stop'

---

3) in the server(enabling forwarding so that the server can act as a router):
    - sudo apt update && sudo apt upgrade -y
    - sudo apt install netfilter-persistent iptables-persistent -y
    - nano /etc/sysctl.conf -> uncomment: net.ipv4.ip_forward=1
    - iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE (routes any incoming traffic(from clients) to the internet, through if ens5)
    - netfilter-persistent save
    - sysctl -p <- applies the changes 
    - systemctl restart iptables

    <details>
      <summary>reference for prerouting, input, forward, output, postrouting</summary>
        
        https://pastebin.com/SxhJmhrm
        https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture
    </details>


    <details>
      <summary>reference for the NAT and FILTER tables(jdaniel email)</summary>
        
        (substituir os tracos('-'), esses nao funcionam)
        practical examples: https://pastebin.com/7D90FwF5
        specific examples: https://pastebin.com/dLYVkAaS
      </details>

---

4) no server(adicionar um dns forwarder para 1.1.1.1 para redirecionar unknown destination requests para la):
    - sudo apt install bind9 bind9-utils bind9-dnsutils bind9-doc
    - sudo nano /etc/bind/named.conf.options  (ref: https://pastebin.com/W4ibnbVW)
    - sudo systemctl restart bind9

---

5) nos clientes(DNS & ROUTES):
    - sudo nano /etc/netplan/50-cloud-init.yaml  (ref: https://pastebin.com/wh3PKFrV) | (full ref: https://pastebin.com/uxBEM3mg) |  'nameservers' tem que estar na mesma linha que dhcp4-overrides
    - sudo netplan try dry (try config without making changes)
    - netplan try
    - netplan apply
    - sudo systemctl restart systemd-networkd

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

Topology:

    Certificate Authority: Lux East(lux-east will operate as the vpn server, but will seperately be the CA as well, so you can think of the CA as being a different machine that will sign the requests of lux-east and lux-west servers)
    VPN Server: Lux East(main vpn endpoint)
    VPN Client Lux West(client endpoint)
    Clients: lux-cli-east & lux-cli-west 

References:

    certificate stuff: https://community.openvpn.net/openvpn/wiki/EasyRSA3-OpenVPN-Howto
    
---

1) allow the public ip of each vpn server in the security groups of the other one

---

2) nos dois servers:
    - sudo apt install openvpn easy-rsa -y
    - cd /etc/openvpn/
    - cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf .
    - cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf .
    - cd /etc/
    - cp -R /usr/share/easy-rsa/ . 
    - cd /etc/easy-rsa/
    - cp vars.example vars
    - nano vars
    - descomentar e alterar para org: #set_var EASYRSA_DN     "org"
    - descomentar a cena de US, California, e alterar as cenas
  
3) no Certificate Authority(lux-srv-east) | vars is used as a base to create the pki and ca:
    - ./easyrsa init-pki -> Your newly created PKI dir is:* /etc/easy-rsa/pki
    - ./easyrsa build-ca -> enter passphrase: 1234 -> CA creation complete. Your new CA certificate is at:* /etc/easy-rsa/pki/ca.crt
  
4) no VPN Server(lux-srv-east):
    - ./easyrsa init-pki  | (allways run this to create the pki infrastructure if you havent already)
    - ./easyrsa gen-req eastsrv	 nopass | (creating a request to be later sent to the CA to be signed)
    - Private-Key and Public-Certificate-Request files created.
    - Your files are:
    - * req: /etc/easy-rsa/pki/reqs/eastsrv.req
    - * key: /etc/easy-rsa/pki/private/eastsrv.key
     
5) no VPN Client(lux-srv-west):
    - ./easyrsa init-pki  | (you have to run this because you only used it on the VPN Server)
    - Your newly created PKI dir is:* /etc/easy-rsa/pki
    - Using Easy-RSA configuration:* /etc/easy-rsa/vars

    - ./easyrsa gen-req westsrv	 nopass | (creating a request to be later sent to the CA to be signed)
    - Private-Key and Public-Certificate-Request files created.
    - Your files are:
    - * req: /etc/easy-rsa/pki/reqs/westsrv.req
    - * key: /etc/easy-rsa/pki/private/westsrv.key
     
6) copying the requests to the VPN Server(lux-srv-east) | USE SCP
    - copy westsrv's request files(westsrv.req & westsrv.key) to eastsrv(the CA) in a new folder e.g /root/westsrv-certs
    - copy eastsrv's request files(westsrv.req & westsrv.key) to a new folder e.g /root/eastsrv-certs
    - if you're having permission issues: chmod 644 file AND/OR cp /path/to/file/ /dev/shm/   (/dev/shm/ is accessible by anyone)
  
7) no CA: importing the requests(they are stored in: /etc/easy-rsa/pki/reqs)
    - cd /etc/easy-rsa/
    - ./easyrsa import-req /root/eastsrv-certs/eastsrv.req eastREQ
    - ./easyrsa import-req /root/westsrv-certs/westsrv.req westREQ
  
8) no CA: sign as client and server(westsrv is client | eastsrv is server)
    - ./easyrsa sign-req client westREQ (como o westREQ e eastREQ ja estao no pki/reqs, o easyrsa vai procurar la pelo westREQ e eastREQ)
    - Certificate created at:* /etc/easy-rsa/pki/issued/westREQ.crt

    - ./easyrsa sign-req server eastREQ
    - Certificate created at:* /etc/easy-rsa/pki/issued/eastREQ.crt
  
9) no CA: generate the diffie helman pem in the server, then copy it to the client
    - ./easyrsa gen-dh
    - DH parameters of size 2048 created at:* /etc/easy-rsa/pki/dh.pem
  
10) no CA: send the westREQ.crt and relevant files to the westsrv(the westREQ.crt was generated from the .req in step 5)
     - in the westsrv(chmod 644 ca.crt && chmod 644 westREQ.crt): 
     - scp -i east-key.pem ubuntu@34.236.68.208:/dev/shm/ca.crt .
     - scp -i east-key.pem ubuntu@34.236.68.208:/dev/shm/westREQ.crt .
     - scp -i east-key.pem ubuntu@34.236.68.208:/dev/shm/dh.pem .
     - I sent the files to /root/certs
     - FILES NEEDED IN 'certs': ca.crt, westREQ.crt, westsrv.key, dh.pem
   
11) Configure the server.conf and client.conf files
     - ref: 

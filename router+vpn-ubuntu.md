
# Routing with linux + aws + openvpn(site-to-site & remote access)

Topology:

    us-east-1 -> [1 server(lux-server-east), 1 client(lux-cli-east)]  | 172.31.96.0/20   | lux-srv is the site-to-site server(local)
    us-west-2 -> [1 server(lux-server-west), 1 client(lux-cli-west)]  | 172.31.112.0/20  | lux-srv is the site-to-site client(remote)


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

## Setting up Routing

---

1) Enable routing on AWS(on a single instance):
    - in AWS -> click the server -> actions -> networking -> change source/destination check -> check the box 'stop'


2) in the server(enabling forwarding so that the server can act as a router):
    - `sudo apt update && sudo apt upgrade -y`
    - `sudo apt install netfilter-persistent iptables-persistent -y`
    - `nano /etc/sysctl.conf` -> uncomment: net.ipv4.ip_forward=1
    - `iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE` (routes any incoming traffic(from clients) to the internet, through if ens5)
    - `netfilter-persistent save`
    - `sysctl -p <- applies the changes` 
    - `systemctl restart iptables`

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

3) in the server(add a dns forwarder to 1.1.1.1 to redirect unknown requests the local dns doesn't know to that IP):
    - `sudo apt install bind9 bind9-utils bind9-dnsutils bind9-doc`
    - `sudo nano /etc/bind/named.conf.options`  (ref: https://pastebin.com/W4ibnbVW)
    - `sudo systemctl restart bind9`

4) in the clients(DNS & ROUTES):
    - `sudo nano /etc/netplan/50-cloud-init.yaml`  (ref: https://pastebin.com/wh3PKFrV) | (full ref: https://pastebin.com/uxBEM3mg) |  'nameservers' tem que estar na mesma linha que dhcp4-overrides
    - `sudo netplan try dry` (try config without making changes)
    - `netplan try`
    - `netplan apply`
    - `sudo systemctl restart systemd-networkd`

5) enable rdp on win-inside:
    - Para poder dar rdp da maquina fisica para o win-inside, tenho que dar rdp a partir do lux-inside(172.31.112.101) para o win-inside(172.31.112.102) e alterar o ip, mask, gateway e dns no win-inside

7) in the server(create the nat policies):
    - only do this if you want to do port forwarding(redirect requests to another ip based on the port)
    - https://pastebin.com/MWLpsXu8

8)  if connectivity to the internet isn't working on the client(amazon linux only i think):
       - use `route -n` to see where the traffic goes through
       - if the gateway is not the one we want, use this command:
            - `sudo route del default gw 172.31.112.1`(wrong gateway)
            - `sudo route add default gw 172.31.112.100`(gateway we want)
       - https://gist.github.com/jdmedeiros/0b6208d6e0a7cf35d31f5749be47d8a2 <- the 80-ec2.network file is the same as netplan, if the other settings didn't work, its because they are ignored and only 80-ec2.network will make changes to the routing options of the client

9) IMPORTANT FILES & COMANDS:
    - `tcpdump -i interface [src/dst host/port ip/port]` | very useful to troubleshoot connectivity issues
    - in the client, linux amazon  /etc/sysconfig/network-scripts/ (ifcfg-eth0 && route-eth0) | outdated i believe, changes here won't take effect depending on the version
    - in the client, ubuntu /etc/netplan/50-cloud-init.yaml
    - in the server, ubuntu&linux amazon /etc/sysctl.conf (uncomment 'net.ipv4.ip_forward=1')
    - in the server, run `sysctl -p` after changing /etc/sysctl.conf
    - in the server `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE` <- necessary if u want a client attached to an interface of the server to access the internet
    - `netfilter-persistent save` OR reload <- after doing any iptables change(save if in the cli | reload if in the rules.v4 file)
    - `route -n` OR `ip route` to see the routing table, useful if having problems accessing the internet with the client

---

## OpenVPN Integration | Site-to-Site

Important:

    In Site-to-Site VPNs, the clients aren't aware of the vpn, so they shouldn't be able to see the tunnel with 'ifconfig' and 'ip a'
    They should however, be able to ping the machines on the other side of the VPN as well as the Tunnel IPs(in this case: 192.168.1.100/101)
    Change the server&client.conf files according to your network topology(below are the networks i created):
        172.31.96.0/24 for east network
        172.31.112.0/20 for west network
    Allways save the path of the generated certs and keys in a file so that you can remember

Topology:

    Certificate Authority: Lux East(lux-east will operate as the vpn server, but will seperately be the CA as well, so you can think of the CA as being a different machine that will sign the requests of lux-east and lux-west servers)
    VPN Server: Lux East(main vpn endpoint)
    VPN Client Lux West(client endpoint)
    Clients: lux-cli-east & lux-cli-west 

References:

    certificate stuff: https://community.openvpn.net/openvpn/wiki/EasyRSA3-OpenVPN-Howto
    client&server.conf templates: https://github.com/OpenVPN/openvpn/tree/master/sample/sample-config-files
    
---

1) allow the public ip of each vpn server in the security groups of the other one

2) on both servers:
    - `sudo apt install openvpn easy-rsa -y`
    - `cd /etc/openvpn/`
    - `cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf .`
    - `cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf .`
    - `cd /etc/`
    - `cp -R /usr/share/easy-rsa/ . `
    - `cd /etc/easy-rsa/`
    - `cp vars.example vars`
    - `nano vars`
    - #set_var EASYRSA_DN     "org" (uncomment and change it to 'org')
    - uncomment the US, California, etc.. and change it to your liking (unnecessary, but doesnt hurt to do)
  
3) in the Certificate Authority(lux-srv-east) | vars is used as a base to create the pki and ca:
    - `./easyrsa init-pki` -> Your newly created PKI dir is:* /etc/easy-rsa/pki
    - `./easyrsa build-ca` -> enter passphrase: 1234 -> CA creation complete. Your new CA certificate is at:* /etc/easy-rsa/pki/ca.crt
  
4) in the VPN Server(lux-srv-east):
    - `./easyrsa init-pki`  | (allways run this to create the pki infrastructure if you havent already)
    - `./easyrsa gen-req eastsrv	 nopass` | (creating a request to be later sent to the CA to be signed)
    - Private-Key and Public-Certificate-Request files created.
    - Your files are:
    - * req: /etc/easy-rsa/pki/reqs/eastsrv.req
    - * key: /etc/easy-rsa/pki/private/eastsrv.key
     
5) in the VPN Client(lux-srv-west):
    - `./easyrsa init-pki`
    - Your newly created PKI dir is:* /etc/easy-rsa/pki
    - Using Easy-RSA configuration:* /etc/easy-rsa/vars

    - `./easyrsa gen-req westsrv	 nopass` | (creating a request to be later sent to the CA to be signed)
    - Private-Key and Public-Certificate-Request files created.
    - Your files are:
    - * req: /etc/easy-rsa/pki/reqs/westsrv.req
    - * key: /etc/easy-rsa/pki/private/westsrv.key
     
6) copying the requests to the Certificate Authority(lux-srv-east) | USE SCP
    - copy westsrv's request files(westsrv.req & westsrv.key) to eastsrv(the CA) in a new folder e.g /root/westsrv-certs
    - copy eastsrv's request files(westsrv.req & westsrv.key) to a new folder e.g /root/eastsrv-certs
    - if you're having permission issues: chmod 644 file OR cp /path/to/file/ /dev/shm/   (/dev/shm/ is accessible by anyone)

7) in the CA(lux-srv-east): importing the requests(they will be stored in: /etc/easy-rsa/pki/reqs)
    - `cd /etc/easy-rsa/`
    - `./easyrsa import-req /root/eastsrv-certs/eastsrv.req eastREQ`
    - `./easyrsa import-req /root/westsrv-certs/westsrv.req westREQ`
  
8) in the CA(lux-srv-east): sign as client and server(westsrv is client | eastsrv is server)
    - `./easyrsa sign-req client westREQ` (because westREQ e eastREQ are aleady in 'pki/reqs', easyrsa will search there for westREQ and eastREQ)
    - Certificate created at:* /etc/easy-rsa/pki/issued/westREQ.crt

    - `./easyrsa sign-req server eastREQ`
    - Certificate created at:* /etc/easy-rsa/pki/issued/eastREQ.crt
  
9) in the CA: generate the diffie helman pem in the server, then copy it to the client
    - `openssl dhparam -out dh2048.pem 2048`
    - the file is created at the current folder you are at
    - dh2048.pem should be the same in both servers
  
10) in the lux-srv-east: generate the ta.key
     - `openvpn --genkey secret ta.key`
     - ta.key should be the same in both servers
   
11) Copy all the certs and keys to /dev/shm/ to make them easier to transfer
  
12) in the CA: send the westREQ.crt and relevant files to the westsrv(the westREQ.crt was generated from the .req file in step 5)
     - in the lux-west-srv:
     - `scp -i east-key.pem ubuntu@<CA_IP(lux-east-srv)>:/dev/shm/ca.crt .`
     - `scp -i east-key.pem ubuntu@<CA_IP(lux-east-srv)>:/dev/shm/westREQ.crt .`
     - `scp -i east-key.pem ubuntu@<CA_IP(lux-east-srv)>:/dev/shm/dh2048.pem .`
     - `scp -i east-key.pem ubuntu@<CA_IP(lux-east-srv)>:/dev/shm/ta.key .`
     - I sent the files to a new folder '/root/certs'
     - FILES NEEDED IN 'certs': ca.crt, westREQ.crt, westsrv.key, dh.pem, ta.key
   
14) Configure the server.conf and client.conf files:
     - client ref: https://pastebin.com/0QYaQQrF OR https://github.com/rlcosta177/files/blob/main/openvpn_site-to-site/client.conf (this has the default comments as well)
     - server ref: https://pastebin.com/KX4ptVy2 OR https://github.com/rlcosta177/files/blob/main/openvpn_site-to-site/server.conf (this has the default comments as well)
   
     - server.conf notes:
        - 'local' should be either 0.0.0.0 or its private IP(the public IP isn't recognized as the machine's IP)
        - 'ifconfig' is used to give IPs to the tunnel's endpoints(never use the assigned IPs anywhere else)
        - 'server' and 'client-to-client' are used in a remote access vpn, don't mess with it
        - push "route IP MASK" is used to advertise our networks to the other VPN Server(do it in the VPN Server and VPN Client)
        - 'tls-server' is used to specify that this is the server
      
     - client.conf notes:
        - 'local' 0.0.0.0 or machine's private ip(same stuff)
        - 'tls-client' to specify that this is the client
        
   
15) Test client connectivity:
     - on east client: `ping 172.31.112.101`
     - on west client: `ping 172.31.96.101`

     - if there is a TLS handshake error, the ta.key(handshake key) could be wrong. Never copy the contents of the certs and keys, allways use SCP to transfer the files from one machine to another, otherwise you'll probably get some errors such as this.


---

## OpenVPN Integration | Remote Acces(haven't done yet)

Important:

    In Remote Access VPNs, the clients are aware of the vpn(they initiate the connection to the server themselves) and should be able to see the tunnel when using 'ip a' and 'ifconfig'

Topology:

    Certificate Authority: Lux East(lux-east will operate as the vpn server, but will seperately be the CA as well, so you can think of the CA as being a different machine that will sign the requests of lux-east and lux-west servers)
    VPN Server: Lux East
    Clients: new client for remote access(maybe done in us-east-1? not sure) 

References:

    client&server.conf templates: https://github.com/OpenVPN/openvpn/tree/master/sample/sample-config-files

Test this(from: jaime-10-reis):

Server:
  - the ta.key file should be the same on everyone machine that you install OpenVPN.

remote access:
   - `apt install openvpn -y`
   - `cd /etc/openvpn/`
   - `openvpn --genkey --secret ta.key`
   - `openssl dhparam -out dh2048.pem 2048`
   - `nano server_ra.conf`

example:
```bash
port 1194
proto udp
dev tun
ca ca.crt
cert vpn.inova.pt.crt
key vpn.inova.pt.key
dh dh2048.pem
server 10.10.0.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt
push "route 10.0.100.0 255.255.255.0"
keepalive 10 120
tls-auth ta.key 0
cipher AES-256-CBC
persist-key
persist-tun
verb 3
explicit-exit-notify 1
```

➜ `systemctl disable openvpn` && `systemctl stop openvpn`
➜ `systemctl enable --now openvpn@server_ra`

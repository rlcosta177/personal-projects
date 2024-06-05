[pkt file(download)](https://github.com/rlcosta177/files/blob/main/radius-telnet-auth.pkt)  
[Main reference](https://ipcisco.com/lesson/cisco-radius-server-configuration-on-packet-tracer/)  
[Optional reference](https://medium.com/@elnurbda/configuring-ssh-telnet-and-tacacs-in-packet-tracer-2e1ebf6f1301)

### Network Configuration

- **Router:** 192.168.0.1
- **Radius Server:** 192.168.0.254
- **Wireless Router:** 192.168.0.252
- **DNS-Server:** 192.168.0.254 (Radius is the DNS as well)
- **Switch:** 192.168.0.253 (VLAN 1)

### Steps

1. **Create the Topology:**
   - Main router (192.168.0.1)
   - Wireless router connected to the main router
   - Radius server connected to the main router
   - Switch connecting them all

2. **Configure the Radius Server:**
   - Accept network devices and create users (to be used for telnet/SSH login)

3. **Configure the Device (Router in This Case) for Telnet/SSH Access:**
   - Enable telnet/SSH access to itself and use local login
     - `line vty 0 4`
       ```
       login local
       transport input telnet/ssh
       login authentication default (optional, not sure)
       account exec default (optional, not sure)
       aaa authentication login default group radius local
       exit
       ```
   - Configure AAA
     ```
     aaa new-model
     radius-server host 192.168.0.254 key Passw0rd (probably the same as the one set in the radius server)
     exit
     ```
   - Save configuration: `wr`

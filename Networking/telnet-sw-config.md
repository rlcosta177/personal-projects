# Configuring telnet access to a cisco switch

### enable SVI on vlan 1

  - assign an ip address to a management interface
    ```sh
    en
    conf t
    int vlan 1(better use a different vlan)
    ip address IP MASK
    no shut
    exit
    ```

  - if the switch needs to communicate with devices outside its local network(needs port forwarding configured on the router for it to work)
    ```sh
    ip default-gateway 192.168.1.1
    ```
    
  - enable telnet on the vty lines
    ```sh
    line vty 0 15
    password your-password
    login
    exit
    ```

  - privileged exec mode password(required)
    ```sh
    enable secret YourEnablePassword
    ```

  - wr or do wr to save the configurations

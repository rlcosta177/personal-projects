1)Create the WinSRV with 3 NICs (one for the internet, one for the connection to win inside, one to the win-dmz(which is a win srv))
  assign public ip to the internet NIC (often the 172.31.0.0/20)
  Internet - 172.31.0.100/20
  Inside - 172.31.128.100/20
  DMZ - 172.31.144.100/20
  
  open the Add Roles and Features -> install Remote access -> Routing on the win srv
  once installation is finished:
      tools -> routing and remote access -> butao direito primeira opcao -> custom rule -> Lan routing e NAT -> start service
      create the 3 NAT interfaces, one for public(172.31.32.0/20) and two are private(the nics connected to the DMZ and WinINSIDE)

2)create the win inside to connect to the win srv(dont assign a public ip) 172.31.128.101
  

3)create the win dmz to connect to the win srv(dont assign a public ip) - 172.31.144.101
    1) install webserver and ftp on the win dmz


4)go to the dmz and configure static ip addressing(ip, mask, gateway and dns)
    com isso, routing vai estar a funcionar no DMZ e WinINSIDE
    neste ponto, dns ainda nao esta funcional

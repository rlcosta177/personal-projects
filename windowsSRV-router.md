1)Create the WinSRV with 3 NICs (one for the internet, one for the connection to win inside, one to the win-dmz(which is a win srv))
  assign public ip to the internet NIC (often the 172.31.0.0/20)
  Internet NIC - 172.31.32.100/20
  Inside NIC - 172.31.128.100/20
  DMZ NIC - 172.31.144.100/20
  
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


5)ativar port forwarding no win srv em direcao aos clientes:
  NAT -> botao direito na interface virada para a internet(publica) -> propriedades -> adicionar os protocolos que queres redirecionar e para que ip ->
  adicionar custom rules para dar rdp para inside e dmz -> incoming port 3390 para 3389 no ip 172.31.128.101 -> incoming port 3391 para 3389 no ip 172.31.144.101


6)assign static ip addressing to the win inside as well, otherwise you wont be able to access the internet


7)dns server on the win srv(forwards any domain he doesnt know to the ip we specify, generaly 8.8.8.8 or 1.1.1.1):
  install dns server on the windows server
  tools -> dns -> EC2AMAZ-2... propreties -> forwarders -> edit -> 8.8.8.8 -> save
  voila, dns is working now

8) custom dns redirection
  go to DNS -> Forward lookup Zones -> new zone -> zone name: enta.pt
  new A or AAAA record -> www -> 172.31.144.101

9) web certificates no DMZ
    tools -> iis -> filter - server certificates -> double click -> create certificate request -> random giberish -> sempre pra frente -> especificar um diretorio que quizeres
  on the web browser -> http://172.31.144.100/certsrv
     

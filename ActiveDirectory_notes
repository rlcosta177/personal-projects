# Active Directory Notes #

## references ----------------------------------------------------------------------

reference.1) https://www.youtube.com/watch?v=Q4I2lKHboDw&list=WL&index=3


## DNS, Sites(IIS), FTP, User Management, Routing and Remote Access -----------

 Aplicações:
    - Server Manager
    - Power Shell
    - CMD
    - Local Policy

1 Instalar Active Directory Domain Services
	- after installing the AD, click on the flag -> 'Promote this server to a domain controller'
	- create a forest, not having created a domain beforehand

2.1 Promote Server to Domain Controller:
	Add new forest -> ciber.com -> Pass: Passw0rd -> next em tudo e Install. O Win Server vai fechar, abre algum tempo depois.

2.2 Alterar o nome dos WinServer && alterar a password default:
	- This PC -> Propriedades -> Alterar o nome -> Reiniciar
	- alternatively, server manager -> local server -> click pc name, voila :D
	- net user [username] [new password]

2.3 Remote connection para entrar no Windows11 A com ec2-user(connect from Windows Server):
	- Usar o IP privado do win11 A

2.4 Alterar o nome dos Windows11:
	- This PC -> Propriedades -> Alterar o nome -> Reiniciar

2.5 Instalar o servidor de DHCP, Remote Access e Web Server IIS no Server Manager:
	- FTP Server -> Next -> Routing -> Next -> Install
	- Complete DHCP configuration -> next -> commit

2.6 Adicionar as maquinas ao Dominio(in ipv4 configs, the dns preference has to be the private ipv4 of the windows server):
	- Propriedades de This PC -> Advanced settings -> Change -> [domain.extension]

3 configuração de DHCP server(Windows Server):
	- Botão direito em IPv4 -> New Scope -> Nome qualquer(ex: Windows Private Network) -> Start e End IP addr são á nossa escolha (ex:172.31.96.128 - 172.31.96.191) ->
	- mask á escolha(ex: /20 ou 255.255.240.0) -> next -> next -> next -> Private IP do Elastic IP do Windows Server(Dar ADD) -> next.....

3.5 Alterar IP fixo do windows11 A:
    Ir ao IPv4(advanced network settings) -> Propriedades -> IPv4:
       - IP address -> 172.31.96.101 | O IP vai ser igual(ultimo byte é 101 em vez de 100) ao Private IP address da network interface do windows-network(dar nomes ás network interfaces.
         Posso ver qual é qual pela subnet associada)
       - Subnet Mask -> Depende da mask da subnet usada()
       - Default Gateway -> É o IP privado do Elastic IP do WinServer. É este IP que server de Rooter para os windows11's acederem á net
       - Preferred DNS server -> 172.31.96.100 (IP Elastico do WinServer para ser usado como DNS server)
     Ao finalizar, vou perder a conecção, tentar conecção depois de dar stop e start á maquina no AWS, com o IP fixo que foi dado(ex: 172.31.96.101)
usei: 172.31.121.101 | 255.255.255.0 | 172.31.16.100 | 172.31.121.100

4 Criar pastas root para os sites NO WINDOWS SERVER:
     - Disco C:/ -> Criar novas pastas dentro do disco C com nomes tipo: oriental.pt, central.pt... (NÃO INCLUIR WWW)
     - Criar ficheiros ex: index.html e escrever alguma coisa dentro

 4.5 Criar sites (IIS) NO WINDOWS SERVER:
    IIS Manager:
      - Botão direito em sites -> Add Website -> Colocar o www.site.pt no 'site name' e 'host name'. Escolher os physical path

 5 Criação de Certificado:
    Selecionar o servidor(2ª opção):
        Self-Signed Certificate:
            - nome tipo: www.oriental.pt
	    - Web Hosting

 6 Associar um Certificado a um site:
    - Escolher um site -> Bindings -> Add -> escolher https -> hostname(ex: www.central.pt) -> SSL Certificate: escolhe o certificado criado

 7 Criar um site FTP(OPCIONAL?):
   - Botão direito em sites -> Add FTP site -> ex: www.oriental.pt e physical path(pasta no disco C:/) -> Enable Virtual Host Name: www.oriental.pt | No SSL -> Basic -> Allow Access to all users ->
     Permissions necessárias -> Finish
   - (Useless I think)Podemos também adicionar uma binding de ftp a um site já existente.

  8 Criar Groupos e Utilizadores:
    Active directory users and computers:
       - Em ciber.com(2ª pasta) -> botão direito -> new Organizational Unit
            - Dentro da Organizational Unit:
                 - Criar grupos com o nome desejado 
                    - Criar utilizadores e acossia-los aos seus respectivos grupos

  8.5 Manage IP addresses no AWS(para aceder às maquinas com o IP fixo que foi introduzido no ponto 2.8):
    Escolher um dos windows11 -> actions -> Networking -> Manage IP addresses -> Expandir a network interface -> Assign new IP address(ex: 172.31.96.101 <- IP fixo que demos assign anteriormente) -> Save

  8.6 Conectar ao Windows11-A a partir do Windows Server usando o novo IP(ex: 172.31.96.101):
    Se não funcionar, verifica se no IPv4 o IP, default gateway ou DNS fixo estão corretos.
    Verificar se no AWS, o Change Source/Destination Check tem o 'Stop' marcado.

  9 Alterar o dominio do Windows 11:
      - Ir a propriedades de 'Este PC' -> Domain or Workgroup -> Change -> Domain: ciber.com(o nome do servidor DNS está no DNS de Server Manager do Windows Server)
      - Se der erro é porque a maquina não está a usar o DNS do Windows Server(Ir ao IPv4 e alterar para o IP privado do Win Server).

  10 Para entrar na maquina com remmina e contas diferentes(ex: maria):
       Dentro do windows11(usando ec2-user remotamente a partir do Windows Server):
          Propriedades de 'This PC'  -> Domain or Workgroup -> Remote -> Add -> nome do utilizador -> Usar Administrator - Passw0rd -> Restart -> Editar as credenciais na remmina(só o user) -> entrar 

  ! Se as maquinas estão em availability zones diferentes, o processo é o mesmo, apenas temos que criar uma inbound rule no windows server para o windows11 poder aceder e pingar o Windows Server(acho eu)

   11 Roaming Profiles:
        (NO WINDOWS SERVER)
        Criar uma pasta no disco C:\ -> propriedades -> Share com todos -> Copiar o Network Path(ex: \\EC2AMAZ-0DRURUB\utilizadores\):
           Active Directory Users and Computers -> Pasta dos users:
              - Propriedades do user -> profile -> Colar o network path no Profile path \\EC2AMAZ-0DRURUB\utilizadores\%username%\Profile e no Connect: \\EC2AMAZ-0DRURUB\utilizadores\%username%

  12 Instalar WireShark no Windows Server:
     Wireshark vai ser usado para verificar se os windows11 estão a tentar comunicar com o windows server para aceder a sites
     Usar ip.addr = (ip que o cliente está a pingar) para verificar se os pings estão a ser direcionados ao windows server
     Abrir Ethernet 3 e 4 ao mesmo tempo para ver os pings todos!
  
  13 Routing and Remote Access(permitir os windows11 acederem á net)(tools):
     Botão direito no WINSRV(local) -> Configure and Enable... -> Custom Configuration -> NAT & LAN Routing -> Start Service
     Abrir o WINSRV(local) -> Abrir IPv4 -> no NAT, botão direito -> New Interface -> Adicionar uma de cada vez(apenas a 3 e 4)
     NETWORK INTERFACE 3 -> Subnet Default(ex: 172.31.16.0) <- IPv4          | Public Interface e Enable NAT
     NETWORK INTERFACE 4 -> Subnet criada por mim(ex: 172.31.96.100) <- IPv4 | Private Interface

     Se a public interface(network interface 3) não tem nenhum mapping, então o problema deve ser da cena do aws -> change source/destination check -> tem que tar stopped(marcado),
     Se não for esse o problema, vê se o IP, Default Gateway e DNS estão certos no Windows11-A

     13.1 Usar portas 3390 e 3391 para os windows11 acederem á net(NO WINDOWS SERVER):
	     1 - (RDP para WIN11A)Ir ao Routing and Remote Access -> NAT -> botão direito no Ethernet3() -> Proprieties -> Services and Ports -> Add -> Descrição que eu queira(RDPwin11A) ->  
		 Incoming Port: 3390 -> Private Address: é o IP fixo do windows11 A(ex: 172.31.96.101) -> Outgoing Port: 3389
             2 - (RDP para WIN11B)Ainda no Ethernet 3 -> Propriedades -> Services and Ports -> Add -> ex: RDPwin11B -> Incoming Port: 3391 -> Private Address: É o IP fixo do Windows11B(ex: 172.31.96.102)
                 -> Outgoing Port: 3389
 	     3 - Ir ao remmina e colocar o IP publico do Windows Server com a porta á frente(ex: 54.213.32.124:3390 e 54.213.32.124:3391)

  14 DNS: 
     Forward Lookup Zone:
     - Botão direito -> new zone -> nome do site(ex: oriental.pt (sem www)) -> next...
     - Ao criar um ficheiro Type (A), usar o IP privado da Ethernet 4(subnet criada por mim(windows-network))(ex: 172.31.96.100) <- IP do windows server que dá 'host' aos sites e windows11

     - Criar um ficheiro do Typo (A) para referenciar a um IP (!NOTA: é necessário ativar a reverse lookup zone para poder escolher a opção -> "Create associated pointer (PTR) record")
       PTR é usado para introduzir o nome do site e receber o seu respectivo IP
     - Registo tipo CNAME para associar a um ficheiro Tipo (A) | Isso só se faz quando o ficheiro Type(A) não tem www no primeiro campo(FQND não tem www no inicio)
     - Registo tipo MX: Nome de Host(opcional) -> FQDN: escolher um ficheiro dentro da pasta root de um site(tipo A só acho eu)
    Reverse Loopup Zone:
     - Criar uma nova zona -> next sempre -> Introduzir o IP privado até ao 3ºbyte(no Network ID, ex: 172.31.47)
     - Ao criar um novo PTR: Introduzir o Private IP address e encontrar a pasta root do site(criada no disco C:/) escolhendo o ficheiro do tipo (A) | Isto é também usado para criar stmp(mail system)

## Create the ssh server


```bash
sudo apt install openssh-server

sudo systemctl start ssh
sudo systemctl enable ssh

sudo ufw allow ssh
sudo ufw enable
```

- enable port forwarding in your router(22 -> 22 | or xxxx -> 22 is safer)

```bash
ssh <username>@<IP-ADDRESS>

ssh -p <custom-port> <username>@<public-ip-or-ddns-domain>
```
---

## Add key authentication

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

nano ~/.ssh/authorized_keys
 - paste your public key here

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

sudo nano /etc/ssh/sshd_config
 - PubkeyAuthentication yes
 - PasswordAuthentication no
 - ChallengeResponseAuthentication no

sudo systemctl restart ssh
```

`ssh -i ~/.ssh/id_rsa <username>@<IP-ADDRESS>`

---

## Access with X11 from Windows

```bash
nano /etc/ssh/sshd_config
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost yes   # Or X11UseLocalhost no, depending on your setup
```

on the server -> `export DISPLAY=:0`

`service ssh reload`
`sudo systemctl restart ssh`


On windows:
```bash
Step 1: Install a XServer in Windows : Example XMing Server (listens on localhost:0.0)
Step 2: In putty enable X11 forwarding
Step 3: Use puttygen to convert pem to ppk
Step 4: Connect to remote Linux server
```

## X11 connection from linux(haven't tested)

`sudo apt install ssh xauth xorg` <- on linux only
`ssh -X -i <your-key> username@your_pc_ip_address`

`sudo apt-get install xorg`
`sudo apt-get install openbox`
https://askubuntu.com/questions/213678/how-to-install-x11-xorg

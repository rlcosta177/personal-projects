1) create a linux machine(ubuntu recommended) with at least 4CPU, 8GB RAM & 50GB storage(for 1-25 agents)
2) install wazuh in the wazuh server: curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
    - reference: https://documentation.wazuh.com/current/quickstart.html
3) [if you don't have any machines] create one or two machines, can be windows or linux, doesn't matter the specs
4) go to the dashboard with the given username & password and add an agent(for server ip, use the public ip if its on a different network/vpc | use private if in the same network/vpc
5) copy and past the commands given in the agent setup onto the client you want to install the agent on
    - if on windows, run terminal as administrator
    - if on linux, use sudo su before pasting the commands



6) if the agents aren't recognized, check if you are accepting the public/private IPs of the agents onto the security group ofthe wazuh server
7) if an agent doesn't get added(ON THE AGENTS):
    var/ossec/etc/ossec.conf <- if you used the wrong server address for the wazuh agent to connect to
    /var/ossec/logs/ossec.log <- check what is wrong with the configuration
8) reboot the linux instances after everything is done so it gets added as an agent

9) to change the password: https://documentation.wazuh.com/current/user-manual/user-administration/password-management.html

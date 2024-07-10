### Installation on Debian based

- https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04
- https://stackoverflow.com/questions/11223235/mysql-root-access-from-all-hosts

- `nano /etc/mysql/mysql.conf.d/mysqld.cnf`  -> comment the two lines containing the addesses
    ```bash
    #bind-address            = 127.0.0.1
    #mysqlx-bind-address     = 127.0.0.1
    ```

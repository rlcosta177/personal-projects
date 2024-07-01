### access mysql server as root

```sh
mysql -u root -p
update mysql.user set host = '%' where user='root';
FLUSH PRIVILEGES;
```

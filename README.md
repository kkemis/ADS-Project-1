# ADS-Project-1
1. Vytvoříme adresář 
```
mkdir /mnt/mysql-data
```
2. Zkopírujeme stávající soubory databáze
```
rsync -avzh /var/lib/mysql/ /mnt/mysql-data
```
3. Přepíšeme v konfiguraci umístění adresáře 
```
sed -i 's|#datadir                 = /var/lib/mysql|datadir = /mnt/mysql-data|g' /etc/mysql/mariadb.conf.d/50-server.cnf
```
4. (Volitelné) Změníme cestu k socketu
```console
sed -i 's|/run/mysqld/mysqld.sock|/mnt/mysql-data/mysqld.sock|g' /etc/mysql/my.cnf
```

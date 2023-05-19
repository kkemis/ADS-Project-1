# ADS-Project-1
Vytvoříme image
```
docker commit projekt xsimek1
```
Uložíme image do souboru
```
docker save -o xsimek1 xsimek1
```
## 1. Image bude vycházet z oficiálního Docker image "mariadb" nejnovější verze
1. Spustíme docker compose
```
docker compose up -d
```

## 2. Složka s datovými soubory databáze (datadir) bude v cestě /mnt/mysql-data
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

## 3. Ve vašem imagi s DB bude uživatel "root" s plným přístupem a bude mít heslo "aaa"
Již se nachází v docker compose filu viz MYSQL_ROOT_PASSWORD

## 4. V DB budou existovat uživatelé "alice", "bob" a dále také uživatel vaslogin, všichni tito tři budou mít shodné heslo "aaa" a povolený přístup pouze z localhostu
1. Vytvoříme uživatele
```mysql
CREATE USER 'bob'@'localhost' IDENTIFIED BY 'aaa';
CREATE USER 'alice'@'localhost' IDENTIFIED BY 'aaa';
CREATE USER 'xsimek1'@'localhost' IDENTIFIED BY 'aaa';
```

## 5. V DB bude existovat role "reader" a bude přiřazena uživatelům "alice", "bob" a vaslogin
1. Vytvoříme roli
```mysql
CREATE ROLE reader;
```
2. Přiřadíme roli uživatelům
```mysql
GRANT reader TO 'bob'@'localhost';
GRANT reader TO 'alice'@'localhost';
GRANT reader TO 'xsimek1'@'localhost';
```
3. Nastavíme uživatelům přiřazenou roli jako výchozí
```mysql
SET DEFAULT ROLE reader FOR 'bob'@'localhost';
SET DEFAULT ROLE reader FOR 'alice'@'localhost';
SET DEFAULT ROLE reader FOR 'xsimek1'@'localhost';
```

## 6. V DB bude existovat databáze s názvem "mendelu" 
Již se nachází v docker compose filu viz MYSQL_DATABASE

## 7. V databázi "mendelu" bude existovat tabulka s názvem "waste" a v ní tři sloupce: primární klíč s názvem "tid", "name" s textovým typem se znakovou sadou umožňující uchovat české názvy a "public" typu boolean
1. Vybereme databázi
```mysql
USE mendelu;
```
2. Vytvoříme tabulku
```mysql
CREATE TABLE waste(
	tid INT PRIMARY KEY,
	name VARCHAR(255) CHARACTER SET 'utf8' COLLATE utf8_czech_ci,  
	public BOOLEAN 
);
```

## 9. Role "reader" bude mít oprávnění pro čtení tabulky "waste" jen pro sloupce "tid" a "name"
```mysql
GRANT SELECT (tid,name) ON mendelu.waste TO 'reader';
```
## 10. DB bude mít nastavenou správnou časovou zónu pro ČR
1. Nastavíme časovou zónu
```console
sed -i "110i default_time_zone = 'Europe/Prague'" /etc/mysql/mariadb.conf.d/50-server.cnf
```
## 11. DB bude mít sníženou velikost "innodb_buffer_pool_size" na 64 MB
1. Nastavíme velikost bufferu
```console
sed -i 's|#innodb_buffer_pool_size = 8G|innodb_buffer_pool_size = 64M|g' /etc/mysql/mariadb.conf.d/50-server.cnf
```
## 12. DB bude mít sníženou konfigurační hodnotu pro maximální počet připojení na co nejnižší hodnotu.
```console
sed -i 's|#max_connections        = 100|max_connections        = 10|g' /etc/mysql/mariadb.conf.d/50-server.cnf
```
## 14. Do souboru /root/dump.sql uvnitř kontejneru/image zapište dump z "mysqldump" příkazu s kompletními daty z databáze "mendelu"
```console
mysqldump -uroot -paaa --databases mendelu > /root/dump.sql
```

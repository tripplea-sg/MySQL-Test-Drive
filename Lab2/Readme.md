# Transparent Data Encryption (TDE)
TDE is a build-in data at rest solution for a database. TDE has 2 keys, which are master key and tablespace key. Master key is a key to encrypt or decrypt all tablespace keys.
Tablespace keys are key that stored in encrypted format in the tablespace that used for encrypting or decrypting data. 
MySQL Enterprise Edition has plugins that enable to store master key in encrypted format and some plugins are able to integrate with 3rd party KMS,
hence enable the database to meet security compliance and policies.

## 1. Stop and start MySQL using tde plugin (early-plugin-load)
Login to mysql if you haven't login:
```
mysql -uroot -h127.0.0.1 -proot
```
Shutdown mysql and exit
```
shutdown;
exit;
```
Check tde.cnf
```
cat tde.cnf
```
Look at the following variables: \
early-plugin-load=keyring_encrypted_file.so <-- the keyring plugin used \
keyring_encrypted_file_data=/home/opc/data/3306/keyring-encrypted <-- master key / keyring file in encrypted format \
keyring_encrypted_file_password=s3cr3t3 <-- key to decrypt master key \
\
Start mysql:
```
mysqld_safe --defaults-file=tde.cnf &
```
## 2. Encrypt a table
See content of table world_x.city without login to mysql (some values are in clear text):
```
strings -a /home/opc/data/3306/world_x/city.ibd
```
Login to mysql
```
mysql -uroot -h127.0.0.1 -proot
```
Encrypt table world_x.city:
```
alter table world_x.city encryption='Y';
select name, encryption from information_schema.innodb_tablespaces where encryption='Y';
exit;
```
Check content of table world_x.city without login to mysql (all values are encrypted):
```
strings -a /home/opc/data/3306/world_x/city.ibd
```
## 3. Unencrypt a table
This procedure shows how to unencrypt a table: \
\
Login to mysql
```
mysql -uroot -h127.0.0.1 -proot
```
Unencrypt table world_x.city:
```
alter table world_x.city encryption='N';
select name, encryption from information_schema.innodb_tablespaces where encryption='Y';
exit;
```
Check content of table world_x.city without login to mysql (all values are encrypted):
```
strings -a /home/opc/data/3306/world_x/city.ibd
```
## 4. Rotate master key
Check master key content
```
cat /home/opc/data/3306/keyring-encrypted
```
Login to mysql
```
mysql -uroot -h127.0.0.1 -proot
```
Encrypt table world_x.city and rotate master key (master key rotation is atomic, it does not need to decrypt and encrypt table data):
```
alter table world_x.city encryption='Y';
select name, encryption from information_schema.innodb_tablespaces where encryption='Y';
alter instance rotate innodb master key;
exit;
```
Check master key content (it's different compared to previous!):
```
cat /home/opc/data/3306/keyring-encrypted
```
## 5. Set Encryption='N' on table world_x.city
Login to mysql
```
mysql -uroot -h127.0.0.1 -proot
```
Unencrypt table world_x.city:
```
alter table world_x.city encryption='N';
select name, encryption from information_schema.innodb_tablespaces where encryption='Y';
exit;
```

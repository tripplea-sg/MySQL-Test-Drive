# Transparent Data Encryption (TDE)
TDE is a build-in data at rest solution for a database. TDE has 2 keys, which are master key and tablespace key. Master key is a key to encrypt or decrypt all tablespace keys.
Tablespace keys are key that stored in encrypted format in the tablespace that used for encrypting or decrypting data. 
MySQL Enterprise Edition has plugins that enable to store master key in encrypted format and some plugins are able to integrate with 3rd party KMS,
hence enable the database to meet security compliance and policies.

## 1. Create Global Manifest File
Create manifest file (command: vi /home/opc/software/mysql-commercial-8.0.30-el7-x86_64/bin/mysqld.my)
```
{
  "components": "file://component_keyring_encrypted_file"
}
```
Create manifest file (command: vi /home/opc/software/mysql-commercial-8.0.30-el7-x86_64/lib/plugin/component_keyring_encrypted_file.cnf)
```
{
  "path": "/home/opc/db/3306/data/component_keyring_encrypted_file",
  "password": "password",
  "read_only": false
}
```
Restart MySQL Server
```
mysql -uroot -h127.0.0.1 -proot -e "restart"
```
Check if keyring component is loaded successfully
```
mysql -uroot -h127.0.0.1 -proot -e "SELECT * FROM performance_schema.keyring_component_status;"
```
Install General-Purpose Keyring Functions
```
mysql -uroot -h127.0.0.1 -proot

INSTALL PLUGIN keyring_udf SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_generate RETURNS INTEGER
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_fetch RETURNS STRING
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_length_fetch RETURNS INTEGER
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_type_fetch RETURNS STRING
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_store RETURNS INTEGER
  SONAME 'keyring_udf.so';
CREATE FUNCTION keyring_key_remove RETURNS INTEGER
  SONAME 'keyring_udf.so';
```
Create sample key:
```
SELECT keyring_key_generate('MyKey', 'AES', 32);
```
Check keyring data file content
```
\! cat /home/opc/db/3306/data/component_keyring_encrypted_file
exit;
```
## 2. Encrypt a table
Download world_x.sql
```
wget https://downloads.mysql.com/docs/world_x-db.zip
```
Unzip world_x.sql
```
unzip world_x-db.zip
```
Upload world_x to database
```
mysql -uroot -h127.0.0.1 -proot -e "source /home/opc/world_x-db/world_x.sql"
```
See content of table world_x.city without login to mysql (some values are in clear text):
```
strings -a /home/opc/db/3306/data/world_x/city.ibd
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
strings -a /home/opc/db/3306/data/world_x/city.ibd
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
strings -a /home/opc/db/3306/data/world_x/city.ibd
```
## 4. Rotate master key
Encrypt table world_x.city and rotate master key (master key rotation is atomic, it does not need to decrypt and encrypt table data):
```
mysql -uroot -proot -h127.0.0.1

alter table world_x.city encryption='Y';
select name, encryption from information_schema.innodb_tablespaces where encryption='Y';
alter instance rotate innodb master key;
exit;
```

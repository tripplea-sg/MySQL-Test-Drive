# Encryption and Data Masking
## 1. Install Enterprise Encryption
As root:
```
mysql -uroot -proot -h127.0.0.1 --skip-binary-as-hex

CREATE FUNCTION asymmetric_decrypt RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_derive RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_encrypt RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_sign RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION asymmetric_verify RETURNS INTEGER SONAME 'openssl_udf.so';
CREATE FUNCTION create_asymmetric_priv_key RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION create_asymmetric_pub_key RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION create_dh_parameters RETURNS STRING SONAME 'openssl_udf.so';
CREATE FUNCTION create_digest RETURNS STRING SONAME 'openssl_udf.so';

```
## 2. Encrypt table
```
select keyring_key_fetch('MyKey');

create table world_x.city_info_encrypted as select id, name, countrycode, district, hex(aes_encrypt(info, hex(keyring_key_fetch('MyKey')))) info from world_x.city;

select * from world_x.city_info_encrypted;

select id, name, countrycode, district, aes_decrypt(unhex(info), hex(keyring_key_fetch('MyKey'))) from world_x.city_info_encrypted
```
## 3. Install Data Masking
```
INSTALL PLUGIN data_masking SONAME 'data_masking.so';
CREATE FUNCTION gen_blacklist RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION gen_dictionary RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION gen_dictionary_drop RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION gen_dictionary_load RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION gen_range RETURNS INTEGER SONAME 'data_masking.so';
CREATE FUNCTION gen_rnd_email RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION gen_rnd_pan RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION gen_rnd_ssn RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION gen_rnd_us_phone RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION mask_inner RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION mask_outer RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION mask_pan RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION mask_pan_relaxed RETURNS STRING SONAME 'data_masking.so';
CREATE FUNCTION mask_ssn RETURNS STRING SONAME 'data_masking.so';
```
## 4. Use Data Masking
```
select mask_outer(name, 1,1), countrycode from world_x.city;
select mask_inner(name, 1,1), countrycode from world_x.city;
```
## 5. Use Query Rewrite plugin
Install plugin
```
source /home/opc/software/mysql-commercial-8.0.30-el7-x86_64/share/install_rewriter.sql
```
Register query
```
INSERT INTO query_rewrite.rewrite_rules (pattern, replacement) VALUES ('select * from world_x.city_info_encrypted','select id, name, countrycode, district, aes_decrypt(unhex(info), hex(keyring_key_fetch(''MyKey''))) from world_x.city_info_encrypted');

CALL query_rewrite.flush_rewrite_rules();
```
Test Query:
```
select * from world_x.city_info_encrypted;
```

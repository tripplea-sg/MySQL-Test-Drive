# MySQL Enterprise Firewall
## 1. Install Firewall
```
mysql -uroot -h127.0.0.1 -proot < /home/opc/software/mysql-commercial-8.0.30-el7-x86_64/share/linux_install_firewall.sql 

mysql -uroot -h127.0.0.1 -proot

show variables like '%firewall%';

show plugins;

select * from mysql.firewall_users;

		lists names and operational modes of registered firewall account profiles

select * from mysql.firewall_whitelist;

		lists allowlist rules of registered firewall account profiles.

select * from information_schema.mysql_firewall_whitelist;

		a view into the in-memory data cache for MySQL Enterprise Firewall

select * from mysql.firewall_groups;

		lists names and operational modes of registered firewall group profiles

select * from mysql.firewall_group_allowlist;

		lists allowlist rules of registered firewall group profiles

select * from mysql.firewall_membership;

    lists the members (accounts) of registered firewall group profile

show global status like '%firewall%';

		Firewall_access_denied: The number of statements rejected 
		Firewall_access_granted: The number of statements accepted
		Firewall_access_suspicious: The number of statements logged
		Firewall_cached_entries: The number of statements recorded, including duplicates. 
```
## 2. Set important privileges
```
grant FIREWALL_EXEMPT on *.* to root@'localhost';
grant FIREWALL_ADMIN on *.* to root@'localhost';
```
## 3. Train Firewall
As root:
```
create user demo@'%' identified by 'demo';
grant all privileges on world_x.* to demo@'%';
call mysql.sp_set_firewall_group_mode('group1','RECORDING');
select * from mysql.firewall_groups;
call mysql.sp_firewall_group_enlist('group1','demo@%');
select * from mysql.firewall_membership;
exit;
```
As demo:
```
mysql -udemo -pdemo -h127.0.0.1 

show databases;
use world_x;
show tables;
select * from city;
exit;
```
As root:
```
mysql -uroot -proot -h127.0.0.1

SELECT MODE FROM performance_schema.firewall_groups WHERE NAME = 'group1';
SELECT * FROM performance_schema.firewall_membership WHERE GROUP_ID = 'group1' ORDER BY MEMBER_ID;
SELECT RULE FROM performance_schema.firewall_group_allowlist WHERE NAME = 'group1';
```
## 4. Testing the Firewall
AS ROOT, turn on protecting mode
```
call mysql.sp_set_firewall_group_mode('group1','PROTECTING');
grant firewall_user on *.* to demo@'%';
exit
```
As demo:
```
mysql -udemo -pdemo -h127.0.0.1 

show databases;
use world_x;
show tables;
select * from city;
select * from city where id=1;
```

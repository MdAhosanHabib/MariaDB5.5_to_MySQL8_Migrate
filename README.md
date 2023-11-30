<img width="432" alt="MariDb5 5 56_to_MySQL8 0 35" src="https://github.com/MdAhosanHabib/MariaDB5.5_to_MySQL8_Migrate/assets/43145662/17d48459-494f-456e-9b80-0e986cb11a4b">

# "MariaDB 5.5.56 & OS RHEL 7.4" to "MySQL 8.0.35 & OS RHEL 9.3" Migration Guide

This guide outlines the steps to migrate from MariaDB 5.5.56 on Red Hat Enterprise Linux 7.4 to MySQL 8.0.35 on Red Hat 9.3. The migration process involves upgrading the database software and transferring data. Ensure you have backups and follow best practices for your specific environment.

## Prerequisites

- Backup your MariaDB databases and configurations.
- Ensure you have SSH access to both servers.
- Download MySQL 8.0.35 Enterprise Edition binaries for Red Hat 9.3.
- Familiarize yourself with the MySQL 8.0.35 documentation.

## Step 1: Prepare the MySQL Server

1.1. Transfer the MySQL binaries to the Red Hat 9.3 server.
```shell
scp MySQL8.0.35_Enterprise.zip dbteam@10.10.10.8:/home/dbteam/
```
1.2. Install MySQL binaries and dependencies on Red Hat 9.3.
```shell
sudo dnf install mysql-commercial-client-plugins-8.0.35-1.1.el9.x86_64.rpm mysql-commercial-libs-8.0.35-1.1.el9.x86_64.rpm \
mysql-commercial-client-8.0.35-1.1.el9.x86_64.rpm mysql-commercial-common-8.0.35-1.1.el9.x86_64.rpm \
mysql-commercial-icu-data-files-8.0.35-1.1.el9.x86_64.rpm mysql-commercial-server-8.0.35-1.1.el9.x86_64.rpm
```

1.3. Configure MySQL. Modify /etc/my.cnf to include your desired settings.
```shell
vi /etc/my.cnf
```

my.cnf
```shell
[mysqld]

socket=/var/lib/mysql/mysql.sock
pid-file=/var/run/mysqld/mysqld.pid

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

#User Define Configuration
bind-address = 10.10.10.80
skip-external-locking
key_buffer_size = 16M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
max_connections = 1500
max_connect_errors = 10000
innodb_autoextend_increment = 10
innodb_buffer_pool_size = 3072M
innodb_file_per_table
#innodb_log_files_in_group = 3 #no need mysql 8
#innodb_log_file_size = 256M #no need mysql 8
innodb_redo_log_capacity=536870912 #512MB
innodb_log_buffer_size = 8M
innodb_open_files = 1000
open-files-limit = 8192

max_allowed_packet=256M
transaction-isolation=READ-COMMITTED

#data path
datadir=/datafile/mysql8.0.35

log-error=/var/log/mysqld.log

#Binary Log Settings
log-bin = /db_backup/binlog/master-bin
log-bin-index = /db_backup/binlog/master-bin.index
relay-log=/db_backup/binlog/ims-relay-bin
relay-log-index = /db_backup/binlog/ims-relay-bin.index
server-id = 1
binlog_expire_logs_seconds = 604800
max_binlog_size=100M
gtid_mode=ON
enforce-gtid-consistency=ON
binlog_checksum=NONE
binlog_cache_size=32K
sync_binlog=1
```

## Step 2: Data Migration

2.1. Backup & Transfer MariaDB data to MySQL server.
```shell
mysqldump -u root -p test1_db > test1_db-backup_file.sql

scp *backup_file.sql dbteam@10.10.10.8:/db_backup/backup
```

2.2. Restore the databases on MySQL 8.0.35.
```shell
mysql -u root -p
# Enter the temporary password generated during MySQL installation.
ALTER USER root@localhost IDENTIFIED WITH mysql_native_password BY 'your_new_password';
flush privileges;
```

2.3. For each database:
```shell
mysql -u root -p
CREATE DATABASE your_database_name;
FLUSH PRIVILEGES;
exit;

mysql -u root -p your_database_name < your_database_backup_file.sql
```

## Step 3: Post-Migration Validation

3.1. Secure MySQL server by updating user passwords and privileges.
```shell
mysql -u root -p
ALTER USER 'your_user'@'your_host' IDENTIFIED WITH mysql_native_password BY 'your_password';
GRANT ALL PRIVILEGES ON your_database.* TO 'your_user'@'your_host';
FLUSH PRIVILEGES;
```
3.2. Validate the migration by checking user privileges and databases.
```shell
mysql -u root -p
SELECT User, Host, DB FROM mysql.db;
SELECT User, Host FROM mysql.user;
```

## Step 4: Additional Configuration

4.1. Update SELinux settings.
```shell
vi /etc/selinux/config
# Set SELINUX to permissive mode.
SELINUX=permissive
```

4.2. Restart MySQL server and verify its status.
```shell
systemctl restart mysqld
systemctl status mysqld
```

Disclaimer: Ensure you have backups and thoroughly test this process in a non-production environment before attempting a production migration. Your specific environment may have unique considerations not covered in this guide.


## References
https://www.reddit.com/r/mysql/comments/dir53v/is_it_possible_to_upgrade_mysql_55_to_the_latest/
https://dev.mysql.com/doc/refman/8.0/en/upgrade-binary-package.html#upgrade-procedure-logical
https://copyprogramming.com/howto/time-function-changes-between-5-5-and-5-6#mysql-installing-or-upgrading-mysql-55-to-56-in-linux-mint
https://hackernoon.com/upgrading-mysql-55-to-mysql-8-a-step-by-step-guide-ltal35h3


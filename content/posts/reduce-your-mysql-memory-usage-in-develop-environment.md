+++
title = 'Reduce your mysql memory usage in develop environment'
date = 2024-07-22T13:38:00Z
draft = false
+++

```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid


#### These optimize the memory use of MySQL
#### http://www.tocker.ca/2014/03/10/configuring-mysql-to-use-minimal-memory.html
innodb_buffer_pool_size=5M
innodb_log_buffer_size=256K
query_cache_size=0
max_connections=10
key_buffer_size=8
thread_cache_size=0
host_cache_size=0
innodb_ft_cache_size=1600000
innodb_ft_total_cache_size=32000000

# per thread or per operation settings
thread_stack=131072
sort_buffer_size=32K
read_buffer_size=8200
read_rnd_buffer_size=8200
max_heap_table_size=16K
tmp_table_size=1K
bulk_insert_buffer_size=0
join_buffer_size=128
net_buffer_length=1K
innodb_sort_buffer_size=64K

#settings that relate to the binary log (if enabled)
binlog_cache_size=4K
binlog_stmt_cache_size=4K

#### from https://mariadb.com/de/node/579
performance_schema = off
```

[Reference link](https://github.com/alexanderkoller/low-memory-mysql/blob/master/low-memory-my.cnf)
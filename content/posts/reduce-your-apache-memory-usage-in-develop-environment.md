+++
title = 'Reduce your apache memory usage in develop environment'
date = 2024-07-25T14:10:26Z
draft = false
+++

[Reffernce](https://repost.aws/knowledge-center/ec2-apache-memory-tuning)

## Short description

By default, Apache is configured to accept 256 connections at the same time. This limit is defined by **ServerLimit** in Apache's configuration.  

Small instance types, such as t2.small, have only 2 GB of memory. Therefore, small instance types can’t handle 256 concurrent Apache connections, and they might receive an **Out of memory** error when the server gets heavy traffic. In this case, you see an error related to any of the following issues:  

- Out of memory
- Oom
- Oom-killer
- Failure to fork process
- A similar note about insufficient memory in the system logs  

To analyze the system logs of your instances, check the following log files based on your OS distribution:  
- **For Debian and Ubuntu**: /var/log/syslog
- **For Amazon Linux, CentOS, and RHEL**:  /var/log/messages

## Resolution
To avoid memory errors with your instance, set a limit on the number of concurrent Apache and httpd connections that the server accepts. This limit is defined by the parameter **MaxRequestWorkers** (Apache 2.4), and is located in Apache’s configuration. For non-threaded Prefork servers, **MaxRequestWorkers** represents the maximum number of child processes that are launched to serve requests.   

Refer to the following steps to calculate the best value for **MaxRequestWorkers** according to your instance's workload.   

**Note**: The following steps assume that the Apache MPM Model is Prefork, because this is the default MPM that’s used in Apache webservers. To confirm your MPM, run the following command:

```
# apachectl -V | grep "MPM"
```

To calculate the approximate value to set for **MaxRequestWorkers**, determine how much of the following aspects you use for your workload:   

- Your server's physical RAM
- **Remaining Memory** after other services are considered
- Your Apache process that uses the most memory

**Note**: It’s a best practice to use a range of 90-100% of **REMAINING RAM** for **MaxRequestWorkers**.   

## Calculate your server's physical RAM

To calculate RAM, run the following command:   
``` 
# free -m | grep Mem: | awk '{print $2}'
```

## Calculate the remaining memory after other services are considered

First, calculate the memory usage from all other major services. Run the following command for each service that you want to explicitly consider for memory utilization, not including Apache:

``` 
# usage_mbytes=0; for usage_by_pids in `ps -C $proc_cmd -o rss | grep -v RSS`; do usage_mbytes=$(($usage_mbytes + $usage_by_pids)); done; echo $(($usage_mbytes/1024)) MB;
```

**Note**: In the preceding command, replace **$proc_cmd** with the corresponding name or command of the service. For example, use **mysqld** to calculate the memory usage from mysqld service at the given point of time. Repeat this for each service that you want to include in total memory utilization.  

After you calculate each service’s memory usage, add them together to get the sum total of memory usage by all other major services.  

Subtract the memory usage by all other major services from your server's physical RAM in MB. The result of this calculation is your remaining memory after other services are considered:

```
Remaining memory after other services considered = (Your server's physical RAM) - (Memory usage by all other major services)
```

## Calculate the Apache process that uses the most memory

To calculate the process that uses the most memory, run the following command:

```
# high_mem=0; for pid in `ps aux | grep "httpd\|apache2" | grep -v ^root | awk '{print $2 }'`; do memory=`pmap -d $pid | egrep "writeable/private" | awk '{ print $4 }'`; if [[ ${memory::-1} -gt $high_mem ]]; then high_mem="${memory::-1}"; fi; done; echo $(($high_mem/1024)) MB
```

Note how much memory this process consumes.

## Calculate MaxRequestWorkers

Use the following expression to calculate **MaxRequestWorkers**.

```
MaxRequestWorkers = [(Remaining Memory after other services considered) * 90/100]/[Memory consumed by largest Apache process]
```

**Note**: In the preceding expression, replace the values with your results from the previous sections.

This calculation is the same for Ubuntu, Debian, and RHEL based systems. However, the Apache configuration path and file names depend on the OS type. Complete the following steps to set **MaxRequestWorkers** on different OS systems. All of the following examples assume an Apache version of 2.4 or later.   

## For Debian and Ubuntu

1. Log in to the server with SSH.
2. Open the file **/etc/apache2/mods-available/mpm_prefork.conf** in editor mode:
```
# vi /etc/apache2/mods-available/mpm_prefork.conf
```
3. Change the parameter **MaxRequestWorkers** to the value that you calculated. Refer to the following example:
```
<IfModule mpm_prefork_module>  
    StartServers            5  
    MinSpareServers         5  
    MaxSpareServers        10  
    MaxRequestWorkers    your_value  
    MaxConnectionsPerChild  0  
</IfModule>
```
**Note**: Change **your_value** to your calculated value for MaxRequestWorkers.

4.    Save the file, and then exit from the file.

5.    Run the following command to perform an Apache syntax check:

```
# apachectl configtest
```

6. If you receive an output of **Syntax OK**, then run the following command to restart Apache:

```
# systemctl restart apache2
```

## On Amazon Linux 2, CentOS, and RHEL based systems

1.    Log in to the server with SSH.

2.    Create a file with the path **/etc/httpd/conf.d/mpm_prefork.conf**, and add the following command block to the file:

```
<IfModule mpm_prefork_module>  
    StartServers            5  
    MinSpareServers         5  
    MaxSpareServers        10  
    MaxRequestWorkers    <value>  
    MaxConnectionsPerChild  0  
</IfModule>
```

**Note**: Change **your_value** to your calculated value for MaxRequestWorkers. Make sure that the other values in your command block align with your use case.

3.    Save the file, and then exit from the file.

4.    Run the following command to check Apache's configuration syntax:

```
# httpd -t
```
5.    If you receive an output of **Syntax OK**, then run the following command to restart Apache:
```
# systemctl restart httpd
```
# tic

Download tick stack:
====================
https://portal.influxdata.com/downloads/

1. Download required rpms to your laptop and then move to target system.
    Choose the rpm built for RedHat & CentOS
TeleGraf -- v.1.14.0
wget https://dl.influxdata.com/telegraf/releases/telegraf-1.14.0-1.x86_64.rpm

InfluxDB -- v1.7.10
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.10.x86_64.rpm

Chronograf -- v1.8.0
wget https://dl.influxdata.com/chronograf/releases/chronograf-1.8.0.x86_64.rpm


SCP rpms to target system:
scp *.rpm username@<server>:/home/tracerx/tick


Install them:
# rpm -ivh chronograf-1.8.0.x86_64.rpm influxdb-1.7.10.x86_64.rpm telegraf-1.14.0-1.x86_64.rpm
Preparing...                ########################################### [100%]
   1:chronograf             ########################################### [100%]
   2:telegraf               ########################################### [ 50%]
   3:influxdb               ########################################### [100%]
[root@kwikemart2 tick]#



No need to configure chronograh: Once it is installed, we can easily do

Add below to IP Tables: iptables -L
cat /etc/sysconfig/iptables
-A INPUT -p tcp --dport 8888 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -p tcp --sport 8888 -m conntrack --ctstate ESTABLISHED -j ACCEPT


Configure influxdb: This is going to be our metrics repo.
==================
Error logfile: /var/log/influxdb/influxd.log
Config file: /etc/influxdb/influxdb.conf

If you want to store data other than default location. Change it in config file.
Make sure to create "meta", "data" and "wal" directories and change ownership to influxdb.
At line 24:
 [meta]
 # Where the metadata/raft database is stored
    dir = "/home/tick/influxdb/meta"

At line 43:
[data]
 # The directory where the TSM storage engine stores TSM files.
    dir = "/home/tick/influxdb/data"

    # The directory where the TSM storage engine stores WAL files.
    wal-dir = "/home/tick/influxdb/wal"

    wal-fsync-delay = "5s"
    cache-max-memory-size = "1g"
    cache-snapshot-memory-size = "250m"
    cache-snapshot-write-cold-duration = "100m"

 Start InfluxDB:
 service influxdb restart  (CentOS 7 systemctl )
 Check if there any issues in error logfile (/var/log/influxdb/influxd.log)

 If everything goes fine, you can see below process.
 ps -ef | grep influx
 influxdb  6700     1  2 11:54 ?        00:00:10 /usr/bin/influxd -pidfile /var/run/influxdb/influxd.pid -config /etc/influxdb/influxdb.conf



Telegraf:
========
Config file: /etc/telegraf/telegraf.conf
For configuring telegraf,
   1. Credentials of "to be monitored database".
   2. Url of "repo database (influxdb)". Generally influxdb accept passwordless http request so we do not have to create
      username & password for connecting influxdb.


Create monitor username and password in "to be monitored db" (mysql/Mariadb):
mysql> create user 'mysql_monitor'@'localhost' identified by "mYSQlPwd2";
Query OK, 0 rows affected (0.00 sec)

mysql> grant all on *.* to 'mysql_monitor'@'localhost';
Query OK, 0 rows affected (0.00 sec)


Open config file (/etc/telegraf/telegraf.conf),
1. At line 105, make sure "[[outputs.influxdb]]" is uncommented.
2. At line 112, uncomment and set url as "urls = ["http://localhost:8086"]"
3. at line 116, set "database = "telegraf""
4. At line 146, set "timeout = "15s""
5. At line 1800, make sure to uncomment "[[inputs.cpu]]" and enable required values.
6. at line 1812. uncomment "[[inputs.disk]]" and at 1822, [[inputs.diskio]]
7. at 3652, uncomment mysql plugin "[[inputs.mysql]]" and Maria/MySql connection info that we created earlier.
    a. Also uncomment required matrics variable under this plugin.

Restart Telegraf and make sure no errors in "/var/log/telegraf/telegraf.log".
service telegraf restart
telegraf process was stopped [ OK ]
Starting the process telegraf [ OK ]
telegraf process was started [ OK ]


ChronoGraf:
==========

Follow below screenshots.
https://www.influxdata.com/blog/how-predefined-dashboards-influxdatas-chronograf-make-metrics-simple-blog-post/

1.
user admin
---- -----
> CREATE USER telegraf_user WITH PASSWORD 'admin1' WITH ALL PRIVILEGES;
> show users;
user          admin
----          -----
telegraf_user true

-A INPUT -p tcp --dport 8888 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -p tcp --sport 8888 -m conntrack --ctstate ESTABLISHED -j ACCEPT



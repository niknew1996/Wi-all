# STEP BY STEP: HOW TO Config Redis-Sentinel

After install Redis in /etc/redis will have Sentinel.conf file

##### 2.1. create Directory Redis and Sentinel
```bash
mkdir -p /app/redis-sentinel
chown redis:redis -R /app/redis-sentinel
```
 ##### 2.5. Verify redis and sentinel   “If service is not start follow step 2.4”
```bash
 ps -ef | grep redis |grep -v grep

 #result
 ps -ef | grep redis |grep -v grep
redis     482399       1  0 11:28 ?        00:00:09 /usr/bin/redis-server 0.0.0.0:6379
redis     482422       1  0 11:28 ?        00:00:10 /usr/bin/redis-sentinel *:26379 [sentinel]

 ```
 ##### 2.6. Start service and verify service redis
```bash
#redis
systemctl start redis
systemctl enable redis
ps -ef | grep redis
 ```
 #### 3 Start Redis-sentinel and with user redis by systemctl

##### 3.1 stop redis-sentinel
```bash
systemctl stop redis-sentinel
 ```
##### 3.2 edit config Redis
```bash
vi /etc/redis/sentinel.conf
 ```

```bash
#add config
supervised systemd

#edit value 
pidfile "/run/redis/redis-sentinel.pid"

# Set the log level to "debug" for development (more verbose output)
loglevel debug

# Logfile location for easy troubleshooting
logfile "/app/redis-sentinel/log/sentinel.log"

# Define the master Redis instance to monitor (using local Redis)
sentinel monitor vdomwdev <master-IP> 6379 2

# Authentication password for the master instance (only if needed in dev environment)
sentinel auth-pass vdomwdev <password for replica>

# Time in milliseconds to consider the master or replica down after being unreachable
sentinel down-after-milliseconds vdomwdev 10000

# Failover timeout (milliseconds) to determine how long to wait for a failover to complete
sentinel failover-timeout vdomwdev 60000
 ```
##### 3.3 Change Permissions file
 ```bash
#configfile
sudo chown redis:redis /etc/redis/sentinel.conf
sudo chmod 640 /etc/redis/sentinel.conf

#Log Directory
mkdir -p /app/redis-sentinel/log/sentinel.log
chown redis:redis -R /app/redis-sentinel
chmod 770 /app/redis-sentinel/log

#PID Directory
touch /run/redis/redis-sentinel.pid
chown redis:redis -R /run/redis
chmod 755 /run/redis
chmod 644 /run/redis/redis-sentinel.pid
 ```
 ##### 3.4. Start service and verify service redis
```bash
#redis
systemctl start redis-sentinel
systemctl status redis-sentinel
ps -ef | grep redis-sentinel
redis     482422       1  0 11:28 ?        00:00:11 /usr/bin/redis-sentinel *:26379 [sentinel]
root      499723  494631  0 15:09 pts/1    00:00:00 grep --color=auto redis-sentinel
 ```
 ```bash
#Check in sentinel
redis-cli -p 26379 info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_tilt_since_seconds:-1
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=vdomwdev,status=ok,address=10.253.134.81:6379,slaves=2,sentinels=3
 ```
##### Test down Master
on master node
 ```bash
systemctl stop redis
 ```
 on some node
```bash
#Check in sentinel
redis-cli -p 26379 info sentinel

# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_tilt_since_seconds:-1
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=vdomwdev,status=ok,address=10.253.134.82:6379,slaves=2,sentinels=3
 ```
 master will change to another node and rewrite in sentinel.conf
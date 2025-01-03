# STEP BY STEP: HOW TO Create Redis Replica

### step 1 :  Crete user redis nologin
```bash
sudo useradd -M -s /sbin/nologin redis
```

### Step 2 : install Redis and Sentinel
##### 2.1. create Directory Redis and Sentinel
```bash
mkdir -p /app/redis
mkdir -p /app/redis-sentinel
chown redis:redis -R /app/redis/
```
##### 2.2 Upload file redis rpm
Upload file to server
> redis-7.2.6-1.el9.remi.x86_64.rpm

##### 2.3. move file redis rpm to /tmp
```bash
mv redis-7.2.6-1.el9.remi.x86_64.rpm /tmp
```
###### 2.4. install redis
```bash
 rpm -ivh redis-7.2.6-1.el9.remi.x86_64.rpm
 ```

 ##### 2.5. Verify redis and sentinel install  “If service is not start follow step 2.4”
```bash
 ps -ef | grep redis |grep -v grep

 #result
redis     482399       1  0 11:28 ?        00:00:02 /usr/bin/redis-server 0.0.0.0:6379
 ```
 ##### 2.6. Start service and verify service redis
```bash
#redis
systemctl start redis
systemctl enable redis
ps -ef | grep redis
 ```
 #### 3 Start Redis and with user redis by systemctl

##### 3.1 stop redis
```bash
systemctl stop redis
systemctl stop redis-sentinel
 ```
##### 3.2 edit config Redis
```bash
vi /etc/redis/redis.conf
#add config
supervised systemd

#edit value 
bind 0.0.0.0
pidfile "/run/redis/redis_6379.pid"
logfile "/app/redis/log/redis.log"
dir "/app/redis/db"
masterauth "password for replica"
requirepass "password for client and sentinel"
 ```
##### 3.3 Change Permissions file
 ```bash
#configfile
sudo chown redis:redis /etc/redis/redis.conf
sudo chmod 640 /etc/redis/redis.conf

#DB Directory
mkdir -p /app/redis/db
chown redis:redis -R /app/redis/db
chmod 770 /app/redis/db

#Log Directory
mkdir -p /app/redis/log
chown redis:redis -R /app/redis/db
chmod 770 /app/redis/db

#PID Directory
mkdir -p /run/redis
touch /run/redis/redis_6379.pid
chown redis:redis -R /run/redis
chmod 755 /run/redis
chmod 644 /run/redis/redis_6379.pid
 ```
 ##### 3.4. Start service and verify service redis
```bash
#redis
systemctl start redis
systemctl status redis
ps -ef | grep redis

#Check
redis-cli
> AUTH "password for client and sentinel"
PING

#Resualt
PONG
 ```
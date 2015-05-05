# IdevOps.net Infrastructure
idevops.net constructed at zone GD1 of QingCloud.

# Computer

|ID        | Host Name ( idevops.net )| Image (ID)                       | Network                                | Size   |
|----------|--------------------------|----------------------------------|----------------------------------------|-------|
|i-tvkhhb1r| ldap chef| ubuntu 14.04 (trustysrvx64c)| (devo-vxnet) / 192.168.200.5| 2C 4G|
|i-99ouavas| build ci| centos 7 (centos7x64b)| (devo-vxnet) / 192.168.200.3| 2C 4G|
|i-j5zkfqhl| web idevops| centos 7 (centos7x64b)| (devo-vxnet) / 192.168.200.2| 2C 4G|
|i-iv8w9pok| jenkins slave | centos 7 (centos7x64b)| (devo-vxnet) / 192.168.200.6| 2C 4G|
|i-dy8hm7fs| nalanda | centos 7 (centos7x64b)| (devo-vxnet) / 192.168.200.7| 2C 8G|

### /etc/hosts

```
192.168.200.5   ldap chef chef.idevops.net ldap.idevops.net
192.168.200.3   ci build ci.idevops.net build.idevops.net
192.168.200.2   idevops web web.idevops.net www.idevops.net idevops.net
192.168.200.7   mesos
```
### Services

VM ID     | IP          | Services
----------|-------------|-----------
i-j5zkfqhl|192.168.200.2| httpd
i-99ouavas|192.168.200.3| jenkins, zuul, docker-registry
i-tvkhhb1r|192.168.200.5| ldap, chef, phpldapadmin
i-iv8w9pok|192.168.200.6| jenkins centos7 slave
i-iv8w9pok|192.168.200.7| nalanda

Host          | Container Name | IP            | Port             | Services
--------------|----------------|---------------|------------------|---------
192.168.200.7 |zookeeper_1     |               | 2181, 2888, 3888 | Zookeeper cluster
192.168.200.7 |zookeeper_2     |               | 2182, 2889, 3889 | Zookeeper cluster
192.168.200.7 |zookeeper_3     |               | 2183, 2890, 3890 | Zookeeper cluster
192.168.200.7 |mesos_master_1  |192.168.200.20 |                  | Mesos master
192.168.200.7 |mesos_master_2  |192.168.200.21 |                  | Mesos master
192.168.200.7 |mesos_master_3  |192.168.200.22 |                  | Mesos master
192.168.200.7 |mesos_slave     |192.168.200.30 |                  | Mesos slave
192.168.200.3 |mesos_slave     |192.168.200.31 |                  | Mesos slave
192.168.200.6 |mesos_slave     |192.168.200.32 |                  | Mesos slave
192.168.200.7 |marathon_1      |192.168.200.40 |                  | Mesos marathon
192.168.200.7 |marathon_2      |192.168.200.41 |                  | Mesos marathon
192.168.200.7 |chronos_1       |192.168.200.42 |                  | Mesos chronos
192.168.200.7 |chronos_2       |192.168.200.43 |                  | Mesos chronos
192.168.200.7 |hdfs_namenode   |192.168.200.23 |                  | Hadoop HDFS Namenode
192.168.200.7 |hdfs_datanode_1 |192.168.200.24 |                  | Hadoop HDFS Datanode
192.168.200.7 |hdfs_datanode_2 |192.168.200.25 |                  | Hadoop HDFS Datanode
192.168.200.7 |hdfs_datanode_3 |192.168.200.26 |                  | Hadoop HDFS Datanode

```
* openldap server and phpldapadmin are running on `192.168.200.5` as docker container. LDAP data is under /data/slapd as docker container volume.
  $ docker run -d -p 389:389 -v /data/slapd/config:/etc/ldap/slapd.d -v /data/slapd/database:/var/lib/ldap -e LDAP_ORGANISATION=Zerbtech -e LDAP_DOMAIN="idevops.net" -e LDAP_ADMIN_PASSWORD="adc2tek" -e SERVER_NAME="ldap" -e USE_TLS=false --name ldap 192.168.200.3:5000/openldap:0.10.1
  $ docker run -p 4403:443 -e LDAP_HOSTS=192.168.200.5 -e SSL_CRT_FILENAME=ldap.crt -e SSL_KEY_FILENAME=ldap.key -v /var/opt/chef-server/nginx/ca:/osixia/phpldapadmin/apache2/ssl -d --name phpldapadmin 192.168.200.3:5000/phpldapadmin:0.5.0

* gerrit server is running on `192.168.200.5` as docker container. Gerrit data is under /data/gerrit as docker container volume.
  $ docker run -t -d --name="gerrit" -p 8080:8080 -p 29418:29418 -e AUTH_TYPE=LDAP -e GERRIT_URL=http://review.idevops.net:33080 -e LDAP_SERVER=ldap://192.168.200.5 -e LDAP_ACCOUNT_BASE="ou=people,dc=idevops,dc=net" -e REPLICATE_KEY=/home/gerrit2/.ssh/ci_rsa -e REPLICATE_USER=idevops-ci -v /data/gerrit:/home/gerrit2/gerrit -v /data/ssh_key:/home/gerrit2/.ssh 192.168.200.3:5000/gerrit:0.0.2

* Zuul server is running on `192.168.200.3` as docker container.
  $ docker run --name zuul-server -d -e GERRIT_SERVER=192.168.200.5 -e GERRIT_URL=http://review.idevops.net:33080 -e JENKINS_SSH_KEY=/home/jenkins/.ssh/ci_rsa -v /build/ssh_key/ssh:/home/jenkins/.ssh -v /root/ci/project-config/zuul:/etc/zuul-layout -p 4730:4730 192.168.200.3:5000/zuul:0.0.2

* Zookeeper cluster is running on `192.168.200.7` as docker container.
  $ git clone https://github.com/power-stack/nalanda.git
  $ cd nalanda/docker-scripts && ./docker-zookeeper

* Docker image registry is running on `192.168.200.3` as docker container. Image data is under /build/docker_registry as docker container volume.
  $ docker run -v /build/docker_registry/:/build/docker_registry -e STORAGE_PATH=/build/docker_registry -e SEARCH_BACKEND=sqlalchemy -p 5000:5000 --name docker_registry -t -d registry
```

# Router
ID          |	Name  |	Status  |Type  |	EIP            |Private IP
------------|-------|---------|------|-----------------|---------------
rtr-gmrunkjf|	devo_r|	  Active|	Small|**121.201.13.44**|192.168.200.1

## Forward Table

Name|	Protocol|	Source Port|	Intranet IP|	Intranet Port
----|---------|------------|-------------|---------------
https|	tcp|	33443|	192.168.200.2|	33443
ssh|	tcp|	22|	192.168.200.2|	22
http|	tcp|	33080|	192.168.200.2|	33080

# Firewall Rules
Priority|	Protocol|	Action|	Start Port|	End Port|	Source IP
--------|---------|-------|-----------|---------|----------
1|	ICMP|Accept|	Echo|	Echo request|
2|	TCP	|Accept|	33080||
2|	TCP	|Accept|	22|	22|
3|	TCP	|Accept|	33443||

# Volumes
ID          |	Name |	Status  | Instances/Device|	Size (GB)|	Type
------------|------|----------|-----------------|----------|-------
vom-toojo38e|	ci|	  In Use|	(build)/dev/sdc |100|	High Capacity
vom-8jf5awac|	disk1|	  In Use|	(web)/dev/sdb | 100|	High Capacity
vom-yefsvdzp|ldap|  In Use| (ldap)/dev/sdc | 100|	High Capacity

## Mount points

* **192.168.200.2 (web)**
```
[bryan@idevops ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
└─sda1   8:1    0   20G  0 part /
sdb      8:16   0  100G  0 disk
├─sdb1   8:17   0   50G  0 part /var/www
└─sdb2   8:18   0   50G  0 part /var/lib/jenkins
sdc      8:32   0    4G  0 disk [SWAP]
```

* **192.168.200.3 (ci)**
```
[root@ci ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
└─sda1   8:1    0   20G  0 part /
sdb      8:16   0    4G  0 disk [SWAP]
sdc      8:32   0  100G  0 disk
├─sdc1   8:33   0   50G  0 part /build
└─sdc2   8:34   0   50G  0 part /var/lib/jenkins
```

* **192.168.200.5 (chef/ldap)**
```
root@ldap:~# lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0    20G  0 disk
└─sda1   8:1    0    20G  0 part /
sdb      8:16   0     4G  0 disk [SWAP]
sdc      8:32   0   100G  0 disk
├─sdc1   8:33   0    50G  0 part /data
└─sdc2   8:34   0    50G  0 part
```

# IdevOps.net Infrastructure
idevops.net constructed at zone GD1 of QingCloud.

# Computer

|ID        | Host Name ( idevops.net )| Image (ID)                       | Network                                | Size   |
|----------|--------------------------|----------------------------------|----------------------------------------|-------|
|i-tvkhhb1r| ldap chef| ubuntu 14.04 (trustysrvx64c)| (devo-vxnet) / 192.168.200.5| 2C 4G|
|i-99ouavas| build ci| centos 7 (centos7x64b)| (devo-vxnet) / 192.168.200.3| 2C 4G|
|i-j5zkfqhl| web idevops| centos 7 (centos7x64b)| (devo-vxnet) / 192.168.200.2| 2C 4G|

### /etc/hosts

```
192.168.200.5   ldap chef chef.idevops.net ldap.idevops.net
192.168.200.3   ci build ci.idevops.net build.idevops.net
192.168.200.2   idevops web web.idevops.net www.idevops.net idevops.net
```
### Services

VM ID     | IP          | Services
----------|-------------|-----------
i-j5zkfqhl|192.168.200.2| httpd
i-99ouavas|192.168.200.3| jenkins, gerrit, docker-registry
i-tvkhhb1r|192.168.200.5| ldap, chef

```
* openldap server and phpldapadmin are running on `192.168.200.5` as docker container.
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

# ans-docker

This repo has the ansible playbook, which does the following.

* Creates two docker containers which runs the SSHD daemons.
* Two container allows password-less SSH login from the host.
* Generates Password SSH keys on the first docker and copies ssh public key to the second docker.

Following are the detailed steps.

1. Creates an Docker image "centos-sshd" with CentOS as the base. 
   * Image was created using the Dockerfile. 
   * Image has "docker_root" user created and gives passwordless SSH  access t
1. Creates a bridge network "br01". 
   * Not necessary to create a bridge we could also use the default bridge.
1. Starts two container "sshd01" and "sshd02" on the br01
1. Adds two container with IPs to the ansible inventory group "sshd_dockers".
1. Using ansible "user" module, we create passwordless key in the sshd01.
1. Using ansible "authorized_key", add the public key to the container sshd02.

### Pre-requisites

* Ansible and Docker should be install. 
* Docker python api version should be below 3.0.0. I was trying with docker-py version >3.1, I was facing the issue https://github.com/ansible/ansible/issues/35612 while building the docker image
* Docker daemon should be running.

### To run the playbook

Clone the repo. 
```
$ git clone https://github.com/ashokraj/ans-docker
$ cd ans-docker
```  

Add the host SSH passwordless public key to "group_vars/all"
```
$ cat group_vars/all
dockuser: docker_root

pubkey: "<Place Your Public SSH key here>"
$
$
$ ansible-playbook --ask-sudo-pass -i hosts.yml play.yml

```

Once the playbook ran fine check the 

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
414a14bccec0        centos-sshd         "/sbin/sshd -D"     15 minutes ago      Up 15 minutes       22/tcp              sshd02
3b6e81946496        centos-sshd         "/sbin/sshd -D"     15 minutes ago      Up 15 minutes       22/tcp              sshd01

$docker inspect -f '{{ .NetworkSettings.Networks.br01.IPAddress }}' sshd01
172.18.0.2

$  docker inspect -f '{{ .NetworkSettings.Networks.br01.IPAddress }}' sshd02
172.18.0.3


$ ssh 172.18.0.2 -l docker_root
Last login: Mon Apr 16 04:59:33 2018 from gateway
[docker_root@8d33107e0f3e ~]$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.2  netmask 255.255.0.0  broadcast 0.0.0.0
        inet6 fe80::42:acff:fe12:2  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ac:12:00:02  txqueuelen 0  (Ethernet)
        RX packets 250  bytes 92901 (90.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 248  bytes 27904 (27.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 250  bytes 21791 (21.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 250  bytes 21791 (21.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[docker_root@8d33107e0f3e ~]$ ssh sshd02
Last login: Mon Apr 16 04:59:48 2018 from gateway

[docker_root@1836b61a1a2a ~]$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.3  netmask 255.255.0.0  broadcast 0.0.0.0
        inet6 fe80::42:acff:fe12:3  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ac:12:00:03  txqueuelen 0  (Ethernet)
        RX packets 201  bytes 94101 (91.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 195  bytes 22015 (21.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 210  bytes 17717 (17.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 210  bytes 17717 (17.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```


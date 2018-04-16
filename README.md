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

Add the host SSH passwordless public key to "docker/defaults/main.yml"
```
$ cat docker/defaults/main.yml
---
# defaults file for ans-docker
pubkey: "<PLACE YOUR KEY HERE>"

$
$
$ ansible-playbook --ask-sudo-pass -i hosts.yml play.yml

```


# ans-docker

This repo has the ansible playbook for

* Creates two docker containers which runs the SSHD command.
* Two container allows password less SSH access to the host.
* Generates Password SSH keys on the first docker.
* Copies ssh public key to the second docker.

Following are the detailed steps.

1. Install and Start Docker.
1. Creates an Docker image "centos-sshd" with CentOS as the base. 
   * Image was created using the Dockerfile. 
   * Image has "docker_root" user created and gives passwordless SSH  access t
1. Creates a bridge network "br01". Not necessary to create a bri we could also use the default bridge.
1. Starts two container "sshd01" and "sshd02" on the br01
1. Adds two container with IPs to the ansible inventory group "sshd_dockers".
1. Using ansible "user" module, we create passwordless key in the sshd01.
1. Using ansible "authorized_key", add the public key to the container sshd02.

For testing 
Currently we need to know the IP address to 

FROM centos:latest

ARG dockuser
ARG pubkey

RUN useradd $dockuser 
## ENV 
RUN yum -y install openssh-server openssh openssh-clients net-tools
RUN ssh-keygen -A



USER $dockuser
RUN mkdir ~/.ssh
RUN echo $pubkey >> ~/.ssh/authorized_keys
USER root

EXPOSE 22

CMD ["/sbin/sshd","-D"]

FROM ubuntu:18.04
MAINTAINER t00402375, From: https://github.com/GRomR1/docker-slurmbase

#最新release版本
ENV SLURM_VER=20.02.3

# Create users, set up SSH keys (for MPI)
RUN useradd -u 2001 -d /home/slurm slurm
RUN useradd -u 6000 -ms /bin/bash ddhpc
COPY etc/sudoers.d/ddhpc /etc/sudoers.d/ddhpc
COPY home/ddhpc/ssh/config /home/ddhpc/.ssh/config
COPY home/ddhpc/ssh/id_rsa /home/ddhpc/.ssh/id_rsa
COPY home/ddhpc/ssh/id_rsa.pub /home/ddhpc/.ssh/id_rsa.pub
COPY home/ddhpc/ssh/authorized_keys /home/ddhpc/.ssh/authorized_keys
RUN chown -R ddhpc:ddhpc /home/ddhpc/.ssh/
RUN chmod 400 /home/ddhpc/.ssh/*

#使用华为云的Ubuntu源，国内嘛
RUN sed -i "s@http://.*archive.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list
RUN sed -i "s@http://.*security.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list

# Install packages
RUN apt-get update && apt-get -y  dist-upgrade
RUN apt-get install -y munge curl gcc make bzip2 supervisor python python-dev \
    libmunge-dev libmunge2 lua5.3 liblua5.3-dev libopenmpi-dev openmpi-bin \
    gfortran vim python-mpi4py python-numpy python-psutil sudo psmisc \
	software-properties-common iputils-ping \
    openssh-server openssh-client \
    automake autoconf unzip \
    libgtk2.0-dev libglib2.0-dev \
    libhdf5-dev \
    libcurl4-openssl-dev


# 提前下载包, compile and install SLURM
# 1） https://download.schedmd.com/slurm/slurm-20.02.3.tar.bz2
# 2） 这个删了 https://github.com/GRomR1/influxdb-slurm-monitoring/archive/master.zip
COPY slurm-${SLURM_VER}.tar.bz2 /
#COPY influxdb-slurm-monitoring-master.zip /
RUN tar --bzip -x -f slurm-${SLURM_VER}.tar.bz2 -C /opt/ && \
    cd /opt/slurm-${SLURM_VER}/ && \
    ./configure && \
    make && make install
COPY etc/slurm/slurm.conf /usr/local/etc/slurm.conf
COPY etc/slurm/cgroup.conf /usr/local/etc/cgroup.conf
#暂时不用 ACCOUNTING
#COPY etc/slurm/acct_gather.conf /usr/local/etc/acct_gather.conf


# Configure OpenSSH
# Also see: https://docs.docker.com/engine/examples/running_ssh_service/
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
RUN mkdir /var/run/sshd
RUN echo 'ddhpc:ddhpc' | chpasswd
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
COPY etc/supervisord.d/sshd.conf /etc/supervisor/conf.d/sshd.conf


# Configure GlusterFS
# RUN add-apt-repository ppa:gluster/glusterfs-3.8 && \
#     apt-get update -y && \
#     apt-get install -y glusterfs-server
#
# RUN mkdir -p /data/ddhpc
# COPY etc/supervisord.d/glusterd.conf /etc/supervisor/conf.d/glusterd.conf


# Configure munge (for SLURM authentication)
COPY etc/munge/munge.key /etc/munge/munge.key
RUN mkdir /var/run/munge && \
    chown root /var/lib/munge && \
    chown root /etc/munge && chmod 600 /var/run/munge && \
    chmod 755  /run/munge && \
    chmod 600 /etc/munge/munge.key
COPY etc/supervisord.d/munged.conf /etc/supervisor/conf.d/munged.conf

EXPOSE 22
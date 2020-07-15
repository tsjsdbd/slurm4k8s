# Docker SLURM Cluster



项目参考： -》 from **[GRomR1](https://github.com/GRomR1/docker-slurmbase)** -》 它又是 from **[Data Driven HPC](https://github.com/datadrivenhpc/docker-slurmbase)**  但是这货已经404了。

当前目录执行：
```
docker build -t tsjsdbd/slurmbase .
```

附注：生成 slurm.conf 配置文件，可以使用 configurator.html

----
原内容：
that provides a set of containers that can be used to run a SLURM HPC cluster as a set of Docker
containers. The project consists of three components:

1. [docker-slurmctld](https://github.com/GRomR1/docker-slurmctld) provide
a SLURM controller or "head node".

2. [docker-slurmd](https://github.com/GRomR1/docker-slurmd) provides a
SLURM compute node.

3. [docker-slurmbase](https://github.com/GRomR1/docker-slurmbase) is the
base container from which both docker-slurmctld and docker-slurmd inherit.

In the repository was added [influxdb-plugin](https://github.com/GRomR1/influxdb-slurm-monitoring) 
that gather accounting data about tasks and store it in an external database ([InfluxDB](https://docs.influxdata.com/influxdb)). 

This repository contains the container source files. The ready built container
images are available via DockerHub: [https://hub.docker.com/r/gromr1](https://hub.docker.com/r/gromr1).

The Docker SLURM cluster is configured with the following software packages:

- Ubuntu 16.04 LTS
- SLURM 16.05.9
- GlusterFS 3.8
- Open MPI 1.10.2

A user `ddhpc` is configured across all nodes for MPI job execution and a shared
GlusterFS volume *ddhpc* is mounted on all nodes as `/data/ddhpc`. The head node
runs an SSH server for accessing the cluster.

## Launch a New SLURM cluster

Create a new directory with a `docker-compose.yml` file:

```
version: '2'

services:
  slurmctld:
    container_name: slurmctld
    environment:
      SLURM_CLUSTER_NAME: ddhpc
      SLURM_CONTROL_MACHINE: slurmctld
      SLURM_NODE_NAMES: slurmd
      INFLUXDB_HOST: influxdb
      INFLUXDB_DATABASE_NAME: docker_slurm
    tty: true
    hostname: slurmctld
    networks:
      default:
        aliases:
          - slurmctld
    image: gromr1/slurmctld
    stdin_open: true
  slurmd:
    container_name: slurmd
    environment:
      SLURM_CONTROL_MACHINE: slurmctld
      SLURM_CLUSTER_NAME: ddhpc
      SLURM_NODE_NAMES: slurmd
      INFLUXDB_HOST: influxdb
      INFLUXDB_DATABASE_NAME: docker_slurm
    tty: true
    hostname: slurmd
    networks:
      default:
        aliases:
          - slurmd
    image: gromr1/slurmd
    depends_on:
      - slurmctld
    stdin_open: true
```

After that you can create and run the configured containers with a command `docker-compose up -d`.

For a stopping them run `docker-compose down`. 


**Configuration variables**:

  * `SLURM_CLUSTER_NAME`: the name of the SLURM cluster.
  * `SLURM_CONTROL_MACHINE`: the host name of the controller container. This should match `hostname` in the `slurmctld` section.
  * `SLURM_NODE_NAMES`: the host name of the compute node container. This should match `hostname` in the `slurmd` section.
  * `INFLUXDB_HOST`: the host name of the database host. 
  * `INFLUXDB_DATABASE_NAME`: the name of existing database in influxdb host. Database should exists a retention policy with name 'default'. 

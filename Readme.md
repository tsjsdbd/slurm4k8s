# 使用K8s一键部署Slurm集群

项目： -》 From **[GRomR1](https://github.com/GRomR1/docker-slurmbase)** -》 它又是 From **[Data Driven HPC](https://github.com/datadrivenhpc/docker-slurmbase)**  但是这货已经404了。

## 构建镜像

1. 基础镜像
```
cd ./docker-images/slurmbase
docker build -t tsjsdbd/slurmbase .
```

2. worker镜像
```
cd ./docker-images/slurmd
docker build -t tsjsdbd/slurmd .
```

3. head镜像
```
cd ./docker-images/slurmctld
docker build -t tsjsdbd/slurmctld .
```


## 使用k8s启动Slurm集群

修改`slurm-master.yaml`和 `slurmd.yaml`中镜像地址，然后创建以下4个对象：
```
kubectl create -f ./slurm-master-svc.yaml
kubectl create -f ./slurmd-svc.yaml
kubectl create -f ./slurm-master.yaml
kubectl create -f ./slurmd.yaml
```
默认4个slurm节点(容器)

### 如何使用
登录到master容器里面

```
kubectl get pod
kubectl exec -it slurm-master-7946898f57-kn8wt /bin/bash
su ddhpc
sinfo
```

## 单机启动slurm集群
使用docker-compose在本机启动多个容器（注：先安装docker-compose）
```
cd ./docker-compose
docker-compose up -d
```

### 如何使用
```
docker-compose ps
docker exec -it docker-compose_slurmctld_1 /bin/bash
sinfo -lN
```
删除
```
docker-compose down
```

## 调整slurm节点数量
1. 环境变量 `SLURM_NODE_NAMES` 当前为 `slurmd-[0-4]`，可以设大一点
2. slurmd 的副本数，参数设置 `replicate=10` 就有10个worker节点

## 原理

创建了：

1. slurmctld 使用了 deployment
2. slurmctld -> Headless Service （使得slurmd可以通过master-name获得pod ip）
2. slurmd 使用了 StatefulSet （使得每个pod的 hostname结尾 0-N 格式）
3. slurmd 给一个 Headless Service （使得slurmctld可以通过slave-name获得pod ip）

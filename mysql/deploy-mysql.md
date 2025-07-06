---
title: "Mysql服务部署"
weight: 1
catalog: true
date: 2025-03-9 10:50:57
subtitle:
tags:
- Mysql
catagories:
- Mysql
---

# 1. 通过容器的方式部署

```bash
mkdir -p ~/data/mysql

docker run --name my-mysql -v ~/data/mysql:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

# 2. 通过k8s deployment的方式部署

部署的注意事项：

- mysql数据的持久化：数据卷映射和固定宿主机

- 设置账号密码及服务端口

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate  # 先删除旧 Pod，再启动新 Pod
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "rootpass"
            - name: MYSQL_USER
              value: "user"
            - name: MYSQL_PASSWORD
              value: "userpass"
          ports:
            - containerPort: 3306  # MySQL 默认端口
              hostPort: 13306  # 直接映射到宿主机端口
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql  # MySQL 数据目录
      volumes:
        - name: mysql-storage
          hostPath:
            path: /data/mysql # 宿主机上的 MySQL 存储目录
            type: DirectoryOrCreate
      nodeName: mysql-node
```

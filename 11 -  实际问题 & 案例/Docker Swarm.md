## 基本概念
Docker Swarm是Docker官方的容器编排工具，将多台Docker主机组成一个集群，提供容器化应用的分布式部署和管理能力

一个Swarm集群的构成：

**Swarm**：计算机集群（cluster）在用Docker连接后的状态，docker swarm命令可以创建、加入、离开一个集群；

**Node**：分为Manager和Worker。一个Swarm至少要有一个Manager，才能执行管理其他Node相关命令。

**Stack**：一组Service， 默认情况下，一个Stack共用一个Network，相互可访问，与其它Stack网络隔绝。 这个概念只是为了编排的方便。 docker stack命令可以方便地操作一个Stack，而不用一个一个地操作Service

**Service**：一类容器，一般来讲Service有两种模式。replicated，指定一个Service运行容器的数量；global，在所有符合运行条件的Node上，都运行一个这类容器；docker service这一类命令可以操作service。

**Task**: 指运行一个容器的任务，是Swarm执行命令的最小单元。 要成功运行一个Service，需要执行一个或多个Task（取决于一个Service的容器数量），确保每一个容器都顺利启动。 通常用户操作的是Service，而非Task



![[Pasted image 20250527174658.png]]

 为什么需要多个Docker主机？
- 应对高负载需求：单台服务器的资源（CPU、内存、带宽）有限，无法满足高并发场景
- 高可用性保障：单机模式下，如果主机发生故障，所有容器将不可用
- 资源优化利用：CPU密集型容器和内存密集型容器可调度到不同特性的节点上；AI推理服务可以专门调度到有GPU的节点上运行
- 分阶段更新和测试：配置更新策略，先在部分节点上部署新版本
- 成本优化：多台较小配置的服务器通常比一台超大配置服务器成本更低；根据需求增减节点，避免资源浪费

为什么要用Swarm管理多个Docker主机？

假设你运营一个电商平台，即将迎来"双十一"购物节。平时网站流量适中，但促销期间流量将暴增10倍

- **弹性扩展**: 促销前，你可以简单地执行docker service scale web=20将Web服务从3个实例扩展到20个，应对流量高峰。促销结束后，同样简单地缩减回来。
- **零停机更新**：促销期间发现bug需要紧急修复，使用docker service update --image newversion:v2 web可以执行滚动更新，一次替换一个容器，确保服务持续可用。
- **自动故障恢复**：如果某个节点因高负载崩溃，Swarm会自动将该节点上的容器迁移到健康节点，无需人工干预。
- **部署简单**：整个应用栈(Web、API、数据库、缓存等)可以通过一个[[#yaml | docker-compose.yml]]文件定义并一键部署：docker stack deploy -c docker-compose.yml ecommerce
![[Pasted image 20250528203959.png]]
## Swarm命令
`docker swarm init` -- 初始化swarm集群

`docker swarm init --advertise-addr <IP地址>` -- 多网卡初始化

`docker service create --name web --replicas 3 -p 80:80 nginx` -- 部署3副本的web服务

`docker service ls` -- 查看服务列表

`docker service ps web` -- 查看特定服务详情

`docker service scale web=5` -- 服务器扩容至5个实例

离开Swarm的命令

`docker stack rm <stack_name>` -- 清理Stack资源

`docker service rm $(docker service ls -q)` -- 移除服务资源

`docker swarm leave --force` -- 回到标准的单机Docker



### yaml
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - app-network

  api:
    image: node:14-alpine
    command: npm start
    deploy:
      replicas: 2
    ports:
      - "3000:3000"
    networks:
      - app-network
    environment:
      - NODE_ENV=production

networks:
  app-network:
    driver: overlay
```




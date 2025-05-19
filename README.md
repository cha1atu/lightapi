# TBC服务套件部署说明书

## 部署步骤

### 准备工作

1. 确保已安装Docker和Docker Compose
   ```
   docker --version
   docker compose --version
   ```

2. 拉取项目
   ```
   git clone https://github.com/cha1atu/lightapi
   cd lightapi
   ```


### 启动服务

1. 启动所有服务
   ```
   docker compose up -d
   ```

2. 查看服务启动情况
   ```
   docker compose ps
   ```

## 服务状态检查

### 1. TBCNode检查

进入容器并检查节点状态：
```
docker exec -it tbcnode /bin/bash
alias tbc-cli="/TBCNODE/bin/bitcoin-cli -conf=/TBCNODE/node.conf -datadir=/TBCNODE/node_data_dir"
tbc-cli getinfo
```

查看日志：
```
docker exec -it tbcnode /bin/bash
pm2 logs tbcd
```

查看区块同步状态：
```
docker exec -it tbcnode /bin/bash
alias tbc-cli="/TBCNODE/bin/bitcoin-cli -conf=/TBCNODE/node.conf -datadir=/TBCNODE/node_data_dir"
tbc-cli getblockcount
```


### 2. ElectrumX检查

查看ElectrumX日志：
```
docker logs electrumx --tail 100
```

查看实时日志：
```
docker logs --tail 100 -f electrumx
```


### 3. TBCAPI检查

查看API服务日志：
```
docker logs tbcapi --tail 100
```

测试API是否启动：
```
curl http://localhost:5000/v1/tbc/main/health
```

## 常见问题排查

### 如果服务无法启动

检查容器错误日志：
```
docker logs [容器名称]
```


### 数据持久化确认

检查数据目录是否正常创建：
```
ls -la ./node_data
ls -la ./electrumx_data
ls -la ./mysqldata
```

## 服务更新

更新所有服务：
```
docker compose pull
docker compose up -d
```

更新特定服务：
```
docker compose pull [服务名]
docker compose up -d [服务名]
```


## 服务停止和清理

停止所有服务：
```
docker compose down
```

停止服务并删除网络（保留数据）：
```
docker compose down --remove-orphans
```

完全清理（慎用，会删除所有数据）：
```
docker compose down -v --remove-orphans
```

## 注意事项

部署过程中，如果遇到tbcapi捕获区块建立索引过慢的情况：使用docker logs tbcapi -f -n 10 通过日志发现每两秒才扫描一个新块
可以采用以下方法解决：

1.使用docker stop tbcapi暂时关闭tbcapi

2.使用docker logs electrumx -f -n 10查看electrumx的同步情况

3.等待electrumx同步到最新区块后，使用docker start tbcapi打开tbcapi开始同步

4.使用docker logs tbcapi -f -n 10查看tbcapi的同步情况

5.等待tbcapi同步到最新区块后，可对外提供服务以及自行测试

该问题只会在从头部署轻节点时出现，后续重启，升级都不会出现这个问题。

其它问题：

1.目前TBCAPI重启需要重新建立数据库，比较浪费时间（大约1个小时）

该问题我们会在后续tbcapi功能更新中一并解决，也会发布新的镜像；该问题不会影响tbcapi本身的服务质量，只是会造成tbcapi同步时间过长，不会造成功能性故障。

2.electrumx重启会导致重启前的数据库中的时间和高度信息被覆盖为重启前的最新高度和时间，

因此重启electrumx需要将持久化数据删除，让其从头建立数据库，可规避对先前数据库的修改。

该问题后续也会改正，目前的工作主要集中在tbcapi上的性能优化，electrumx需要重启的次数很少。



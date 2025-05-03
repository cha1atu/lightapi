# TBC服务套件部署说明书

<div align="center">
  <a href="README.md">English</a> | <a href="README_zh.md">中文</a>
</div>

## 部署步骤

### 准备工作

1. 确保已安装Docker和Docker Compose
   ```
   docker --version
   docker compose --version
   ```

2. 创建项目目录结构
   ```
   mkdir -p tbc-deploy
   cd tbc-deploy
   ```

3. 解压docker compose.yml和init.sql文件到当前目录

### 登录账户
由于仓库是私有仓库，需要登录后才能拉取镜像
1. 登录账户
   ```
   docker login -u chalatu
   ```
2. 输入TBC方提供的PAT密码（Personal access tokens） 默认是90天过期
   ```
   dckr_pat_************
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
docker logs tbcnode --tail 100
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

测试API是否可用：
```
curl http://localhost:5000/api/v1/status
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

## 备份数据

TBCNode数据备份：
```
tar -czvf node_backup.tar.gz ./node_data
```

ElectrumX数据备份：
```
tar -czvf electrumx_backup.tar.gz ./electrumx_data
```

MySQL数据备份：
```
docker exec db mysqldump -u root -pTBCdb@#2024Secure! --all-databases > mysql_backup.sql
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

## 配置需求

在AWS上实例类型为c5.xlarge:4H8G规格服务器上可正常运行，
存储占用情况：
（不算docker环境）
镜像文件：3.236+2.603=5.839G

数据库文件:
10G左右

为了能长期使用，至少选择分配256G以上的存储空间。

当前同步区块为884720

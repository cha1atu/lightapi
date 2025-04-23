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

3. 从GitHub下载docker compose.yml和init.sql文件到当前目录


### 启动服务

1. 启动所有服务
   ```
   sudo docker compose up -d
   ```

2. 查看服务启动情况
   ```
   sudo docker compose ps
   ```

## 服务状态检查

### 1. TBCNode检查

进入容器并检查节点状态：
```
sudo docker exec -it tbcnode /bin/bash
alias tbc-cli="/TBCNODE/bin/bitcoin-cli -conf=/TBCNODE/node.conf -datadir=/TBCNODE/node_data_dir"
tbc-cli getinfo
```

查看日志：
```
sudo docker logs tbcnode --tail 100
```

查看区块同步状态：
```
sudo docker exec -it tbcnode /bin/bash
alias tbc-cli="/TBCNODE/bin/bitcoin-cli -conf=/TBCNODE/node.conf -datadir=/TBCNODE/node_data_dir"
tbc-cli getblockcount
```

测试节点是否已同步到一定区块高度：
```
sudo docker exec -it tbcnode /bin/bash -c "alias tbc-cli='/TBCNODE/bin/bitcoin-cli -conf=/TBCNODE/node.conf -datadir=/TBCNODE/node_data_dir' && block_height=\$(tbc-cli getblockcount) && [ \$block_height -gt 1000 ] && echo '节点已同步至少1000个区块: '\$block_height || echo '节点尚未达到最小区块高度: '\$block_height"
```

### 2. ElectrumX检查

查看ElectrumX日志：
```
sudo docker logs electrumx --tail 100
```

查看实时日志：
```
sudo docker logs --tail 100 -f electrumx
```

检查ElectrumX端口监听状态：
```
sudo docker exec -it electrumx netstat -tulpn | grep 50001
```

### 3. 数据库(MySQL)检查

检查数据库容器状态：
```
sudo docker logs db --tail 100
```

连接到数据库：
```
sudo docker exec -it db mysql -u root -pTBCdb@#2024Secure! -e "SHOW DATABASES;"
```

检查数据库表：
```
sudo docker exec -it db mysql -u root -pTBCdb@#2024Secure! -e "USE TBC20721; SHOW TABLES;"
```

### 4. TBCAPI检查

查看API服务日志：
```
sudo docker logs tbcapi --tail 100
```

测试API是否可用：
```
curl http://localhost:5000/api/v1/status
```

## 常见问题排查

### 如果服务无法启动

检查容器错误日志：
```
sudo docker logs [容器名称]
```

### 服务间连接问题

检查网络配置：
```
sudo docker network inspect tbc-network
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
sudo docker compose pull
sudo docker compose up -d
```

更新特定服务：
```
sudo docker compose pull [服务名]
sudo docker compose up -d [服务名]
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
sudo docker compose down
```

停止服务并删除网络（保留数据）：
```
sudo docker compose down --remove-orphans
```

完全清理（慎用，会删除所有数据）：
```
sudo docker compose down -v --remove-orphans
``` 
# TBC Service Suite Deployment Guide

<div align="center">
  <a href="README.md">English</a> | <a href="README_zh.md">中文</a>
</div>

## Deployment Steps

### Preparation

1. Ensure Docker and Docker Compose are installed
   ```
   docker --version
   docker compose --version
   ```

2. Create project directory structure
   ```
   mkdir -p tbc-deploy
   cd tbc-deploy
   ```

3. Download docker compose.yml and init.sql files from GitHub to the current directory


### Start Services

1. Start all services
   ```
   sudo docker compose up -d
   ```

2. Check service startup status
   ```
   sudo docker compose ps
   ```

## Service Status Check

### 1. TBCNode Check

Enter the container and check node status:
```
sudo docker exec -it tbcnode /bin/bash
alias tbc-cli="/TBCNODE/bin/bitcoin-cli -conf=/TBCNODE/node.conf -datadir=/TBCNODE/node_data_dir"
tbc-cli getinfo
```

View logs:
```
sudo docker logs tbcnode --tail 100
```

Check block synchronization status:
```
sudo docker exec -it tbcnode /bin/bash
alias tbc-cli="/TBCNODE/bin/bitcoin-cli -conf=/TBCNODE/node.conf -datadir=/TBCNODE/node_data_dir"
tbc-cli getblockcount
```

Test if node has synced to a certain block height:
```
sudo docker exec -it tbcnode /bin/bash -c "alias tbc-cli='/TBCNODE/bin/bitcoin-cli -conf=/TBCNODE/node.conf -datadir=/TBCNODE/node_data_dir' && block_height=\$(tbc-cli getblockcount) && [ \$block_height -gt 1000 ] && echo 'Node has synced at least 1000 blocks: '\$block_height || echo 'Node has not reached minimum block height: '\$block_height"
```

### 2. ElectrumX Check

View ElectrumX logs:
```
sudo docker logs electrumx --tail 100
```

View real-time logs:
```
sudo docker logs --tail 100 -f electrumx
```

Check ElectrumX port listening status:
```
sudo docker exec -it electrumx netstat -tulpn | grep 50001
```

### 3. Database (MySQL) Check

Check database container status:
```
sudo docker logs db --tail 100
```

Connect to the database:
```
sudo docker exec -it db mysql -u root -pTBCdb@#2024Secure! -e "SHOW DATABASES;"
```

Check database tables:
```
sudo docker exec -it db mysql -u root -pTBCdb@#2024Secure! -e "USE TBC20721; SHOW TABLES;"
```

### 4. TBCAPI Check

View API service logs:
```
sudo docker logs tbcapi --tail 100
```

Test if API is available:
```
curl http://localhost:5000/api/v1/status
```

## Common Troubleshooting

### If Services Fail to Start

Check container error logs:
```
sudo docker logs [container_name]
```

### Service Connection Issues

Check network configuration:
```
sudo docker network inspect tbc-network
```

### Data Persistence Confirmation

Check if data directories are properly created:
```
ls -la ./node_data
ls -la ./electrumx_data
ls -la ./mysqldata
```

## Service Updates

Update all services:
```
sudo docker compose pull
sudo docker compose up -d
```

Update specific service:
```
sudo docker compose pull [service_name]
sudo docker compose up -d [service_name]
```

## Data Backup

TBCNode data backup:
```
tar -czvf node_backup.tar.gz ./node_data
```

ElectrumX data backup:
```
tar -czvf electrumx_backup.tar.gz ./electrumx_data
```

MySQL data backup:
```
sudo docker exec db mysqldump -u root -pTBCdb@#2024Secure! --all-databases > mysql_backup.sql
```

## Service Shutdown and Cleanup

Stop all services:
```
sudo docker compose down
```

Stop services and delete network (preserve data):
```
sudo docker compose down --remove-orphans
```

Complete cleanup (use with caution, will delete all data):
```
sudo docker compose down -v --remove-orphans
``` 
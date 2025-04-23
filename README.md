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

3. Unzip docker compose.yml and init.sql files to the current directory

### Login to Docker Registry
Since the repository is private, you need to login to pull images
1. Login to Docker
   ```
   docker login -u chalatu
   ```
2. Enter the PAT (Personal access token) provided by TBC (valid for 90 days by default)
   ```
   dckr_pat_************
   ```


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


### 2. ElectrumX Check

View ElectrumX logs:
```
sudo docker logs electrumx --tail 100
```

View real-time logs:
```
sudo docker logs --tail 100 -f electrumx
```


### 3. TBCAPI Check

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

## System Requirements

The service can run normally on an AWS c5.xlarge instance (4 vCPUs, 8GB RAM).
Storage space required for docker images and database files (excluding the Docker environment):
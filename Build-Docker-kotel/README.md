# SvnAdmin V2.0 修改版本 Docker 镜像

## 这个镜像是根据 witersencom 大佬的 SvnAdminV2.0 项目修改而来

## 镜像修改内容

- 支持多架构：amd64 和 arm64
- 基于 Rocky Linux 10 构建
- 重新修改启动脚本，对 /home/svnadmin 目录进行自动判断，不在需要手动复制、授权
- 界面优化调整

### 具体更改 [请点击我](https://github.com/xuewuerduo/svnadmin2-build/commits/master/)

## 镜像标签

- `latest` - 最新版本
- `版本号` - 指定版本 (例如: `2.5.9`)
- `日期` - 按构建日期标记 (例如: `20250827`)

## 使用方法

### 基本运行
```bash 
docker run -d --name svnadmin-kotel \
  -p 80:80 -p 443:443 -p 3690:3690 \
  -v /path/to/svn/data:/home/svnadmin \
  xuewuerduo/svnadmin2-kotel:latest
```
### 使用DockerCompose
创建 `docker-compose.yml` 文件：
```yaml
version: '3.8'
services: 
 svnadmin: 
    image: xuewuerduo/svnadmin2-kotel:latest 
    container_name: svnadmin-kotel 
    restart: unless-stopped 
    ports: 
        - "80:80"
        - "443:443"
        - "3690:3690" 
    volumes: 
        - ./data:/home/svnadmin 
        - ./backup:/home/svnadmin/backup
```
然后运行：
```bash
dockerer-compose up -d
```
## 访问界面
访问 `http://你的IP:80`




# 再次感谢 witersencom大佬！！
-  原仓库地址：[GitHub地址](https://github.com/witersen/SvnAdminV2.0)   [Gitee地址](https://gitee.com/witersen/SvnAdminV2.0)



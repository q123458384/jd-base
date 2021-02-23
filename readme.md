不会再见了。

修改自EV大佬的docker镜像
测试2021.02.23能用

# jd-base
## docker安装方法1↓：
 ```
 docker run -dit \
	-v /安装目录/jd/config:/jd/config \
	-v /安装目录/jd/log:/jd/log \
	-p 5678:5678 \
	-e ENABLE_HANGUP=true \
	-e ENABLE_WEB_PANEL=true \
	--name jd \
	--hostname jd \
	--restart always \
	q123458384/jd:latest
```
如需映射脚本出来直接在上面加一行：
```
-v /安装目录/jd/scripts:/jd/scripts \
```
如你是旁路由，请把-p 5678:5678 \替换成--network host \

## docker-compose 安装方方法：

1、新建一个 `jd` 文件夹,并在jd文件夹在`config`和`log`文件夹
2、在`jd`文件夹下新建`docker-compose.yml` 输入以下内容保存
```
version: "2.0"
services:
  # 第1个容器
  jd1:
    image: q123458384/jd           
    container_name: jd1
    restart: always
    tty: true
    network_mode: "bridge"
    hostname: jd1
    volumes:
      - ./jd1/config:/jd/config
      - ./jd1/log:/jd/log
      #- ./jd1/scripts:/jd/scripts    # 如果想要看到lxk0301大佬的js脚本，以方便的添加额外脚本，可以解除本行注释，下同
    ports:
      - 5678:5678
    environment: 
      - ENABLE_HANGUP=true            # 是否在启动容器时自动启动挂机程序
      - ENABLE_WEB_PANEL=true         # 是否在启动容器时自动启动控制面板

  # 第2个容器
  jd2:
    image: q123458384/jd 
    container_name: jd2
    restart: always
    tty: true
    network_mode: "bridge"
    hostname: jd2
    volumes:
      - ./jd2/config:/jd/config
      - ./jd2/log:/jd/log
    ports:
      - 5672:5678
    environment: 
      - ENABLE_HANGUP=true            # 是否在启动容器时自动启动挂机程序
      - ENABLE_WEB_PANEL=true         # 是否在启动容器时自动启动控制面板

  # 第3个容器，以此类推
  jd3:
    image: q123458384/jd 
    container_name: jd3
    restart: always
    tty: true
    network_mode: "bridge"
    hostname: jd3
    volumes:
      - ./jd3/config:/jd/config
      - ./jd3/log:/jd/log
    ports:
      - 5683:5678
    environment: 
      - ENABLE_HANGUP=true            # 是否在启动容器时自动启动挂机程序
      - ENABLE_WEB_PANEL=true         # 是否在启动容器时自动启动控制面板
```

3、在jd文件在运行 docker-compose up -d


如果您是第一次安装，请等待1-2分钟后执行：docker exec -it jd bash git_pull。

```
# 你要知道的命令↓
1. 手动 git pull 更新脚本
```shell
docker exec -it jd bash git_pull
```
2. 手动删除指定时间以前的旧日志
```shell
docker exec -it jd bash rm_log
 ```
3. 手动导出所有互助码
```shell
docker exec -it jd bash export_sharecodes
```
4. 手动启动挂机程序（**容器会在启动时立即启动挂机程序，所以你想重启挂机程序，你也可以重启容器，而不采用下面的方法。**）
```shell
docker exec -it jd bash jd hangup
```
5. 手动执行薅羊毛脚本，用法如下(其中`-it`后面的`jd`为容器名，`bash`后面的`jd`为命令名，`xxx`为lxk0301大佬的脚本名称)，不支持直接以`node xxx.js`命令运行：
```
docker exec -it jd bash jd xxx      # 如果设置了随机延迟并且当时时间不在0-2、30-31、59分内，将随机延迟一定秒数
docker exec -it jd bash jd xxx now  # 无论是否设置了随机延迟，均立即运行
```
6. Copy自定义脚本到容器目录
```shell
docker cp /宿主机上脚本存放路径/test.js jd:/jd/scripts
```
7. 查看创建日志
```shell
docker logs -f jd
```
8. 重置WEB面板密码
```shell
docker exec -it jd bash jd resetpwd
```
9. 查看挂机脚本日志
```shell
docker exec -it jd pm2 monit`或`docker exec -it jd pm2 logs
```
10. 重启容器
```
docker restart jd
```
11. 如何自动更新容器
```
docker run -d \
    --name watchtower \
    --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --cleanup
```
12
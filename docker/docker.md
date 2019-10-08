#### 安装

<https://www.cnblogs.com/myzony/p/9071210.html>

##### GUI管理配置

推荐使用 Portainer 作为容器的 GUI 管理方案。

官方地址：<https://portainer.io/install.html>

安装命令：

```
docker volume create portainer_data
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

访问你的 IP:9000 即可进入容器管理页面。



#### 容器

#### 镜像
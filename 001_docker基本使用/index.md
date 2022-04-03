# Docker 基本使用


## Docker 容器使用

## Docker 镜像使用

1. 镜像列表

获取镜像列表命令如下所示：

```bash
docker images
```

## Docker 容器连接

## Docker 仓库管理

1. 登录和退出

从某些私有仓库拉取 Docker 镜像，可能需要进行登录。

登录命令如下：

```bash
docker login url
```

登陆成功后，密码会被加密保存在 `~/.docker/config.json` 文件中。

退出命令如下：

```bash
docker logout
```

2. 拉取镜像

```bash
docker pull image_name:tag
```

如果没有指定 tag，默认为 latest。

3. 推送镜像

```bash
docker push username/image_name:tag
```



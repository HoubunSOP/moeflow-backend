# 萌翻[MoeFlow]后端项目

此仓库添加了通过团队中的权限分配来让指定用户获取到团队中所有项目的对应权限

![image](https://github.com/HoubunSOP/moeflow-backend/assets/69132853/66862e1b-c8d5-45d3-bf06-8ef6fe9f8d4f)

## 如何使用

以docker部署为例，首先您先要下载此仓库(git和下载zip二选一即可)，然后打开服务器中的`docker-compose.yml`，在`moeflow-backend`下的`volumes`添加`./models:/app/app/models`与`./core:/app/app/core`

或者您可以直接将镜像源`${GHCR_DOMAIN}/kozzzx/moeflow-backend:${MOEFLOW_BACKEND_VERSION}`改成`ghcr.io/houbunsop/moeflow-backend:main`

但是这仅限于新系统，如果需要从旧数据库进行操作的话还是需要根据下方进行操作更新数据库

具体如下

```yaml
  moeflow-backend:
    image: ${GHCR_DOMAIN}/kozzzx/moeflow-backend:${MOEFLOW_BACKEND_VERSION}
    restart: unless-stopped
    volumes:
      - ./storage:/app/storage
      - ./logs/moeflow-backend:/app/logs
      - ./models:/app/app/models
      - ./core:/app/app/core

```

然后将仓库中`app/models`和`app/core`上传到`docker-compose.yml`的同级目录下

接下来如果您不是第一次启动的话，请打开`docker-compose.yml`的同级目录下的`core/rbac.py`

将`if cls.objects().count() == 0:`更改成`if cls.objects().count() != 0:`来更新权限表

https://github.com/HoubunSOP/moeflow-backend/blob/919c4c3155feffb3b43886f82e2e25ba8a4d2c17/app/core/rbac.py#L165-L180

然后重新运行`docker compose up -d`即可

## 一些小建议

如果感觉前端加载速度过慢，最好删除`docker-compose.yml`中的下面内容

```yaml
  moeflow-frontend:
    image: ${GHCR_DOMAIN}/kozzzx/moeflow-frontend:${MOEFLOW_FRONTEND_VERSION}
    restart: unless-stopped
    volumes:
      - ./nginx/templates:/etc/nginx/templates
      - ./nginx/certificates:/certificates
      - ./storage:/storage
      - ./build:/build
    ports:
      - "${HTTP_PORT}:80"
      - "${HTTPS_PORT}:443"
    environment:
      DOMAIN: ${DOMAIN}
      MAX_CONTENT_LENGTH_MB: ${MAX_CONTENT_LENGTH_MB}
    networks:
      - default
```

然后自己编译一个前端平台或者丢到其他的构建平台来保证前端加载速度（毕竟这个的话还占端口）

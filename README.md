# Nginx With TLS1.3

[![GitHub stars](https://img.shields.io/github/stars/khs1994-website/tls-1.3.svg?style=social&label=Stars)](https://github.com/khs1994-website/tls-1.3)  [![GitHub release](https://img.shields.io/github/release/khs1994-website/tls-1.3.svg)](https://github.com/khs1994-website/tls-1.3/releases) [![Docker Stars](https://img.shields.io/docker/stars/khs1994/nginx.svg)](https://store.docker.com/community/images/khs1994/nginx/) [![Docker Pulls](https://img.shields.io/docker/pulls/khs1994/nginx.svg)](https://store.docker.com/community/images/khs1994/nginx/)

## `Docker Compose`

```yaml
version: "3"
services:

  nginx:
    image: "khs1994/nginx:1.13.9-tls1.3-stretch"
    ports:
      - "80:80"
      - "443:443"  
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./app:/app:rw
      - ./conf.d:/etc/nginx/conf.d:ro
```

## `$ docker run`

```bash
$ docker run -dit \
         -e TZ=Asia/Shanghai \
         -p 80:80 \
         -p 443:443 \
         -v $PWD/app:/app \
         -v $PWD/conf.d:/etc/nginx/conf.d \
         khs1994/nginx:1.13.9-tls1.3-stretch
```

# Who use it?

[khs1994-docker/lnmp](https://github.com/khs1994-docker/lnmp) use this Docker Image.

# More Infortion

* [khs1994-docker/lnmp](https://github.com/khs1994-docker/lnmp)

* [Official NGINX Dockerfiles](https://github.com/nginxinc/docker-nginx)

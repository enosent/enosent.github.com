---
layout: post
title:  "Docker Local Regisry + UI 적용"
date:   2017-02-16 13:45:41 +0900
categories: docker
---

## Docker

### * Docker Local Regisry + UI 적용 테스트
---

1) Docker Local Registry

https://docs.docker.com/registry/

위 링크에 나온데로 했으나 .. 아래와 같은 오류가 발생하면서 동작을 안하다.

```
Error response from daemon: Get https://[ IP ]:5000/v1/_ping: http: server gave HTTP response to HTTPS client
```

원인은 Docker의 push 커맨드는 HTTPS로 진행되는데, 설치한 docker-registry가 HTTP만 지원해서 그렇다.
push를 HTTP로 할 수 있도록 docker 설정을 변경해 주면 된다.

* HTTP를 이용한 Local Registry 구성을 위한 설정 수정

```sh
$ sudo vi /etc/default/docker

# Use DOCKER_OPTS to modify the daemon startup options.
#DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"

DOCKER_OPTS="--insecure-registry [ IP ]:5000"
```

추가로 Ubuntu 환경에서는 파일을 하나 더 만들어야 한다. 이 파일이 없으면 동작을 안함

https://github.com/docker/docker/issues/28321

```
On Ubuntu 16.04.1 modifications to /etc/default/docker/ seem not to work for insecure registry.
```

```sh
$ sudo vi /etc/docker/daemon.json

{
    "insecure-registries": ["[ IP ]:5000"]
}
```

설정 적용 후 ```sudo service docker stop & start ```

저장소로 사용할 폴더에서 docker 실행

```sh
$ cd ~/.dockerbox/

$ docker run -d -p 5000:5000 --restart=always --name registry -v `pwd`/data:/var/lib/registry registry:2
```

Home에 생성한 .dockerbox를 저장소로 사용

저장소 동작 테스트

```sh
$ docker tag ubuntu [ IP ]:5000/ubuntu

$ docker push [ IP ]:5000/ubuntu

$ ls -al .dockerbox/data/docker/registry/v2/
total 16
drwxr-xr-x 4 root root 4096 Feb 14 11:53 .
drwxr-xr-x 3 root root 4096 Feb 14 11:53 ..
drwxr-xr-x 3 root root 4096 Feb 14 11:53 blobs
drwxr-xr-x 3 root root 4096 Feb 14 11:53 repositories
```

PULL 테스트
- pull 도 push 와 동일하게 http 사용가능하게 설정을 해줘야 한다.

```sh
$ docker pull [ IP ]:5000/ubuntu
```

Mac 의 경우에는 설정에서 추가하면 됨

![텍스트](../img/mac_docker.png)

=============

저장소에 저장된 image 확인을 위해 Web UI 설치

```sh
$ docker run \
  -d \
  -e ENV_DOCKER_REGISTRY_HOST=[ IP ] \
  -e ENV_DOCKER_REGISTRY_PORT=5000 \
  -p 9200:80 \
  konradkleine/docker-registry-frontend:v2
```

---
2) Docker UI

Docker container / image 관리를 터미널이 아닌 UI를 이용해 쉽게 사용할 수 있도록 설치

http://portainer.io/

```sh
$ docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
```

현재 설치된 Docker의 container / image 관리 목적이면 -v 옵션 경로를 반드시 지정해주고 실행해주어야 한다.

# Docker Puppeteer

[![Docker Automated build](https://img.shields.io/docker/automated/wangzezhi/puppeteer.svg)](https://hub.docker.com/r/wangzezhi/puppeteer/)
![Docker Build Status](https://img.shields.io/docker/build/wangzezhi/puppeteer.svg?style=popout-square)
![MicroBadger Size](https://img.shields.io/microbadger/image-size/wangzezhi/puppeteer.svg?style=popout-square)

Running Puppeteer in Docker for fu*king GFW inside use.

[![nodesource/node](http://dockeri.co/image/wangzezhi/puppeteer)](https://hub.docker.com/r/wangzezhi/puppeteer/)

## Background

在 Docker 中运行 Puppeteer 是真的很麻烦，以下是引用 Puppeteer GitHub 官方建议，并提出建议使用的 Dockerfile，但是因为某墙的问题，在构建过程中屡次失败，`Shadowsocket 全局模式` 或 `export http_proxy=http://127.0.0.1:1087;export https_proxy=http://127.0.0.1:1087;` 或 切换源 `sources.list` 都未能构建成功，所以才会有这个 repo.

> Getting headless Chrome up and running in Docker can be tricky. The bundled Chromium that Puppeteer installs is missing the necessary shared library dependencies.
> To fix, you'll need to install the missing dependencies and the latest Chromium package in your Dockerfile

官方建议 [Running Puppeteer in Docker](https://github.com/GoogleChrome/puppeteer/blob/master/docs/troubleshooting.md#running-puppeteer-in-docker)

## Getting Started

### Introduction

本镜像使用

- node:10.0.0
- google-chrome-unstable

>_如果需要指定版本可以 PR_

### Usage

```shell
docker pull wangzezhi/puppeteer
```

`Dockerfile` 例子

```shell

FROM wangzezhi/puppeteer:latest

# workdir
WORKDIR /app

# add all to workdir
ADD . /app

# timezone
RUN \
    rm /etc/localtime && \
    ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# set puppeteer npm download host
ENV PUPPETEER_DOWNLOAD_HOST='https://npm.taobao.org/mirrors'

RUN npm i --only=production --registry=https://registry.npm.taobao.org

# Add user so we don't need --no-sandbox.
RUN groupadd -r pptruser && useradd -r -g pptruser -G audio,video pptruser \
    && mkdir -p /home/pptruser/Downloads \
    && chown -R pptruser:pptruser /home/pptruser \
    && chown -R pptruser:pptruser ./node_modules

# Run everything after as non-privileged user.
USER pptruser

# add your own ENV
ENV PORT=7900

# ENV ...

# expose port to access
EXPOSE 7900

ENTRYPOINT ["dumb-init", "--"]
CMD [ "npm", "start" ]
```
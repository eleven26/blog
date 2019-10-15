---
title: Docker 运行 crontab 示例
date: 2019-10-15 14:07:00
tags: Docker
---

### cron 文件

hello-cron

```
* * * * * echo "Hello world" >> /var/log/cron.log 2>&1
# An empty line is required at the end of this file for a valid cron file.
```

### Dockerfile

```dockerfile
FROM ubuntu:latest
MAINTAINER docker@ekito.fr

RUN apt-get update && apt-get -y install cron

# Copy hello-cron file to the cron.d directory
COPY hello-cron /etc/cron.d/hello-cron

# Give execution rights on the cron job
RUN chmod 0644 /etc/cron.d/hello-cron

# Apply cron job
RUN crontab /etc/cron.d/hello-cron

# Create the log file to be able to run tail
RUN touch /var/log/cron.log

# Run the command on container startup
CMD cron && tail -f /var/log/cron.log
```

### 测试

```shell script
docker build --rm -t ekito/cron-example .
docker run -it ekito/cron-example
```


### 说明

可以使用 `cron -f` 作为 dockerfile 的 `CMD`，这样会强制 crontab 前台运行


### 原文地址:

[how-to-run-a-cron-job-inside-a-docker-container](https://stackoverflow.com/questions/37458287/how-to-run-a-cron-job-inside-a-docker-container)

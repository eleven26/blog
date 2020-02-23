---
title: Minikube 初体验
date: 2020-02-23 09:45:00
tags: [Docker, k8s, Kubernetes]
---

## Minikube 是什么 ？

minikube 相当于一个运行在本地的 Kubernetes 单节点，我们可以在里面创建 Pods 来创建对应的服务。

官网描述: Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

简单来说，就是给你本地开发、测试用的，它在虚拟机里面启用 Kubernetes，这个 Kubernetes 只有一个节点，就是虚拟机本身。 


## 国内使用的一些问题

1. k8s.gcr.io 在 minikube 里面无法正常使用

解决方法：

* 使用阿里云的镜像启动 minikube

```
minikube start --vm-driver=virtualbox --registry-mirror=https://registry.docker-cn.com --image-mirror-country=cn --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```

* 创建服务的时候需要替换镜像地址

```
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10 ## 无法使用
```

这里的 `k8s.gcr.io` 需要替换成 `registry.aliyuncs.com/google_containers`，如下所示

```
kubectl create deployment hello-minikube --image=registry.aliyuncs.com/google_containers/echoserver:1.10
```

2. 拿到了 service 的 url，无法正常打开

这就是因为 service 没有被成功创建，但是 minikube 在运行 `kubectl create deployment` 之类的命令的时候并没有明确的报错，我们可以使用 `minikube logs` 来查看错误日志。

解决了网络问题，其实基本上没什么问题了，访问 `minikube service hello-minikube --url` 得到的 url 可以正常打开了。

```
Hostname: hello-minikube-6db89b59b4-dh6bh

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.1
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://192.168.99.100:8080/

Request Headers:
	accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
	accept-encoding=gzip, deflate
	accept-language=zh-CN,zh;q=0.9,zh-TW;q=0.8,en;q=0.7
	connection=keep-alive
	host=192.168.99.100:31663
	upgrade-insecure-requests=1
	user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36

Request Body:
	-no body in request-
```


## 参考链接：

1. [Installing Kubernetes with Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

2. [Kubernetes 之 Minikube 国内安装](https://www.inlighting.org/2020/install-minikube-in-china.html)

3. [minikube是什么](https://www.cnblogs.com/liyuanhong/p/10143157.html)



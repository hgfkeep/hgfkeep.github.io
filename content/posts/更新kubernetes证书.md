---
title: "更新kubernetes 过期证书"
date: 2020-01-25T16:33:26+08:00
lastmod: 2020-01-25T16:33:26+08:00
draft: false
keywords: [kubernetes]
description: "kubernetes 证书默认是1年的期限，本文描述了证书到期后更新证书的方法， 本方法适用于kubernetes1.15以下版本。后续可以考虑自动证书更新"
tags: [kubernetes]
categories: [kubernetes]
author: "heguangfu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: true
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: <img src="https://licensebuttons.net/l/by-nc-sa/3.0/88x31.png"><br/>感谢阅读，如果有问题请您留言，我会及时改正<br/> 本博客所有原创文章版权归hgf所有，转载请注明出处hgfdodo.win/blog
reward: false
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: true

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

typora-root-url: ../../static
---

# 更新kubernetes 过期证书

kubeadm 版本在kubernetes 1.15 版本 提供了强大的证书管理功能，本文章适用于kubernetes1.15以下版本（文章中kubernetes版本是1.13.2）。

> 1.15 版本的证书管理相关文档：

> 1. [官方文档-使用 kubeadm 进行证书管理](https://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
> 2. [官方文档-kubeadm alpha 使用说明]9https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-alpha/)

查看证书有效期方法：

```shell
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text |grep ' Not '
```

⚠️ kubeadm 默认生成的**ca证书有效期是10年**，其**他证书（如etcd证书，apiserver证书）有效期均为1年**。

## 更新证书和配置

整体思路：

1. 备份：在进行证书更新前，**建议备份`/etc/kubernetes`**，防止操作失误。
2. 更新证书：使用`kubeadm alpha certs renew`重新生成证书。*仅更新`***.key`文件，需要原始的crt文件才能生成对应的key文件。*
3. 更新配置：使用`kubeadm init phase kubeconfig all --config ${kubeadm.yaml配置文件}`或者`kubeadm alpha kubeconfig user` 命令。

⚠️ 不同版本的kubeadm对于证书renew的命令有细微的差异，具体情况需要依据已经安装的kubeadm来判断。通过命令行`kubeadm alpha certs renew --help`输出类似如下信息：
![image.png](https://i.loli.net/2020/02/05/a94nSIQRHYiorU2.png)

证书更新策略：

* 单主节点：可以直接运行`kubeadm alpha certs renew all --config kubeadm.yaml`完成证书更新。然后替换kubelet配置
* 多主节点：建议使用原ca证书（有效期10年），每个组件（etcd、apiserver 等）单独更新。

## 多master节点证书更新

### 备份原始配置和证书

所有master节点运行命令：`cp -r /etc/kubernetes /home/heguangfu/kubernetes`

### 更新证书

所有master节点依次完成如下命令：

* etcd 心跳证书：`kubeadm alpha certs renew  etcd-healthcheck-client --config kubeadm-config.ict15.yaml`
* etcd peer证书：`kubeadm alpha certs renew  etcd-peer --config kubeadm-config.ict15.yaml`
* etcd server证书：`kubeadm alpha certs renew  etcd-server --config kubeadm-config.ict15.yaml`
* front-proxy-client 证书：`kubeadm alpha certs renew  front-proxy-client  --config kubeadm-config.ict15.yaml`
* apiserver-etcd-client 证书`kubeadm alpha certs renew  apiserver-etcd-client  --config kubeadm-config.ict15.yaml`
* apiserver-kubelet-client 证书`kubeadm alpha certs renew  apiserver-kubelet-client   --config kubeadm-config.ict15.yaml`
* apiserver 证书`kubeadm alpha certs renew  apiserver    --config kubeadm-config.ict15.yaml`

> ⚠️ 不同的master节点使用的kubeadm配置有细微的差异，执行更新证书是，每个master在`--config`后面使用原来集群创建时，当前master对应的kubeadm配置文件。


### 更新配置

所有master节点，在**更新完证书后**，使用`kubeadm init phase kubeconfig all --config ${kubeadm.yaml配置文件}`更新kubernetes 配置


### 验证集群状态

清理前次的kubectl权限信息：`rm -rf $HOME/.kube`。

重新配置kubectl权限信息：

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

1. 验证etcd：查看etcd中某个节点的docker日志，日志中所有etcd peer均active且加入到同一个集群
2. 验证kubernetes 集群：运行`kubectl cluster-info`和`kubectl get nodes` 符合预期。
3. 确性kubernetes 系统相关的服务运行正常(核心是`kube-apiserver`,`kube-controller-manager`,`kube-proxy`, `kube-flannel`)：`kubectl get pods -n kube-system`
4. 检查pod的运行状态：`kubectl get pods --all-namespaces`。

## 可能的问题

1. `Part of the existing bootstrap client certificate is expired: 2020-01-19 15:10:17 +0000 UTC`：**确认全部证书更新，并且证书更新好后，更新了kubernetes配置**
2. api server日志：`Unable to authenticate the request due to an error: [x509: certificate has expired or is not yet valid, x509: certificate has expired or is not yet valid]`。可能原因有：证书过期；证书部分更新；master上包含了代理配置，导致对apiserver的请求走了代理，证书认证通不过（运行`unset http_proxy;unset ftp_proxy;unset socks_proxy;unset https_proxy`，取消代理配置）。



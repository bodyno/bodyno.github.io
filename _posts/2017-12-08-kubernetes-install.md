---
layout: post
title: "kubernetes高可用集群国内快速安装"
date: 2017-12-08
tags: [kubernetes]
categories: programe
---

最近一直在学习Kubernetes相关的知识，从Deployment到Service到Ingress，监控到日志到分布式追踪到服务网格，这一套基本上都是打通了。

但有一个问题是很痛苦的，就是如何在国内公有云上搭建一个高可用的Kubernetes集群呢？

如果机器是在国外，可以很方便的使用[kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)（非高可用），[kops](https://github.com/kubernetes/kops)，[kubespray](https://github.com/kubernetes-incubator/kubespray)等快速安装kubernetes集群

但是，如何在没有科学上网的情况下安装呢？

哈哈，很简单，已经有哥们帮我们做好了。

(k8s-install-scripts)[https://github.com/zhuchuangang/k8s-install-scripts]这个将yum包下在本地，同时使用阿里云的docker源来进行安装，在国内的安装速度可谓不要不要，如果需要安装高可用版本，就选择kubespray方式安装就好了。

嗯，这里就是简单分享一个这个项目。

如果大家有关于Kubernetes相关的问题要讨论，欢迎骚扰。
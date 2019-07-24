# 安装 kuboard

## 前提

安装 kuboard 时，假设您已经：

* 已经有一个 kubernetes 集群
* 拥有对该 kubernetes 集群执行 kubectl 命令时的所有权限

如果没有 kubernetes 集群，可以有如下选项：

* 通过 阿里云 创建 kubernetes 容器服务，并获得和配置 kubectl 的访问参数
* 参考 [安装 kubernetes 用于测试](install-k8s)
* 或参考 [安装 kubernetes 高可用](install-kubernetes)

[领取阿里云最高2000元红包](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=obezo3pg)



## 兼容性



| Kubernetes 版本 | Kuboard 版本   | 兼容性 | 说明                                                         |
| --------------- | -------------- | ------ | ------------------------------------------------------------ |
| v1.15           | v1.0.0-beta.10 | <span style="font-size: 24px;">😄</span>      | Kuboard作者所使用的Kubernetes版本                            |
| v1.14           | v1.0.0-beta.10 | <span style="font-size: 24px;">😄</span>      | Kuboard作者所使用的Kubernetes版本                            |
| v1.13           | v1.0.0-beta.10 | <span style="font-size: 24px;">🤔</span>      | 理论上可以，尚未听到用户反馈兼容性问题                       |
| v1.12           | v1.0.0-beta.10 | <span style="font-size: 24px;">😐</span>      | Kubernetes Api 尚不支持 dryRun，<br />忽略Kuboard在执行命令式的参数校验错误，可正常工作 |
| v1.11           | v1.0.0-beta.10 | <span style="font-size: 24px;">😐</span>      | 同上                                                         |



**Kubernetes 安装方式**

> * 部分用户使用二进制包的形式安装 Kubernetes，Kuboard 现在的版本不能在这类 Kubernetes 集群中正常工作，作者正在解决此问题。
> * 如果您是使用 kubeadm 安装的 Kubernetes 集群（Kubernetes 官方推荐的安装方式），请放心使用 Kuboard。
> * Kubeadm 相关资料请参考 https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/



## 安装

**获取并修改yaml文件**

```bash
wget https://raw.githubusercontent.com/eip-work/eip-monitor-repository/master/dashboard/kuboard.yaml
```

修改文件 kuboard.yaml 中 Ingress 的 host 为 kuboard.yourclustername.yourdomain.com

**执行安装**

```bash
kubectl apply -f kuboard.yaml 
```

## 获取 token

### 获取管理员用户 token

**拥有的权限**

此Token拥有 ClusterAdmin 的权限，可以执行所有操作

```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}')   
```

执行完该命令后，可获得类似如下的输出：

```
Name: admin-user-token-g8hxb
Namespace: kube-system
Labels: <none>
Annotations: [kubernetes.io/service-account.name](http://kubernetes.io/service-account.name): kuboard-user
[kubernetes.io/service-account.uid](http://kubernetes.io/service-account.uid): 948bb5e6-8cdc-11e9-b67e-fa163e5f7a0f

Type: [kubernetes.io/service-account-token](http://kubernetes.io/service-account-token)

Data
====
ca.crt: 1025 bytes
namespace: 11 bytes
token: eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWc4aHhiIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI5NDhiYjVlNi04Y2RjLTExZTktYjY3ZS1mYTE2M2U1ZjdhMGYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.DZ6dMTr8GExo5IH_vCWdB_MDfQaNognjfZKl0E5VW8vUFMVvALwo0BS-6Qsqpfxrlz87oE9yGVCpBYV0D00811bLhHIg-IR_MiBneadcqdQ_TGm_a0Pz0RbIzqJlRPiyMSxk1eXhmayfPn01upPdVCQj6D3vAY77dpcGplu3p5wE6vsNWAvrQ2d_V1KhR03IB1jJZkYwrI8FHCq_5YuzkPfHsgZ9MBQgH-jqqNXs6r8aoUZIbLsYcMHkin2vzRsMy_tjMCI9yXGiOqI-E5efTb-_KbDVwV5cbdqEIegdtYZ2J3mlrFQlmPGYTwFI8Ba9LleSYbCi4o0k74568KcN_w
```


### 获取只读用户的Token

**拥有的权限**

- view  可查看名称空间的内容
- system:node   可查看节点信息
- system:persistent-volume-provisioner  可查看存储类和存储卷声明的信息

**适用场景**

只读用户不能对集群的配置执行修改操作，非常适用于将开发环境中的 kuboard 只读权限分发给开发者，以便开发者可以便捷地诊断问题

执行如下命令可以获得 <span style="color: #F56C6C; font-weight: 500;">只读用户</span> 的 Token

```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-viewer | awk '{print $1}')   
```



## 访问 Kuboard

### 通过域名访问

在浏览器打开链接 http://kuboard.yourclustername.yourdomain.com （使用前面已修改的域名）

输入前一步骤中获得的 token，可进入控制台界面

### 通过 NodePort 访问

kuboard Service 使用了 NodePort 的方式暴露服务，NodePort 为 32567；您可以按如下方式访问 kuboard

```
http://any-of-your-node-ip:32567/
```

> 您也可以修改 kuboard.yaml 文件，使用自己定义的 NodePort 端口号




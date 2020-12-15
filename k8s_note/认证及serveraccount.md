[TOC]



## 外部client连接API server时需要经过什么样的流程呢？



一般来说，clent接入API server需要进行3个步骤：

1、认证，用于身份校验。

2、授权，决定了你能在集群中做什么

3、准入控制，在授权后进行的一些额外的控制（非必选）

#### 认证方式

k8s的认证方式是通过插件式设计的，可以自由配置符合自己的希望的认证方式。常见的认证方式有 token、ssl证。

- token 预共享秘钥

- ssl认证  双方均有证书，需要进行双方的证书认证

####  授权方式

RBAC

## k8s的资源定义

k8s是通过request path来对资源进行定义的。譬如

```html
curl https://ip:6443/apis/apps/v1/namespaces/default/deployment/nignx-deploy

这个用于获取default namespace下的名字为nignx-deploy的deployment
```

实验，正常通过上面的url是比较难测试的，因为是https接口，需要提供证书，可以通过在起一个proxy来代理，实现我们的目的。

```shell
[root@node01 ~]# kubectl proxy
Starting to serve on 127.0.0.1:8001
[root@node01 ~]# http://127.0.0.1:8001/api/v1/namespaces/default
```

![image-20201211172845356](./k8s_note.assets/image-20201211172845356.png)

## 哪些资源需要与API server交互

1、集群之外的：如通过kubectl、各种语言的api调用方式

2、集群之内的：如k8s之上的各种Pod

#### 交互接口

而API server会提供两种接口方式，一个是通过svc方式，另一个是通过endpoints。pod通过svc进行访问而集群外的client通过endpoints来进行访问。

```shell
[root@node01 ~]# kubectl describe svc kubernetes
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         10.30.250.37:6443
Session Affinity:  None
Events:            <none>
```

**注：k8s自签证书需要包含这两个地址的认证。**

#### 交互用户

- 外部用户，也就是我们平时用的认证用户:  clusterrole
- 内部用户，也就是pod使用的用户:  serviceaccount

```shell
# 每个pod中都有一个默认的用户
[root@node01 ~]# kubectl describe pod app-7d5d74889d-t6fsf
...
Volumes:
  default-token-xmr79:  -- 这个为默认的token认证方式的secret
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-xmr79
    Optional:    false
...
[root@node01 ~]# kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-xmr79   kubernetes.io/service-account-token   3      136d
[root@node01 ~]# kubectl describe secret default-token-xmr79
Name:         default-token-xmr79
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 4e0fcd7c-d09e-11ea-9b34-fa163e069419

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4teG1yNzkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjRlMGZjZDdjLWQwOWUtMTFlYS05YjM0LWZhMTYzZTA2OTQxOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.XRTzFv5oGJcwQsNjCToIjwix1jyi4Jldnsc2az7FabQbVYzXummKOqX8CCX1umAIwIWzF1ylOnwui10cXmykPHmf26nol-1TarjELe_oHU_OqcYM-gtjFG0qEpHeccCJNwLHl91pW4UecoBe6zIhsVZxCHSwzIUIsjFr1T22L_mN2vZF4Rf2cdql8sfkynPl7yEiIiHo8btds1QhRex8o903CBfl3MyJJ68JYAAQjWUge4jyF85_ZIh68dOyvBrbbwGPhCgcHv9Y7fRe0QE_GuoX7QcWoraSZgj4ROlemis112vZ-GOpqBWESh_oZryAvYd4UZ7bFTNmUbHQW6kVAg
```

#### 创建serviceaccount用户

```shell
# 创建一个serviceaccount，仅仅只是创建一个账号，还不涉及授权问题
[root@node01 ~]# kubectl create serviceaccount lintao
serviceaccount/lintao created
[root@node01 ~]# kubectl get serviceaccount
NAME      SECRETS   AGE
default   1         136d
lintao    1         2m27s
[root@node01 ~]# kubectl describe serviceaccount lintao
Name:                lintao
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   lintao-token-ktvnb
Tokens:              lintao-token-ktvnb  -- 绑定了secret
Events:              <none>
[root@node01 ~]# kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-xmr79   kubernetes.io/service-account-token   3      136d
lintao-token-ktvnb    kubernetes.io/service-account-token   3      30s   ---这里自动生成的secret

# debug命令，创建一个serviceaccount，尝试创建 --dry-run
[root@node01 ~]# kubectl create serviceaccount  mytest -o yaml --dry-run
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: null
  name: mytest

# 其他资源也可以通过export来导出yaml文件，方便资源创建
[root@node01 ~]# kubectl get pod app-7d5d74889d-t6fsf -o yaml  --export
```

## kubeconfig解析

- cluster：定义集群信息，包括集群名，连接地址，认证证书
- user: 定义认证账号信息，可以使用证书认证、token认证或账号密码认证
- context：定义上下文信息，具体指使用哪个账号去连接哪个集群的定义

```shell
# 查看目前集群的连接信息
[root@node01 ~]# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.30.250.37:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

# 创建一个认证账号，使用账号密码认证
[root@node01 ~]# kubectl config set-credentials lintao-create --username=lintaoadmin --password=uXFGweU9l35qcif
User "lintao-create" set.

# 创建一个context
[root@node01 ~]# kubectl config set-context lintao-create --cluster=kubernetes --user=lintao-create
Context "lintao-create" created.
```


# 什么是RBAC

k8s使用RBAC的授权方式，前面我们讲了认证方式，接下来讨论一下k8s的授权方式。认证只是让用户能够连上集群，如果没有授权，那么用户什么都做不了。什么是RBAC呢？RBAC全称为Role-Based AC，是通过定义role，将资源授权给role，然后再将role跟用户或者组进行绑定从而实现授权的一种形式。

# Rolebinding

这是namespace级别的权限绑定，可以将useraccount或者serviceaccount绑定到role。

```shell
# 创建role,这里只授权了get和list的权限
[root@node01 roledefine]# kubectl create role pod-reader --verb=get,list --resource=pods
role.rbac.authorization.k8s.io/pod-reader created

# 进行rolebinding,这里lintao的用户是提前用证书进行创建的
[root@node01 ~]# kubectl create rolebinding lintao-read-pod-bingind --role=pod-reader --user=lintao --dry-run -o yaml > lintao-pod-reader-rolebinding.yaml
[root@node01 ~]# kubectl apply -f lintao-pod-reader-rolebinding.yaml


```

# ClusterRolebinding

clusterrolebinding是属于集群级别的，其生效范围是整个集群。创建的方式类似于rolebinding，这里就不进行累赘了。需要注意的是，通过rolebinding可以绑定clusterrole，而clusterrolebinding不能绑定role。

前者的好处是，假若有10个namespace都需要定义类似的role权限，一般来说就需要定义10个不同namespaace并分别绑定到不同的用户中，而如果使用clusterrole的话，只需要定义一个，10个用户均用rolebinding到clusterrole即可。


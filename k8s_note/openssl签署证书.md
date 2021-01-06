# 如何通过openssl生成证书及签署



#### 各种后缀文件解析

- key结尾文件：这个属于私钥，用于进行加密及解密的秘钥
- csr结尾文件：使用私钥进行加密的证书
- crt结尾文件：通过CA的私钥进行签署的证书，是经过认证的

#### 生成个人key

```shell
# 无需密码
openssl genrsa -out lintao.key 2048

```

使用私钥key来生成证书

```shell
openssl req -new -key lintao.key -out lintao.csr -subj "/CN=lintao"  ---CN 一定要指定为你你的账号信息
```



使用CA的key来进行证书签署

```shell
openssl x509 -req -in lintao.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out lintao.crt -days 3560
```

# 证书查看

#### 查看key内容

```shell
openssl rsa -noout -text -in /tmp/lintao.key
```

#### 查看证书csr内容

```shell
openssl req -noout -text -in  /etc/kubernetes/pki/lintao.csr
```

#### 查看签署证书crt内容

```shell
openssl x509 -noout -text -in /tmp/lintao.crt
```

# 附录

另外关于ssl/tls，加密算法等可以参考这个文章：https://www.jianshu.com/p/5170199a1db8
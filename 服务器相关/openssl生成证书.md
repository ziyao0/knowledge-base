# openssl生成证书

[TOC]

## 一 基于openssl x509生成证书

### 1 生成CA相关文件

```sh
#生成私钥
openssl genrsa -des3 -out ca.key 2048
#生成ca证书
 openssl req -new -key ca.key -out ca.csr
 #生成crt
 openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
```

### 2 配置文件

`openssl.cnf`

```sh
[ CA_default ]
copy_extensions = copy
 
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
 
[req_distinguished_name]

C = CN

ST = beijing

L = beijing

O = eason

OU = eason

CN = rancher.eason.com
 
[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
 
[alt_names]

DNS.1 = *.rancher.eason.com

DNS.2 = *.eason.com
```

### 3 生成服务器端证书

~~~sh
mkdir server client
# 
openssl genpkey -algorithm RSA -out server/server.key
#
openssl req -new -nodes -key server/server.key -out server/server.csr -config openssl.cnf -extensions 'v3_req'
# 生成服务端证书
openssl x509 -req -in server/server.csr -out server/server.crt -CA ca.crt -CAkey ca.key -CAcreateserial -extfile ./openssl.cnf -extensions 'v3_req' -days 3650
~~~

### 4 生成客户端证书

~~~sh
#生成客户端私钥
openssl genpkey -algorithm RSA -out client/client.key
# 
openssl req -new -nodes -key client/client.key -out client/client.csr -config openssl.cnf -extensions 'v3_req'
#生成客户端证书
openssl x509 -req -in client/client.csr -out client/client.pem -CA ca.crt -CAkey ca.key -CAcreateserial -extfile ./openssl.cnf -extensions 'v3_req' -days 3650
~~~

### 5 验证证书

~~~sh
# 查看证书过期时间
openssl x509 -in server.pem -noout -dates
~~~


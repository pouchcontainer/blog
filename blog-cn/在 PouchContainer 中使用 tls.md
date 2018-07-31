# 使用TLS建立安全的pouchd连接
如果连接到pouchd的客户端来自同一服务器，则只使用unix socket用参数-l unix:///var/run/pouchd.sock启动pouchd即可.如果要通过远程服务器上的http客户端连接pouchd，则pouchd应该监听tcp端口，例如：-l 0.0.0.0:4243，在这种情况下，如果我们不进行TLS保护的话，pouchd将不会对连接的远程身份加以校验，从而接受所有的连接，这在生产环境中绝对是非常危险的。

为了校验客户端身份，我们将对pouchd和客户端进行CA认证。

## 创建 CA

如下所示，使用通用名字 pouch_test 来创建一个CA。

```shell
openssl req -new -newkey rsa:2048 -days 3650 -nodes -x509 -subj "/C=CN/ST=ZheJiang/L=HangZhou/O=Company/OU=Department/CN=pouch_test" -keyout ca-key.pem -out ca.pem
```

执行上述命令后，在当前目录文件有两个名叫ca.pem和ca-key.pem的文件，就是CA。切记一定要保密。

## 创建证书以守护进程

为pouchd创建一个证书，这个证书只能作为服务器证书使用，HOSTNAME是运行pouchd的机器的变量名。

```shell
name=$HOSTNAME
mkdir -p ${name}
# create a key
/usr/bin/openssl genrsa -out ${name}/key.pem 2048
# create a csr
/usr/bin/openssl req -subj "/C=CN/ST=ZheJiang/L=HangZhou/O=Company/OU=Department/CN=${name}" -new -key ${name}/key.pem -out ${name}/$name.csr
# generate a certificate
/usr/bin/openssl x509 -req -days 3650 -in ${name}/${name}.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out ${name}/cert.pem -extfile <(echo "extendedKeyUsage = serverAuth")
cp ca.pem ${name}/ca.pem
```

执行上述命令后，我们可以看到一个包含创建pouchd的TLS保护所需所有依赖文件的目录，一般用如下命令来设置启动pouch（变量用正确的值替换掉即可）：

```shell
--tlsverify --tlscacert=${name}/ca.pem --tlscert=${name}/cert.pem --tlskey=${name}/key.pem
```

此时，pouchd已经拥有了tls保护，当启动pouchd后，它监听的tcp地址只能与使用相同CA证书的客户端建立连接。

## 创建客户端证书

```shell
name=a_client
mkdir -p ${name}
# create a key
/usr/bin/openssl genrsa -out ${name}/key.pem 2048
# create a csr
/usr/bin/openssl req -subj "/C=CN/ST=ZheJiang/L=HangZhou/O=Company/OU=Department/CN=${name}" -new -key ${name}/key.pem -out ${name}/$name.csr
# generate a certificate
/usr/bin/openssl x509 -req -days 3650 -in ${name}/${name}.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out ${name}/cert.pem -extfile <(echo "extendedKeyUsage = clientAuth")
cp ca.pem ${name}/ca.pem
```

执行上述命令后，我们可以看到一个包含建立pouchd的TLS保护所有依赖文件的目录，一般用以下命令来设置启动pouch（用正确的值替换掉变量name即可）：

```shell
--tlsverify --tlscacert=${name}/ca.pem --tlscert=${name}/cert.pem --tlskey=${name}/key.pem
```

然后我们可以看到一个包含使用PouchContainer客户端所有依赖文件的目录，例如：将此证书用作身份验证来获取pouchd service的版本。

```
./pouch -H ${server_hostname}:4243 --tlsverify --tlscacert=${path}/ca.pem --tlscert=${path}/cert.pem --tlskey=${path}/key.pem version
```

当一个没有证书的客户端或者是一个不是由同一CA发布的证书的客户端想要尝试连接具有TLS保护的pouchd时，连接将被拒绝。

查看原英文文档，请[点击这里](https://github.com/alibaba/pouch/blob/master/docs/features/pouch_with_tls.md)




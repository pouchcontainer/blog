# 使用TLS保护pouchd

当客户端仅通过同一个服务器连接pouchd时，只应该使用unix socket。这意味着需要通过参数 `-l unix:///var/run/pouchd.sock` 启动pouchd。如果pouchd通过远程服务器上的http客户端连接，它应该监听tcp端口，例如：`-l 0.0.0.0：4243`。在这种情况下，如果我们不进行tls保护，pouchd将忽略远程的身份并接受任何连接，这在生产环境中绝不安全。

为了验证客户端身份，应该创建CA来为pouchd和用户生成证书。

## 创建CA

创建通用名称为pouch_test的CA，具体如下：

```shell
openssl req -new -newkey rsa:2048 -days 3650 -nodes -x509 -subj "/C=CN/ST=ZheJiang/L=HangZhou/O=Company/OU=Department/CN=pouch_test" -keyout ca-key.pem -out ca.pem
```

应用上述指令后，当前目录下会出现两个文件，ca.pem和ca-key.pem。这两个文件就是CA，需要对它们保密。

## 创建证书来保护进程

为pouchd创建一个证书，它仅能用作为服务器证书。变量名为运行pouchd机器的主机名。 

```shell
name=$HOSTNAME
mkdir -p ${name}
# 创建key
/usr/bin/openssl genrsa -out ${name}/key.pem 2048
# 创建csr
/usr/bin/openssl req -subj "/C=CN/ST=ZheJiang/L=HangZhou/O=Company/OU=Department/CN=${name}" -new -key ${name}/key.pem -out ${name}/$name.csr
# 生成证书
/usr/bin/openssl x509 -req -days 3650 -in ${name}/${name}.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out ${name}/cert.pem -extfile <(echo "extendedKeyUsage = serverAuth")
cp ca.pem ${name}/ca.pem
```

应用上述指令后，我们有了一个目录，其中包含了所有我们需要用来设置pouchd的tls保护的文件，以此来用类似如下的配置开始pouchd（用正确的值替换变量）：

```shell
--tlsverify --tlscacert=${name}/ca.pem --tlscert=${name}/cert.pem --tlskey=${name}/key.pem
```

当pouchd使用这些tls配置启动时，它监听的tcp地址只能由使用同一CA发布证书的客户端连接。

## 创建客户端证书

```shell
name=a_client
mkdir -p ${name}
# 创建key
/usr/bin/openssl genrsa -out ${name}/key.pem 2048
# 创建csr
/usr/bin/openssl req -subj "/C=CN/ST=ZheJiang/L=HangZhou/O=Company/OU=Department/CN=${name}" -new -key ${name}/key.pem -out ${name}/$name.csr
# 生成证书
/usr/bin/openssl x509 -req -days 3650 -in ${name}/${name}.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out ${name}/cert.pem -extfile <(echo "extendedKeyUsage = clientAuth")
cp ca.pem ${name}/ca.pem
```

应用上述指令后，我们有了一个目录，其中包含了所有我们需要用来设置pouchd的tls保护的文件，以此来用类似如下的配置来开始pouchd（用正确的值替换变量）：

```shell
--tlsverify --tlscacert=${name}/ca.pem --tlscert=${name}/cert.pem --tlskey=${name}/key.pem
```

然后我们有了一个目录，包含了所有我们需要的文件来使用PouchContainer客户端。例如：使用此证书作为身份证明来获取pouchd服务版本：

```
./pouch -H ${server_hostname}:4243 --tlsverify --tlscacert=${path}/ca.pem --tlscert=${path}/cert.pem --tlskey=${path}/key.pem version
```

当没有证书或未由同一CA发布的证书的客户端尝试连接到具有TLS保护的pouchd时，连接将被拒绝。

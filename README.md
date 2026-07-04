# stunnel Docker image

镜像内置 stunnel、OpenSSL 和系统根证书，可作为客户端或服务端。运行方式由 `stunnel.conf` 决定。

## 构建

```bash
docker build -t local-stunnel .
```

## 导出镜像

```bash
docker save -o local-stunnel.tar local-stunnel
```

## 运行

```bash
docker run --rm \
  -p 8080:8080 \
  -v "$PWD/stunnel.conf:/etc/stunnel/stunnel.conf:ro" \
  local-stunnel
```

如果要挂载域名证书或私有 CA：

```bash
docker run --rm \
  -p 443:443 \
  -v "$PWD/stunnel.conf:/etc/stunnel/stunnel.conf:ro" \
  -v "$PWD/certs:/certs:ro" \
  local-stunnel
```

## 客户端模式

客户端模式由服务段里的 `client = yes` 决定。示例：

```conf
debug = info
output = /dev/stdout

[https-client]
client = yes
accept = 0.0.0.0:8080
connect = example.com:443
verifyChain = yes
CAfile = /etc/ssl/certs/ca-certificates.crt
checkHost = example.com
```

镜像已经内置系统根证书，`CAfile = /etc/ssl/certs/ca-certificates.crt` 可以用于校验公开 CA 签发的域名证书。

## 服务端模式

服务端模式由服务段里的 `client = no` 决定。示例：

```conf
debug = info
output = /dev/stdout

[tls-server]
client = no
accept = 0.0.0.0:443
connect = app:80
cert = /certs/fullchain.pem
key = /certs/privkey.pem
```

把证书文件挂载到 `/certs` 即可。证书通常使用 `fullchain.pem`，私钥使用 `privkey.pem`。

## 自定义根证书

信任内网 CA 或自签 CA，把 `.crt` 文件挂载到 `/usr/local/share/ca-certificates`，并设置 `UPDATE_CA_CERTIFICATES=1`：

```bash
docker run --rm \
  -e UPDATE_CA_CERTIFICATES=1 \
  -v "$PWD/ca:/usr/local/share/ca-certificates:ro" \
  -v "$PWD/stunnel.conf:/etc/stunnel/stunnel.conf:ro" \
  local-stunnel
```

## 配置说明

容器启动脚本会自动注入下面两个全局配置，让 stunnel 在容器前台运行：

```conf
foreground = yes
pid =
```

所以你的 `stunnel.conf` 只需要关注转发服务本身。

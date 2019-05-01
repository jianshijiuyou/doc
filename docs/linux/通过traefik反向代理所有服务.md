# traefik.toml 配置

```
logLevel = "INFO"

defaultEntryPoints = ["https"]

[entryPoints]
  [entryPoints.traefik]
  address = ":8080"

  [entryPoints.https]
  address = ":80"
  [entryPoints.https.tls]
    [[entryPoints.https.tls.certificates]]
    certFile = "/certificate/cert.pem"
    keyFile = "/certificate/privkey.pem"

[api]

[file]
  watch = true

[backends]
  [backends.dsm]
    [backends.dsm.servers]
      [backends.dsm.servers.server0]
        url = "http://192.168.50.213:9305"
  [backends.minio]
    [backends.minio.servers]
      [backends.minio.servers.server0]
        url = "http://192.168.50.213:9307"

[frontends]
  [frontends.dsm]
    entryPoints = ["https"]
    backend = "dsm"
    passHostHeader = true
    [frontends.dsm.routes]
      [frontends.dsm.routes.route0]
        rule = "Host:dsm.xxxx.com"
  [frontends.minio]
    entryPoints = ["https"]
    backend = "minio"
    passHostHeader = true
    [frontends.minio.routes]
      [frontends.minio.routes.route0]
        rule = "Host:minio.xxxx.com"
```

# docker 启动

``` bash
docker run --restart=always --name traefik -d -p 9308:8080 -p 93:80 -v /volume1/docker/traefik/traefik.toml:/etc/traefik/traefik.toml -v /usr/syno/etc/certificate/system/default/:/certificate  traefik
```
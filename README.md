<div align="center">

![Aster Explorer](./public/aster.svg)

<h1>Aster Explorer</h1>

**Muse explorer is not only an explorer but also a wallet and more ... 🛠**

[![version](https://img.shields.io/github/tag/ping-pub/explorer.svg)](https://github.com/muses-network/muses-explorer/releases/latest)
[![GitHub](https://img.shields.io/github/license/ping-pub/explorer.svg)](https://github.com/muses-network/muses-explorer/blob/master/LICENSE)
[![Twitter URL](https://img.shields.io/twitter/url/https/twitter.com/bukotsunikki.svg?style=social&label=Follow%20%40ping_pub)](https://twitter.com/musesnetwork)
[![https://discord.gg/rQ4gqrqsDt](https://img.shields.io/badge/discord-join-7289DA.svg?logo=discord&longCache=true&style=flat)](https://discord.gg/rQ4gqrqsDt)


</div>


# Installation:

1. Running with yarn
```
yarn && yarn serve
```

2. Building for web servers, like nginx, apache
```
yarn && yarn build
cp -r ./dist/* <ROOT_OF_WEB_SERVER>
```

# Enable LCD for Aster Explorer

1. Set `enable = true` in `./config/app.toml`
```
###############################################################################
###                           API Configuration                             ###
###############################################################################

[api]

# Enable defines if the API server should be enabled.
enable = true

# Swagger defines if swagger documentation should automatically be registered.
swagger = false

# Address defines the API server to listen on.
address = "tcp://0.0.0.0:1317"

# MaxOpenConnections defines the number of maximum open connections.
max-open-connections = 1000
```

2. add proxy server and enable CORS. NOTE: You must enable https as well.

```
server {
    server_name juno.api.ping.pub;
    listen 443;
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;

        proxy_pass http://<HOST>:1317;

    }
}
```
3. config your blockchain in [./src/chains]()




## Contributors

Developers: @liangping @dingyiming @meta-bowen @JensenLyou


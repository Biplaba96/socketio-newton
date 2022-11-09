# SocketIO + nginx deployment changes

Since websocket connection needs a upgrade in the protocol do nginx provides following default connection configs for extablishing a `wss/ws` protocol while using reverse proxy.

```sh
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection “upgrade”;
```

## troubleshoot

socket.io uses its specific way of connection, to the request it always add `/socket.io` prefix to all of its request including the connection one in the default request path, for correctly redirecting we are writing the regular expression `rewrite ^/socket/(.*) /$1 break;` for proxy rewrting the path else `/socket.io/*` will always become a unknown route for node backend

### correct configurations

```sh
 location / {
    proxy_pass http://127.0.0.1:4000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection “upgrade”;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
}

location ^~ /socket {
           rewrite  ^/socket/(.*)  /$1 break;
           proxy_pass http://127.0.0.1:4000;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host $host;
}
```

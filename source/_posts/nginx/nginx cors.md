---
title: nginx cors
date: 2019-11-02 08:37:00
tags: [nginx]
---

参考链接 : [https://medium.com/@hariomvashisth/cors-on-nginx-be38dd0e19df](https://medium.com/@hariomvashisth/cors-on-nginx-be38dd0e19df)

```
server {
  listen        80;
  server_name   api.test.com;


  location / {

    # Simple requests
    if ($request_method ~* "(GET|POST)") {
      add_header "Access-Control-Allow-Origin"  *;
    }

    # Preflighted requests
    if ($request_method = OPTIONS ) {
      add_header "Access-Control-Allow-Origin"  *;
      add_header "Access-Control-Allow-Methods" "GET, POST, OPTIONS, HEAD";
      add_header "Access-Control-Allow-Headers" "Authorization, Origin, X-Requested-With, Content-Type, Accept";
      return 200;
    }

    ....
    # Handle request
    ....
  }
}
```

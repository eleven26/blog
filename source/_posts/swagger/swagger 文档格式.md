---
title: swagger 文档格式(json)
date: 2019-07-04 20:20:00
tags: [swagger]
---

[官网文档](https://swagger.io/docs/specification/basic-structure/)

##### response example raw content

```
"responses": {
  "405": {
    "description": "Invalid input"
  },
  "200": {
    "description": "OK",
    "schema": {
      "type": "string",
      "example": "{\"a\": 1}\n a test"
    }
  }
},
```


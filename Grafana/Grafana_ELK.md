**If you use TLS certificate in ELK you need next settings**

```
Name              ELK-nginx
```

**HTTP**
```
URL               https://elk.local:9200
Access            Server (default)
```

**Auth**
```
Basic auth
With Credentials
Skip TLS Verify
```

**Basic Auth Details**
```
User              elksearch
Password
```

**Elasticsearch details**
```
Index name        [nginx-]YYYY.MM.DD
Pattern           Daily
Time field name   @timestamp
Version           7.0+
Max concurrent Shard Requests   5
Min time interval 30s
```

![image](https://user-images.githubusercontent.com/62062799/131359281-fe31952e-9c7d-45f2-9c7d-58b7abeaf506.png)

# nginx 常用

## 文件服务器

```
autoindex on;# 显示目录
autoindex_exact_size on;# 显示文件大小
autoindex_localtime on;# 显示文件时间

server {
    charset      utf-8; # 
    listen       8888 ;
    root         /opt/share;
}
```



## https

```
生成自签名证书
openssl  genrsa -out ca.key 2048
openssl  req -new  -x509 -key  ca.key  -out server.crt  -days 3650
```



```
server {
    listen       9999 ssl;

    #ssl证书的文件路径
    ssl_certificate  /opt/https/server.crt;
    #ssl证书的key文件路径
    ssl_certificate_key /opt/https/ca.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    location / {
        root   /opt/https/www;
        index  index.html index.htm;
    }

}

```

## 反向代理

```

server {
    listen       9999 ;
    location / {
       proxy_pass http://realserver:8080;
    }

}
```

## 负载均衡

```
http {
    upstream api{
        server server1:8080;
        server server2:8080;
        server server3:8080;
    }
    server {
        listen       80;
        location / {
            proxy_pass  http://api;
             #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $remote_addr;
        }
    }
}

```

```
http {
    upstream api{
        server server1:8080 weight=1;
        server server2:8080 weight=2;
        # 热备,无法访问才会访问此地址
        server server3:8080 weight=1 backup;
    }
    server {
        listen       80;
        location / {
            proxy_pass  http://api;
        }
    }
}
```

```
http {
    upstream api{
        #同个ip访问同个服务
        ip_hash;
        server server1:8080;
        server server2:8080;
        server server3:8080;
    }
    server {
        listen       80;
        location / {
            proxy_pass  http://api;
        }
    }
}
```





## 路径匹配

```
location [ = | ~ | ~* | ^~ ] uri { ... }

=:  精准匹配全路径, 命中它后直接返回, 不再进行后续匹配, 优先级最高.

^~: 精准匹配开头, 命中开头后直接返回, 不再进行后续匹配, 优先级第二.

~:  区分大小写的正则匹配, 命中后则不进行后续匹配, 立即返回, 优先级第三.

*~: 不区分大小写的正则匹配, 命中后则不进行后续匹配, 立即返回, 优先级第三.

无匹配方式符号: 通用性匹配, 命中后还会继续后续匹配, 最后选取路径最长的匹配, 并储存起来, 优先级第四.
```



## rewrite

```
```


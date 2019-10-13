title: NGINX 配置端口转发
author: Answer
tags:
  - nginx
categories:
  - nginx
date: 2019-10-13 14:15:00
---

# 多个 Springboot 项目运行在**不同端口**，通过**不同路径**转发到不同端口

http://127.0.0.1:8081/index 访问 webA

http://127.0.0.1:8082/index 访问 webB

http://127.0.0.1:8083/index 访问 webC


修改后：

http://127.0.0.1/a/index 访问 webA

http://127.0.0.1/b/index 访问 webB

http://127.0.0.1/c/index 访问 webC

```
http {
    ... 
    upstream weba {
		server 127.0.0.1:8081;
	}

	upstream webb {
		server 127.0.0.1:8082;
	}

	upstream webc {
		server 127.0.0.1:8083;
	}
    ...
    
    server {
        ...
        location /a/ {
		    proxy_pass http://weba/;
		    # proxy_pass http://127.0.0.1:8081/;   # 也可以直接写ip+端口
		}
		
		location /b/ {
		    proxy_pass http://webb/; 
	    }
		
	    location /c/ {
		    proxy_pass http://webc/; 
	    }
	    ...
	}
}
```

# 访问某个路径直接跳转到新页面

输入 http://127.0.0.1/blog 就会跳转到百度首页
```
location ^~ /blog {
                rewrite ^  https://www.baidu.com;
            }
```


# location 与 proxy_pass 斜杠问题

[nginx proxy_pass末尾神奇的斜线](https://blog.51cto.com/chenwenming/1203537)

```
server.port=10240

@RequestMapping("/a")
public String helloA(@RequestParam String name){
    return "Hello "+ name +" ! now time is "+new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(System.currentTimeMillis());
}
```

服务运行在 10240 端口，使用 http://127.0.0.1:10240/a?name=123 即可输出结果

- proxy_pass 后面加上斜杠，访问 http://127.0.0.1/test1/a?name=123 其实访问 http://127.0.0.1:10240/a?name=123
```
location ^~/test1/ {
			proxy_pass http://127.0.0.1:10240/;
}
```

- proxy_pass 后面没有斜杠，访问 http://127.0.0.1/test1/a?name=123 它其实访问的是  http://127.0.0.1:10240/test1/a?name=123
```
location ^~/test1/ {
			proxy_pass http://127.0.0.1:10240;
}
```



# 遇到的问题

- unknown log format "main" in C:\nginx-1.17.3/conf/nginx.conf:28
打开nginx.conf，"main"错误是因为丢失了log_format选项，之前把他屏蔽掉了，修改之后问题解决。


- listen 80 端口不生效
include /etc/nginx/sites-enabled/*; 注释掉，这个

	[nginx 配置根目录不生效问题](https://blog.csdn.net/yin138/article/details/79267169)


- [权限问题导致Nginx 403 Forbidden错误的解决方法](https://www.jb51.net/article/54190.htm)

	于是在nginx.conf头部加入一行：user  root;


# 参考

[nginx 同一个域名下部署多个工程](https://blog.csdn.net/chenhuaping007/article/details/79997901)
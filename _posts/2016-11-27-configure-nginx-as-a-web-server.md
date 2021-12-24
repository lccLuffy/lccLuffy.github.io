---
layout: post
permalink: /configure-nginx-as-a-web-server/
title: 你真的了解如何将 Nginx 配置为Web服务器吗
date: 2016-11-27T23:14:58
lang: zh-CN
---

阅读之前，建议先阅读[初识 Nginx](https://lufficc.com/blog/nginx-for-beginners)。 之后，我们来了解一下 Nginx 配置。


抽象来说，将 Nginx 配置为 Web 服务器就是定义**处理哪些 `URLS` ** 和**如何处理这些`URLS` 对应的请求**。具体来说，就是定义一些虚拟服务器（Virtual Servers），控制具有特定 IP 和域名的请求。


更具体的来说， Nginx 通过定义一系列 `location`s 来控制对 `URIS` 的选择。每一个 `location` 定义了对映射到自己的请求的处理场景：**返回一个文件**或者**代理请求**，或者根据不同的错误代码返回不同的错误页面。另外，根据 `URI` 的不同，请求也可以被**重定向**到其它 `server` 或者 `location` 。

## 设置虚拟服务器

#### `listen`：
Nginx 配置文件至少包含一个 `server` 命令 ，用来定义虚拟服务器。当请求到来时， Nginx  会首先选择一个虚拟服务器来处理该请求。
 
虚拟服务器定义在 `http` 上下文中的 `server` 中：
```
http {
    server {
        # Server configuration
    }
}
```
> *注意： `http` 中可以定义多个 `server`*

 `server` 配置块使用 `listen` 命令监听本机 IP 和端口号（包括 Unix domain socket and path），支持 IPv4、IPv6，IPv6地址需要用方括号括起来：
```
server {
    listen 127.0.0.1:8080;  # IPv4地址，8080端口
	# listen [2001:3CA1:10F:1A:121B:0:0:10]:80;   # IPv6地址，80端口
	# listen [::]:80;  # 听本机的所有IPv4与IPv6地址，80端口
    # The rest of server configuration
}
```
上述配置，如果不写端口号，默认使用80端口，如果不写 IP ，则监听本机所有 IP。

#### `server_name`：
如果多个 `server` 的 `listen` IP 和端口号一模一样， Nginx 通过请求头中的 Host <img src="https://static.lufficc.com/image/474c32008006ad74c1d7c1dcb8f152ed.png" style="display:inline-block">与 `server_name` 定义的主机名进行比较，来选择合适的虚拟服务器处理请求：

```
server {
    listen      80;
    server_name lufficc.com  www.lufficc.com;
    ...
}
```
`server_name` 的参数可以为：
1. 完整的主机名，如：`api.lufficc.com` 。
2. 含有通配符（含有 `*`），如：`*.lufficc.com` 或 `api.*` 。
3. 正则表达式，以 `~` 开头。

> 通配符只能在开头或结尾，而且只能与一个 `.` 相邻。`www.*.example.org` 和 `w*.example.org` 均无效。 但是，可以使用正则表达式匹配这些名称，例如 `~^www\..+\.example\.org$` 和 `~^w.*\.example\.org$` 。 而且 `*` 可以匹配多个部分。 名称 `* .example.org` 不仅匹配 `www.example.org`，还匹配`www.sub.example.org`。
> 对于正则表达式：Nginx 使用的正则表达式与 Perl 编程语言（PCRE）使用的正则表达式兼容。 要使用正则表达式，且必须以 `~` 开头。

命名的正则表达式可以捕获变量，然后使用：
```
server {
    server_name   ~^(www\.)?(?<domain>.+)$;

    location / {
        root   /sites/$domain;
    }
}
```
小括号 `()` 之间匹配的内容，也可以在后面通过 `$1` 来引用，`$2` 表示的是前面第二个 `()` 里的内容。因此上述内容也可写为：
```
server {
    server_name   ~^(www\.)?(.+)$;

    location / {
        root   /sites/$2;
    }
}
```


一个 `server_name` 示例：

```
server {
    listen      80;
    server_name api.lufficc.com  *.lufficc.com;
    ...
}
```

同样，如果多个名称匹配 `Host` 头部， Nginx 采用下列顺序选择：

1. 完整的主机名，如 `api.lufficc.com`。
2. 最长的，且以 `*` 开头的通配名，如：`*.lufficc.com`。
3. 最长的，且以 `*` 结尾的通配名，如：`api.*` 。
4. 第一个匹配的正则表达式。（按照配置文件中的顺序）


即优先级：`api.lufficc.com` > `*.lufficc.com` > `api.*` > 正则。

如果 `Host` 头部不匹配任何一个 `server_name` ,Nginx 将请求路由到默认虚拟服务器。默认虚拟服务器是指：`nginx.conf ` 文件中第一个 `server` 或者 显式用 `default_server` 声明：
```
server {
    listen      80 default_server;
    ...
}
```

## 配置 `location`

#### `URI` 与 `location` 参数的匹配

当选择好 `server` 之后，Nginx 会根据 `URI`s 选择合适的 `location` 来决定代理请求或者返回文件。

`location` 指令接受两种类型的参数：
1. 前缀字符串（路径名称）
2. 正则表达式

对于前缀字符串参数， `URI`s 必须严格的以它开头。例如对于 `/some/path/` 参数，可以匹配 `/some/path/document.html` ，但是不匹配 `/my-site/some/path`，因为  `/my-site/some/path` 不以 `/some/path/` 开头。

```
location /some/path/ {
    ...
}
```
对于正则表达式，以 `~` 开头表示大小写敏感，以 `~*` 开头表示大小写不敏感。注意路径中的 `.` 要写成 `\.` 。例如一个匹配以 ` .html` 或者 `.htm` 结尾的 `URI` 的 `location`：
```
location ~ \.html? {
    ...
}
```

正则表达式的优先级大于前缀字符串。如果找到匹配的前缀字符串，仍继续搜索正则表达式，但如果前缀字符串以 `^~` 开头，则不再检查正则表达式。

具体的搜索匹配流程如下：
1. 将 `URI` 与所有的前缀字符串进行比较。
2. `=` 修饰符表明 `URI` 必须与前缀字符串相等（不是开始，而是相等），如果找到，则搜索停止。
3. 如果找到的最长前缀匹配字符串以 `^~` 开头，则不再搜索正则表达式是否匹配。
4. 存储匹配的最长前缀字符串。
5. 测试对比 `URI` 与正则表达式。
6. 找到第一个匹配的正则表达式后停止。
7. 如果没有正则表达式匹配，使用 4 存储的前缀字符串对应的 `location`。


`=` 修饰符拥有最高的优先级。如网站首页访问频繁，我们可以专门定义一个 `location` 来减少搜索匹配次数（因为搜索到 `=` 修饰的匹配的 `location` 将停止搜索），提高速度：
```
location = / {
    ...
}
```

#### 静态文件和代理

`location` 也定义了如何处理匹配的请求：**返回静态文件** 或者 **交给代理服务器处理**。下面的例子中，第一个 `location` 返回 `/data` 目录中的静态文件，第二个 `location` 则将请求传递给 https://lufficc.com 域名的服务器处理：
```
server {
    location /images/ {
        root /data;
    }

    location / {
        proxy_pass https://lufficc.com;
    }
}
```


`root` 指令定义了静态文件的根目录，并且和 `URI` 拼接形成最终的本地文件路径。如请求 `/images/example.png`，则拼接后返回本地服务器文件 ` /data/images/example.png` 。


`proxy_pass` 指令将请求传递到 URL 指向的代理服务器。让后将来自代理服务器的响应转发给客户端。 在上面的示例中，所有不以 `/images /` 开头的 `URI` 的请求都将传递给代理服务器处理。

比如我把 `proxy_pass` 设置为 `https://www.baidu.com/`，那么访问 http://search.lufficc.com/ 将得到百度首页一样的响应（页面）（感兴趣的童鞋可以自己试一试搜索功能，和百度没差别呢）：
```
server{
      listen 80;
      server_name search.lufficc.com;
      location / {
              proxy_pass https://www.baidu.com;
      }
}
```

## 使用变量（Variables）

你可以使用变量来使 Nginx 在不同的请求下采用不同的处理方式。变量是在运行时计算的，用作指令的参数。 变量由 `$` 开头的符号表示。 变量基于 Nginx 的状态定义信息，例如当前处理的请求的属性。

有很多预定义变量，例如核心的 HTTP 变量，你也可以使用 `set`，`map` 和 `geo` 指令定义自定义变量。 大多数变量在运行时计算，并包含与特定请求相关的信息。 例如，`$remote_addr` 包含客户端 IP 地址，`$uri` 保存当前URI值。

一些常用的变量如下：

| 变量名称 | 作用 |
| -------- | -------- |
| `$uri`     | 请求中的当前URI(不带请求参数)，它可以通过内部重定向，或者使用index指令进行修改，`$uri`不包含主机名,如 `/foo/bar.html`。     |
| `$arg_name`     | 请求中的的参数名，即“?”后面的`arg_name=arg_value`形式的`arg_name`     |
| `$hostname`     | 主机名     |
| `$args`     | 请求中的参数值     |
| `$query_string`     | 同 `$args`     |
| `$request`     | 代表客户端的请求地址     |
| `$request_uri`     | 这个变量等于包含一些客户端请求参数的原始URI，它无法修改，不包含主机名，如：`/cnphp/test.php?arg=freemouse`。     |
| ...     | ...     |

一个简单的应用就是从 `http` 重定向到 `https` 时带上路径信息：
```
server{
       ...
       return      301 https://lufficc.com$request_uri;
       ...
}
```

## 返回特定状态码
如果你的网站上的一些资源永久移除了，最快最简洁的方法就是使用 `return` 指令直接返回：
```
location /wrong/url {
    return 404;
}
```
`return` 的第一个参数是响应代码。可选的第二个参数可以是重定向（对应于代码301，302，303和307）的 URL 或在响应正文中返回的文本。 例如：
```
location /permanently/moved/url {
    return 301 http://www.example.com/moved/here;
}
```

`return` 指令可以包含在 `location` 和 `server` 上下文中：
```
server{
      location / {
              return 404;
      }
}
```

或者：

```
server{
      ...
      return 404;
      location / {
          ...            
      }
}
```

## 错误处理
`error_page` 命令可以配置特定错误码的错误页面，或者重定向到其他的页面。下面的示例将在 404 错误发生时返回 `/404.html` 页面。
```
error_page 404 /404.html;
```

`error_page` 命令定义了如何处理错误，因此不会直接返回，而 `return` 确实会立即返回。当代理服务器或者 Nginx 处理时产生相应的错误的代码，均会返回相应的错误页面。

在下面的示例中，当 Nginx 找不到页面时，它将使用代码301替换代码404，并将客户端重定向到 `http://example.com/new/path.html` 。 此配置很有用，比如当客户端仍尝试用旧的 `URI` 访问页面时，301代码通知浏览器页面已永久移除，并且需要自动替换为返回的新地址。

```
location /old/path.html {
    error_page 404 =301 http:/example.com/new/path.html;
}
```
## 重写 `URI`s
`rewrite` 指令可以多次修改请求的 `URI`。`rewrite` 的第一个参数是 `URI`需要匹配的正则表达式，第二个参数是将要替换的 `URI`。第三个参数可选，指示是否继续可以重写或者返回重定向代码（301或302）。例如：
```
location /users/ {
    rewrite ^/users/(.*)$ /show?user=$1 break;
}
```

您可以在 `server ` 和 `location` 上下文中包括多个 `rewrite` 指令。 `Nginx` 按照它们发生的顺序一个一个地执行指令。 当选择 `server` 时，`server` 中的 `rewrite` 指令将执行一次。

在 `Nginx` 处理一组 `rewrite` 指令之后，它根据新的 `URI` 选择 `location` 。 如果所选 `location` 仍旧包含 `rewrite` 指令，它们将依次执行。 如果 `URI` 匹配所有，则在处理完所有定义的 `rewrite` 指令后，搜索新的 `location` 。

以下示例将 `rewrite` 指令与 `return` 指令结合使用：
```
server {
    ...
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 last;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  last;
    return  403;
    ...
}
```
诸如 `/download/some/media/file` 的 URI 被改为 `/download/some/mp3/file.mp3` 。 由于 `last` 标志，后续指令（第二个 `rewrite` 指令和 `return` 指令）被跳过，但 Nginx 继续以更改后的 URI 处理请求。 类似地，诸如 `/download/some/audio/file` 的 URI 被替换为 `/download/some/mp3/file.ra`。 如果 URI 不匹配 `rewrite` 指令，Nginx 将403 错误代码返回给客户端。

`last` 与 `break`的区别是：

- `last` ： 在当前 `server` 或 `location` 上下文中停止执行 `rewrite` 指令，但是 Nginx 继续搜索与重写的URI匹配的 `location`，并应用新 `location` 中的任何 `rewrite` 指令（这意味着 URI 可能再次改变）。
- `break` ：停止当前上下文中 `rewrite` 指令的处理，并取消搜索与新 URI 匹配的 `location`。 不会执行新 `location`中的 `rewrite` 指令。

## 附录
#### 常用正则
- `.`  ： 匹配除换行符以外的任意字符
- `?` ： 重复0次或1次
- `+` ： 重复1次或更多次
- `* `： 重复0次或更多次
- `\d` ：匹配数字
- `^` ： 匹配字符串的开始
- `$` ： 匹配字符串的结束
- `{n}` ： 重复n次
- `{n,}` ： 重复n次或更多次
- `[c]` ： 匹配单个字符c
- `[a-z] `： 匹配a-z小写字母的任意一个

#### 全局变量

- `$args` ： #这个变量等于请求行中的参数，同`$query_string`
- `$content_length` ： 请求头中的Content-length字段。
- `$content_type` ： 请求头中的Content-Type字段。
- `$document_root` ： 当前请求在root指令中指定的值。
- `$host` ： 请求主机头字段，否则为服务器名称。
- `$http_user_agent` ： 客户端agent信息
- `$http_cookie` ： 客户端cookie信息
- `$limit_rate` ： 这个变量可以限制连接速率。
- `$request_method` ： 客户端请求的动作，通常为GET或POST。
- `$remote_addr` ： 客户端的IP地址。
- `$remote_port` ： 客户端的端口。
- `$remote_user` ： 已经经过Auth Basic Module验证的用户名。
- `$request_filename` ： 当前请求的文件路径，由root或alias指令与URI请求生成。
- `$scheme` ： HTTP方法（如http，https）。
- `$server_protocol` ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
- `$server_addr` ： 服务器地址，在完成一次系统调用后可以确定这个值。
- `$server_name` ： 服务器名称。
- `$server_port` ： 请求到达服务器的端口号。
- `$request_uri` ： 包含请求参数的原始URI，不包含主机名，如：`/foo/bar.php?arg=baz`。
- `$uri` ： 不带请求参数的当前URI，$uri不包含主机名，如`/foo/bar.html`。
- `$document_uri` ： 与$uri相同。

例如请求：http://localhost:88/test1/test2/test.php

|  `$host`    |   `localhost`   |
|      ---      |        ---          |
|   `$server_port`   |   88   |
|   `$request_uri`   |   /test1/test2/test.php   | 
|   `$document_uri`   |   /test1/test2/test.php   |
|   `$document_root`   |  /var/www/html    |
|   `$request_filename`   |   /var/www/html/test1/test2/test.php   |

## 参考
1. https://www.nginx.com/resources/admin-guide/nginx-web-server/
2. http://seanlook.com/2015/05/17/nginx-location-rewrite/

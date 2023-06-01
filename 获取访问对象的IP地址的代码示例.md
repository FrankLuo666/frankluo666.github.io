---
title: "获取访问对象的IP地址的代码示例"
date: 2023-04-18T20:35:35+09:00
draft: false
categories:
- IT
tags:
- java
- python
---



#### 在Java Spring Boot中，获取访问对象的IP地址：

在Java Spring Boot中，可以使用HttpServletRequest对象来获取访问者的IP地址。HttpServletRequest是一个接口，它提供了有关HTTP请求的信息，包括请求URL、请求头、请求参数等等。

下面是一个示例代码，展示如何使用HttpServletRequest对象来获取访问者的IP地址：

```java
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.beans.factory.annotation.Autowired;

import javax.servlet.http.HttpServletRequest;

@RestController
public class MyController {

    @Autowired
    private HttpServletRequest request;

    @RequestMapping(value = "/get-ip", method = RequestMethod.GET)
    public String getIP() {
        String ipAddress = request.getHeader("X-Forwarded-For");
        if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("HTTP_CLIENT_IP");
        }
        if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getRemoteAddr();
        }
        return ipAddress;
    }
}

```

在上面的示例代码中，我们注入了HttpServletRequest对象，并在getIP()方法中使用该对象来获取访问者的IP地址。为了获取IP地址，我们首先尝试从请求头中获取“X-Forwarded-For”、“Proxy-Client-IP”、“WL-Proxy-Client-IP”、“HTTP_CLIENT_IP”和“HTTP_X_FORWARDED_FOR”等标准请求头。如果无法从这些请求头中获取IP地址，我们将使用HttpServletRequest对象的getRemoteAddr()方法来获取IP地址。

需要注意的是，由于HTTP请求可能经过代理服务器，因此访问者的IP地址可能包含在“X-Forwarded-For”等标准请求头中。在实际开发中，我们需要根据自己的实际情况选择合适的方式来获取IP地址。



---

#### 在python的Flask 框架中获取访问对象的的 IP 地址：

在 Python 项目中，可以使用 Flask 框架的 request 对象来获取访问者的 IP 地址。Flask 是一款轻量级的 Web 应用框架，它提供了方便的 request 对象，用于处理 HTTP 请求的各种参数和信息。

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/")
def index():
    ip_address = request.remote_addr
    return "Your IP address is: " + ip_address

if __name__ == "__main__":
    app.run()

```

在上面的示例代码中，我们定义了一个路由，当访问根路径时，会返回访问者的 IP 地址。我们使用 Flask 的 request 对象的 remote_addr 属性来获取访问者的 IP 地址。

需要注意的是，由于 HTTP 请求可能经过代理服务器，因此访问者的 IP 地址可能会被代理服务器所覆盖。如果要获取真实的访问者 IP 地址，可以考虑使用一些 HTTP 请求头信息，例如 X-Forwarded-For 头等。如果你使用的是 Flask 框架，你可以使用 request.headers.get("X-Forwarded-For") 来获取该请求头信息。



---

#### 用python代码获取本机IP 地址：

Python 可以使用标准库中的 socket 模块来获取访问者的 IP 地址。socket 模块是 Python 中用于网络编程的标准库，它提供了一些常见的网络编程操作，例如创建和操作套接字（socket）、发送和接收数据等。

```python
import socket

def get_ip_address():
    hostname = socket.gethostname()
    ip_address = socket.gethostbyname(hostname)
    return ip_address

if __name__ == "__main__":
    print("Your IP address is: " + get_ip_address())
```

在上面的示例代码中，我们使用 socket 模块的 gethostname() 方法来获取本机主机名，然后使用 gethostbyname() 方法来获取本机 IP 地址。这种方法只能获取本机的 IP 地址，如果想要获取访问者的 IP 地址，需要通过其他方式，例如解析 HTTP 请求头信息。

需要注意的是，在获取访问者的 IP 地址时，同样需要考虑代理服务器的影响。在实际开发中，建议使用 Flask 或其他成熟的 Web 框架来处理网络请求，这样可以方便地获取访问者的真实 IP 地址。




## 1. 什么是协议

协议是指双方或多方，相互约定好，通信方都需要遵守的规则，叫协议。

## 2. HTTP协议

所谓HTTP协议(HyperText Transfer Protocol，超文本传输协议)就是客户端和服务器通信时需要遵守的规则。

## 3. 请求的HTTP协议格式

**请求**：客户端给服务器发送数据，分为GET请求和POST请求两种

**响应**：服务器给客户端回传数据

### 3.1 GET请求

```http
GET /07_servlet/servlet1 HTTP/1.1
Host: localhost:8088
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36 Edg/88.0.705.68
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
```

1. 请求行

   （1）请求的方式                                         `GET     `

   （2）请求的资源路径[+?+请求参数]        ` /07_servlet/servlet1`

   （3）请求的协议版本号                             `HTTP/1.1`

2. 请求头

   key:value  组成         不同的键值对，表示不同的含义

   | key             | 含义                                                         |
   | --------------- | ------------------------------------------------------------ |
   | Host            | 表示请求的服务器ip和端口号                                   |
   | Connection      | 告诉服务器请求连接如何处理<br>keep-alive：告诉服务器回传数据不要马上关闭，保持一小段时间的连接；<br>closed：马上关闭 |
   | User-Agent      | 浏览器信息                                                   |
   | Accept          | 告诉服务器，客户端可以接收的数据类型                         |
   | Accept-Encoding | 告诉服务器，客户端可以接收的数据编码（压缩）格式             |
   | Accept-Language | 告诉服务器，客户端可以接收的语言类型                         |

### 3.2 POST请求

```http
POST /07_servlet/parameterServlet HTTP/1.1
Host: localhost:8088
Connection: keep-alive
Content-Length: 35
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36 Edg/88.0.705.68
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://localhost:8088/07_servlet/form.html
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6

username=y11&password=123&hobby=cpp
```

1. 请求行

   （1）请求的方式                                         `POST     `

   （2）请求的资源路径[+?+请求参数]        ` /07_servlet/parameterServlet`

   （3）请求的协议版本号                             `HTTP/1.1`

2. 请求头

   key:value  组成         不同的键值对，表示不同的含义

   | key            | 含义                                                         |
   | -------------- | ------------------------------------------------------------ |
   | Content-Length | 表示发送的数据长度                                           |
   | Cache-Control  | 表示如何控制缓存                                             |
   | Referer        | 表示请求发起时，浏览器地址栏中的地址（从哪来）               |
   | Content-Type   | 表示发送的数据类型<br>application/x-www-form-urlencoded：<br>表示提交的数据格式是：name=value&name=value，然后对其进行url编码，<br>url编码是把非英文内容转换为：%xx%xx）<br>multipart/form-data<br>表示以多段的形式提交数据给服务器（以流的形式提交，用于上传） |

**空行**

3. 请求体

   就是发送给服务器的数据：username=y11&password=123&hobby=cpp

## 4. 响应的HTTP协议格式

### 4.1 响应格式

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Date: Thu, 18 Feb 2021 11:04:21 GMT
Accept-Ranges: bytes
ETag: W/"611-1613640379836"
Last-Modified: Thu, 18 Feb 2021 09:26:19 GMT
Content-Type: text/html
Content-Length: 611

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="http://localhost:8088/07_servlet/parameterServlet" method="post">
        用户名：<input type="text" name="username"><br/>
        密码：<input type="password" name="password"><br/>
        兴趣爱好：<input type="checkbox" name="hobby" value="cpp">C++
                <input type="checkbox" name="hobby" value="Java">Java
                <input type="checkbox" name="hobby" value="js">JavaScript<br/>
        <input type="submit">
    </form>
</body>
</html>
```

1. 响应行

   （1）响应的协议和版本号       `HTTP/1.1`

   （2）响应状态码                      `200`

   （3）响应状态描述符              `OK`

2. 响应头

   key : value          不同的响应头，表示不同的含义

   | key            | 含义                       |
   | -------------- | -------------------------- |
   | Server         | 表示服务器的信息           |
   | Content-Type   | 表示响应体的数据类型       |
   | Content-Length | 响应体的长度               |
   | Date           | 请求响应的时间（格林时间） |

**空行**

3. 响应体：回传给客户端的数据

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
       <form action="http://localhost:8088/07_servlet/parameterServlet" method="post">
           用户名：<input type="text" name="username"><br/>
           密码：<input type="password" name="password"><br/>
           兴趣爱好：<input type="checkbox" name="hobby" value="cpp">C++
                   <input type="checkbox" name="hobby" value="Java">Java
                   <input type="checkbox" name="hobby" value="js">JavaScript<br/>
           <input type="submit">
       </form>
   </body>
   </html>
   ```

### 4.2 常用的响应码说明

| 响应码 | 含义                                                       |
| ------ | ---------------------------------------------------------- |
| 200    | 请求成功                                                   |
| 302    | 请求重定向                                                 |
| 404    | 请求服务器已经收到了，但是请求的数据不存在（请求地址错误） |
| 500    | 服务器已经收到请求，但是服务器内部错误（代码错误）         |

## 5. MIME类型说明

MIME(Multipurpose Internet Mail Extensions, 多功能Internet邮件扩充服务)是HHTP协议中的数据类型。<br/>

MIME类型的格式是：大类型/小类型，并与某一种文件的扩展名相对应。<br/>

| 文件              | MIME类型   | 格式                     |
| ----------------- | ---------- | ------------------------ |
| 超文本标记语言    | .html,.htm | text/html                |
| 普通文本          | .txt       | text/plain               |
| RTF文本           | .rtf       | application/rtf          |
| GIF图形           | .gif       | image/gif                |
| JPEG图形          | .jpeg,.jpg | image/jpeg               |
| au声音文件        | .au        | audio/basic              |
| MIDI音乐文件      | .mid,.midi | audio/midi, audio/x-midi |
| RealAudio音乐文件 | .ra,.ram   | audio/x-pn-realaudio     |
| MPEG文件          | .mpg,.mpeg | video/mpeg               |
| AVI文件           | .avi       | video/x-msvideo          |
| GZIP文件          | .gz        | application/x-gzip       |
| TAR文件           | .tar       | application/x-tar        |
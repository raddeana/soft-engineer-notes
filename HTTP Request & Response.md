### Request
```
POST / HTTP/1.1
Host: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
field1=aaa&code=x%3D1
```

- HTTP 协议是一个文本型的协议，文本型的协议一般来说与二进制的协议是相对的，也意味着这个协议里面所有内容都是字符串，每一个字节都是字符串的一部分。
- HTTP 协议的第一行叫做 request line，包含了三个部分：
  - Method：例如 POST，GET 等
  - Path：默认就是 “/”
  - HTTP和HTTP版本：HTTP/1.1


- 然后接下来就是 Headers
  - Header的行数不固定
  - 每一行都是以一个冒号分割了 key: value 格式
  - Headers是以空行进行结束

- 最后的一部分就是 body 部分：
  - 这个部分的内容是以 Content-Type来决定的
  - Content-Type 规定了什么格式，那么 body 就用什么格式来写

### response
```
HTTP/1.1 200 OK
Content-Type: text/html
Date: Mon, 23 Dec 2019 06:46:19 GMT
Connection: keep-alive
Transfer-Encoding: chunked
26
Hello World 
0
```
- 首先第一行的 status line 与 request line 相反
  - 第一部分是 HTTP协议的版本：HTTP/1.1
  - 第二部分是 HTTP 状态码：200 (在实现我们的浏览器，为了更加简单一点，我们可以把200以外的状态为出错)
  - 第三部分是 HTTP 状态文本：OK

- 随后的部分就是 header 部分
  - HTML的 request 和 response 都是包含 header 的
  - 它的格式跟 request 是完全一致的
  - 最后是一个空行，用来分割 headers 和 body 内容的部分的

- 最后这里的就是 body 部分了
  - 这里 body 的格式也是根据 Content-Type 来决定的
  - 这里有一种比较典型的格式叫做 chunked body (是 Node 默认返回的一种格式)
  - Chunked body 是由一个十六进制的数字单独占一行
  - 后面跟着内容部分，最后跟着一个十六进制的0，0之后就是整个 body 的结尾了，这个也是用来分割 body 的内容

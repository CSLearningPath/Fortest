# 注册请求处理器

在服务器接收请求并解析后，需要派发给不同的请求处理器处理。例如，HTTP 中，我们会解析 HTTP 方法以及请求的路径，然后匹配对应的处理器函数。而 RPC 中，我们会解析命令号，然后跳转到对应的处理函数。
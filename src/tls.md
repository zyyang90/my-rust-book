# TLS/HTTPS
## 认证、授权的基本概念
- 认证（identification）：确定身份。认证方式有：用户名+密码、手机二维码、邮箱、人脸识别等。
- 授权（authorization）：颁发一个授权媒介，不可被篡改、不可伪造、受保护。
- 鉴权（authentication）：和授权是一一对应的，通过授权媒介，验证其有效性、合法性。
- 权限管理（permission control）：管理权限列表，标注哪些资源在什么样的情况下可以被访问，哪些被禁止。
  - RBAC（role-based access control）：基于角色的权限控制
  - ABAC（attribute-based access control）：基于属性的权限控制
参考：
https://www.cnblogs.com/knqiufan/p/16670178.html
https://docs.authing.cn/v2/guides/access-control/choose-the-right-access-control-model.html

## TLS 和 SSL 有什么区别？
TLS 由 SSL 演变而来。最早，Netscape 开发了 SSL（Secure Socket Layer）。之后，SSL 3.1 在发布时，改成了 TLS 1.0。这两个术语有时会混用。

## TLS 和 HTTPS 有什么区别？
HTTPS = HTTP + TLS。

## TLS 的功能
- 加密：隐藏数据
- 认证：确保信息交换的各方的身份。
- 完整性：验证数据没有被篡改或伪造

## Keep Alive
可以设置 HttpServer 的 keep-alive 行为。
Http 为短链接，为了复用连接，HttpServer 在回复一个报文后，不会马上关闭连接。
TCP keep-alive 和 HTTP keep-alive 的区别
- HTTP keep-alive：目的在于连接复用，在一个连接上尽量完成多的响应。
- TCP keep-alive：心跳，检测连接是否正常。
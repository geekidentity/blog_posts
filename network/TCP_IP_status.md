---
categories: network

tags: 
  - 

title: TCP/IP状态图

date: 2017-01-23
---
在TCP/IP状态图中，有很多种的状态，它们之间有的是可以互相转换的，可以从一种状态转换到另一种状态，但是这种转是要满足一定的条件的。下图即为TCP/IP状态。

![image](http://blog.geekidentity.com/images/tcp_ip_state_transition_diagram.jpg)

由图可以看出，一共有11种不同的状态：

1. CLOSED：关闭状态，没有连接活动；
2. LISTEN：监听状态，服务端正在等待客户端TCP连接请求；
3. SYN_SENT：客户端已经向服务端发送了SYN报文，进入SYN_SENT状态等待服务端确认；
4. SYN_RCVD：服务端收到客户端发送SYN报文，进入SYN_RCVD状态，发送ACK+SYN给客户端；
5. ESTABLISHED：连接建立成功，服务端发送完ACK+SYN后进入该状态，客户端收到ACK后也进入该状态。
6. FIN_WAIT_1：主动关闭连接。无论哪方调用close函数发送FIN报文都会进入这个这个状态。
7. FIN_WAIT_2：被动关闭方同意关闭连接。主动关闭连接方收到被动关闭方返回的ACK后，会进入该状态。
8. TIME_WAIT：收到对方的FIN报文并发送了ACK报文，就等2MSL后即可回到CLOSED状态了。如果FIN_WAIT_1状态下，收到对方同时带FIN标志和ACK标志的报文时，可以直接进入TIME_WAIT状态，而无须经过FIN_WAIT_2状态。
9. CLOSING：双方同时关闭连接。如果双方几乎同时调用close函数，那么会出现双方同时发送FIN报文的情况，就会出现CLOSING状态，表示双方都在关闭连接。
10. CLOSE_WAIT：被动关闭方等待关闭。当收到对方调用close函数发送的FIN报文时，回应对方ACK报文，此时进入CLOSE_WAIT状态。
11. LAST_ACK：被动关闭方发送FIN报文后，等待对方的ACK报文状态，当收到ACK后进入CLOSED状态。
[TOC]
# TCP
![](https://gitee.com/zacharytse/image/raw/master/img/20201219192832.png)
TCP三次握手和四次挥手的本质是对对方发过来的报文的确认。
## 三次握手
第一次是client向server发出连接请求，第二次是server确认client的连接请求，同时发出自己的连接请求，第三次是client确认server的连接请求，这样才能构成全双工的TCP连接
## 四次挥手
第一次是client发出FIN报文，请求断开连接。第二次是server对client的FIN报文的确认(FIN)。第三次之前，server会继续向client发送未发送完的数据，第三次是server向client发出断开连接的请求(FIN),同时server会进入LAST_ACK的状态。第四次当server收到来自client的FIN确认报文之后，server会进入CLOSED状态。同时client会进入TIME_WAIT的状态。
## TIME_WAIT的作用
- 最后一次挥手是client向server发送FIN的确认报文，如果server没有收到ACK报文，需要client再次发送ACK，如果client此时是CLOSED状态，就无法正常向server发送ACK
- 经过2msl的时间，保证此次连接产生的所有报文段都会从网络消失，避免无效的syn报文重复和server连接
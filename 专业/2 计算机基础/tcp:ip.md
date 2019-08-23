# 本篇说明
本篇介绍tcp/ip协议基础知识，tcp篇和ip篇将对tcp和ip两个协议作详细介绍。<br>
<span style="color:red">红字</span>为重要说明，<span style="color:orange">橙字</span>为不确定说明，<span style="color:green">绿字</span>待定。<br>

# 计算机网络体系
![](./tcp:ip/osi模型.webp)
![](./tcp:ip/osi与tcp模型.webp)

osi与tcp模型在分层上稍有区别，osi更注重每一层的<span style="color:orange">功能</span>，tcp更注重<span style="color:orange">实用性</span>。

# tcp/ip协议
tcp/ip协议是指<span style="color:red">tcp/ip协议族</span>，并非字面意思的tcp与ip协议，这一协议族上有很多协议，参考上图。

## 数据包
网络中传输的<span style="color:red">数据包</span>由两个部分构成，一部分是<span style="color:red">协议首部</span>，一部分是上层传递过来的<span style="color:red">数据</span>，首部由具体的协议规范定义。

网络模型中，每个分层都会对所发数据附加一个<span style="color:red">首部</span>，这个首部包含了此层所必要的通信信息(如：端口、目标地址、协议等)。从下层的角度看，上层传递过来的<span style="color:red">数据包</span>都被认为是<span style="color:red">本层的数据</span>，如下图所示：
![](./tcp:ip/数据封包过程.webp)

<span style="color:red">数据包、桢、包、段、消息</span>等5个数据单位大致区分如下：<br>
- <span style="color:orange">数据包</span> **:** 全能性术语，泛指网络传输中的数据单位<br>
- <span style="color:red">frame(桢)</span> **:** 链路层数据单位<br>
- <span style="color:red">packet(包)</span> **:** 网络层数据单位<br>
- <span style="color:red">segment(段)</span> **:** 传输层数据单位<br>
- <span style="color:red">message(消息)</span> **:** 应用层数据单位<br>

## 数据流转过程
下图为用户 a 向用户 b 发送邮件的数据流转过程：
![](./tcp:ip/数据流转过程.webp)

# 补充知识点
## 单工、半双工、全双工
![](./tcp:ip/全双工.gif)

<span style="color:red">单工</span> **:** 支持数据在固定方向上传输，例：电视、广播<br>
<span style="color:red">半双工</span> **:** 支持数据在两个方向上传输，但不能同时传输，例：对讲机<br>
<span style="color:red">全双工</span> **:** 支持数据在两个方向上同时传输，例：电话<br>

## localhost、127.0.0.1、本机地址
<span style="color:red">localhost</span> **:** 指域名，这个域名可以指向127.0.0.1，也可以指向任何一个地址，通常指向127.0.0.1<br>
<span style="color:red">127.0.0.1</span> **:** 127.0.0.1这个地址通常分配给loopback接口<br>
<span style="color:red">本机地址</span> **:** 指绑定在物理或虚拟网络接口上的ip地址，可供其他设备访问

## loopback接口
虚拟网络接口，可用来测试tcp/ip协议栈

# 参考文献
0. [一篇文章带你熟悉TCP/IP协议](https://www.jianshu.com/p/9f3e879a4c9c)
0. [单工，半双工和全双工有何区别和联系？](https://zhidao.baidu.com/question/58243700.html)
0. [localhost、127.0.0.1 和 本机IP 三者的区别？](https://www.zhihu.com/question/23940717)

实验概述

本次实验中，我们会实现一个名为RTP的可靠传输协议。不同于lab1使用TCP来实现传输功能，我们这次将会使用UDP作为基本的传输API，所以在这次实验中，我们还需要补全分片、滑动窗口、超时重传等功能，才能在UDP的简陋支持下实现一个可靠传输协议。

UDP——最简单的传输层协议

！Linux Man

UDP的使用准备和TCP差不多。如果你并不清楚该如何使用UDP相关的函数来完成传输，请在命令行分别输入如下命令：

```bash
man sendto
man recvfrom
```

来查看如何使用这些API。不只是C/C++，部分其他语言和linux shell的命令也支持通过man xxx的方式来查阅如何使用。在以后的工作和学习中，你也可以用这种方式来学习某个命令/API的使用。

？为什么要查阅手册，这里明明只需要几行，不能直接给我代码吗？为什么不能上网查呢？

未来在工作和科研中，大家会接触到许多他人已经写好的代码，其中往往会使用一些同学们可能没有见到过的库函数（例如：strtok，或者其他需要另外安装的库），又或者在初次实现某些功能时，会在博客和开源项目中看到一些没见过的函数，但你又不懂如何使用它。这种情况下，比起等待大神的回复，一行man xxx显然更快。所以，学会自己阅读手册是非常重要的。上网查询的好处在于中文博客没有阅读障碍，但是那毕竟不是原版的手册，而手册是会随着库的版本更新的。如果版本不能匹配，或者因为低质量博客引入了错误的代码，那就得不偿失了。

RTP协议头描述

在开始写代码之前，我们还需要介绍一下RTP的协议头结构：

``` cpp
typedef struct RTP_Header {
    uint8_t type;
    uint16_t length;
    uint32_t seq_num;
    uint32_t checksum;
} rtp_header_t;
```

为了简化操作，RTP头里的所有字段的字节序均是小端法

**type**: 标识了一个RTP报文的类型，0:`START`, 1:`END`, 2:`DATA`, 3:`ACK`

**length**: 标识了一个RTP报文**数据段**的长度（即RTP头后的报文长度），对于`START`,`END`,`ACK`类型的报文来说，长度为0

**seq_num**: 序列号，用于识别顺序做按序送达

**checksum**: 为RTP头以及RTP报文数据段基于32-bit CRC计算出的值，注意，计算checksum之前，checksum字段应被初始化为0

综上所述，RTP头共计有xxx个字节，由于实际的物理网络允许的最大包尺寸是xxx字节，我们在发送UDP报文时，需要对传输的数据进行分片

（对分片逻辑的描述）

RTP协议的简要工作流程

**建立连接** `sender`首先发送一个`type`为`START`，并且`seq_num`为随机值的报文，此后`receiver`应该将带有相同`seq_num`的`ACK`报文发回，`sender`收到`ACK`报文并确认`seq_num`正确后，即认为连接已经建立

**数据传输** 在完成连接的建立之后，要发送的数据由`DATA`类型的报文进行传输。发送方的数据报文的`seq_num`从0开始每个报文递增。注意，数据传输阶段使用的`seq_num`与建立连接时使用的`seq_num`没有联系

**终止连接** 在数据传输完毕后，`sender`发送一个`type`为`END`的报文以终止连接。为了保证所有数据都传输完成，该`END`报文的`seq_num`应当和`receiver`期望收到的下一个报文的`seq_num`相同，在接收到由`receiver`发回的带有相同`seq_num`的`ACK`报文后即认为连接断开

！KISS法则：先完成，再完美

在往年的lab中，总有同学希望一口气写完全部的代码，然后再慢慢debug。但是在面对一些大型项目时，这种做法很容易一口气引入大量的bug，让测试过程变得苦不堪言。KISS的意思是：Keep It Simple and Stupid。在实现复杂逻辑的时候，先从最基本的功能开始搭建，然后立马测试，保证这部分代码正确后，再去实现下一批功能。lab2的代码量或许不大，但它要实现的逻辑比较复杂，按照KISS法则步步为营的推进，可以大大提高开发的效率。

第一部分：实现RTP Sender

第二部分：实现RTP Receiver

第三部分：优化RTP Sender/Receiver的重传逻辑

测试你的RTP协议

！基础设施的重要性

lab2的测试涉及随机要素，和你的程序交互的进程将会随机产出一些传输错误，来检查你的传输逻辑是否能够正确的处理这些错误。因此，每一次测试的过程中，你实现的协议的行为都会有一些区别（例如这次丢了第1000个包，但是下一次测试却没丢），这种不确定性将会大大提高debug的难度。这种情况下，为这个项目搭建debug工具将会非常重要。

为了鼓励大家积极使用日志输出来辅助调试，助教团队已经在xxxxx文件提供了一些代码来方便大家为RTP协议增加日志功能：

（一些代码）

作为搭建基础设施一点小小的代价，大家需要为这个宏补齐一部分代码。相信在看到它调用的函数vsprintf之后，结合之前对man的教学，你应该已经很清楚如何完成这部分代码了。
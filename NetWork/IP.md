# IP - 网际协议

IP 是 TCP / IP 协议族中最为核心的协议。所有的 TCP、UDP、ICMP 及 IGMP 数据都是以 IP 数据报格式传输。IP 提供**不可靠、无连接**的数据报传送服务。

* 不可靠的意思是它不能保证 IP 数据报能成功地到达目的地。IP 仅提供尽力而为的传输服务。任何要求的可靠性必须由上层来提供（如 TCP）。

* 无连接的意思是 IP 并不维护任何关于后续数据报的状态信息。每个数据报的处理是相互独立的。这也说明，IP 数据报可以**不按发送顺序接收**。

## IP 首部

普通的 IP 首部长为 20 个字节，除非含有选项字段。

* 标识字段唯一地标识珠玑发送的每一份数据报。通常每发送一份报文它的值就会**加 1**。

* TTL（time-to-live）生存时间字段设置了数据报可以经过的最多路由器数。它指定了数据报的生存时间。TTL 的初始值由源主机设置（通常为 32 或 64），一旦经过一个处理它的路由器，它的值就会**减去 1**。当该字段的值为 0 时，数据报就会被丢弃，并发送 ICMP 报文通知源主机。

* 首部检验和字段是根据 IP 首部计算的检验和码。它不对首部后面的数据进行计算。 ICMP、IGMP、UDP 和 TCP 在它们各自的首部均含有同时覆盖首部和数据检验和码。

  为了计算一份数据报的 IP 校验和，首先把检验和字段置为 0。然后，对首部中每个 16bit 进行**二进制反码求和**（整个首部看成是由一串 16bit 的字组成），结果存在检验和字段。当收到一份 IP 数据报后，同样对首部中每个 16bit 进行**二进制反码求和**。由于接收方中包含了发送方存在首部中的校验和，因此，如果首部在传输过程中没有发生任何差错，那么接收方计算的结果应该为**全 1**。如果不是全 1（即检验和错误），那么 IP 就会丢弃收到的数据报。但是不生成差错报文，由上层发现丢失的数据报并进行重传。

  ICMP、IGMP、UDP 和 TCP 都采用相同的检验和算法，尽管 TCP 和 UDP 除了本身的首部和数据外，在 IP 首部中还包含不同的字段。

每一份 IP 数据报都包含源 IP 地址和目的 IP 地址，它们都是 32 bit 的值。

## IP 路由选择

如果目的主机与源主机直接相连（如点对点链路）或都在一个共享网络上（以太网或令牌环网），那么 IP 数据报就直接送到目的主机上。否则，主机把数据发送到默认的路由器上，由路由器来转发该数据报。

在一般的体制中，IP 可以从 TCP、UDP、ICMP 和 IGMP 接收数据报（即在本地生成的数据报）并进行发送，或者从一个网络接口接收数据（待转发的数据报）并进行发送。IP 层在内存中有一个**路由表**。当收到一份数据报并进行发送时，它都要对该表搜索一次。当数据报来自某个网络接口时，IP 首先检查目的 IP 地址是否为本机的 IP 地址之一或者 IP 广播地址。如果确实是这样，数据报就被送到由 IP 首部协议字段所指定的协议模块进行处理。如果数据报的目的不是这些地址，如果 IP 层被设置为路由器功能，那么就对数据报进行转发，否则丢弃数据报。

路由表中的每一项都包含下面这些信息：

* 目的 IP 地址，既可以是一个完整的主机地址，也可以是一个网络地址，由该表目中的标志字段来指定。主机地址有一个非 0 的主机号，以指定某一特定的主机，而网络地址中的主机号为 0，以指定网络中的所有主机（如以太网、令牌网）。

* 下一站（或下一跳）路由器的 IP 地址，或者有直接连接的网络 IP 地址。下一站路由器是指一个在直接相连的网络上的路由器，通过它可以转发数据报。下一站路由器不是最终目的，但是它可以把传送给它的数据报转发到最终目的。

* 标志，其中一个标志指明目的 IP 地址是网络地址还是主机地址，另一个标志指明下一站路由器是否为真正的下一站路由器，还是一个直接相连的接口。

* 为数据报的传输指定一个网络接口。

IP 路由选择是逐跳进行的。数据报在各站的传输过程中目的 IP 地址始终不变，但是封装和目的链路层地址在每一站都可以改变。

IP 路由选择主要完成以下这些功能：

1. 搜索路由表，寻找能与目的 IP 地址完全匹配的表目（网络号和主机号都要匹配）。如果找到，则把报文发送给该表目指定的下一站路由器或直接连接的网络接口（取决于标志字段的值）

2. 搜索路由表，寻找能与目的**网络号**相匹配的表目。如果找到，则把报文发送给该表目指定的下一跳路由器或直接相连的网络接口（取决于标记字段的值）。目的网络上的所有主机都可以通过这个表目来处置。这种搜索网络的匹配方法必须考虑可能的子网掩码。

3. 搜索路由表，寻找标为“默认”的表目，如果找到，则把报文送给该表目指定的下一站路由器。

如果上面这些步骤不成功，那么该数据报就不能被传送。如果不能传送的数据报来自主机，那么一般回向生成数据报的应用程序返回一个 “主机不可达” 或 “网络不可达” 的错误。

为一个网络指定一个路由器，而不必为每个主机指定一个路由器，这是 IP 路由器选择机制的另一个基本特征。这样做可以极大地缩小路由表的规模。

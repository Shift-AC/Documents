# 模块划分

- Kernel Module

    修改内核，使之支持三种新的Socket Option。
    
    1. SO_UDPFEEDBACK  
        
        int型，只对UDP有效，当此Option为非负值$fd$时，内核会通过当前的Socket向用户发送文件号为$fd$的TCP Socket的以下两种Socket Option功能中指明的信息。（默认为-1）
    
    2. SO_ARQDETAIL  
        
        bool型，只对TCP有效，在此Option为真时，TCP会将接收到的数据包的header部分发送给用户，包括ACK包。（默认为假）
    
    3. SO_RETXDETAIL  
        
        bool型，只对TCP有效，在此Option为真时，若SO_ARQDETAIL为真，则TCP会在重传数据包时将数据包头部发给用户。（默认为假）

- MPTCP Controller

    MPTCP模块使用附加了以上功能的TCP与UDP来实现用户层的MPTCP功能。其重要功能部件为：

    1. Data Source  
        
        包装后的文件描述符。此对象的功能是将数据包提供给MPTCP模块作为数据输入，一种可行的实现思路是：它是对监听Tunnel的Raw Socket的封装，此对象初始化时会将系统置于转发模式，并将所有的符合某种条件的IP包（例：TCP包）发到Tunnel，便于程序接收。

    2. Session Manager

        此模块直接接收Data Source发来的数据包，并对每一个TCP flow建立Session对象，对于Client，指定其优先级与可用的Interface集合。对于Proxy，指定其出口时使用的端口号。

    3. Scheduler

        根据数据包与Session信息，以及可用的，自定的额外信息（见Control Packet Parser节），指定数据包的发送顺序，以及数据包发送时使用的Interface。

    4. Retransmit Manager 

        接收内核发来的控制信息，并维护Session重传队列，在超时时决定是否在其他Interface上重传数据。

    5. Object Cache

        对一些特定的协议（如HTTP），对其内容进行缓存，并记录同时出现的对同一个资源的请求个数，根据个数动态干涉Session的属性。

    6. Control Packet Parser  

        通用的控制信息收发接口。提供一个特定的控制数据格式，使得在收到控制数据时能调用一个可自定义的回调函数进行处理。

    7. Receiver

        数据接收器。对于Client，在接收到下行数据时将之转发给用户。对于Proxy，在接收到上行数据时将之转发给服务器。

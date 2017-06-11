在谢希仁的《计算机网络》一书中，详尽的学习了网络协议七层塔。也明白了在TCP/IP协议中，可靠性是由传输层来保证的，而传输层的两大协议UDP与TCP，都是在基于网络层IP协议的基础上首次提供端到端的通信。其中，UDP是使用数据报提供服务的，而TCP则提供可靠的流服务。UDP提供的是不可靠的服务（相当于篮球运动员，直接将球扔向篮筐，不关注结果），TCP提供的是可靠的服务（投球不中，将一直投，知道投中为止）。在网络传输过程中，TCP的可靠性很大部分是由重传与确认机制来保证的。


----------

目录：

[TOC]


----------


“谁忽视了TCP，谁注定要推倒重来”——FTP创始人James Van Bokkelen.


----------


#How to create a tcp connection

##client
###WSAStartup

这个函数主要是用来加载Socket库，俩参数，

```
WSAStartup(
      _In_ WORD wVersionRequired,
      _Out_ LPWSADATA lpWSAData);
```

- wVersionRequired，指明需要的Socket版本；一般使用宏MAKEWORD(2,2)。

- lpWSAData，是一个输出变量，指系统实际返回的库信息。

```
SOCKET _tcp_socket;
WSADATA wsa;
if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0){
	perror("WSASartup error !");
}
```
###socket

创建socket，函数原型是这样滴，

```
socket (
       _In_ int af,
       _In_ int type,
       _In_ int protocol);
```

- af 协议族，常用的协议族有AF_INET、AF_INET6、AF_LOCAL（或称AF_UNIX，Unix域Socket）、AF_ROUTE等。
- type Socket类型，常用的socket类型有SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等。
 - TCP ，流式Socket（SOCK_STREAM）是一种面向连接的Socket，针对于面向连接的TCP服务应用。
 - UDP，数据报式Socket（SOCK_DGRAM）是一种无连接的Socket，对应于无连接的UDP服务应用。
- protocol 指定协议，常用协议有IPPROTO_TCP、IPPROTO_UDP、IPPROTO_STCP、IPPROTO_TIPC等，分别对应TCP传输协议、UDP传输协议、STCP传输协议、TIPC传输协议。

**注意**：type和protocol不可以随意组合，如SOCK_STREAM不可以跟IPPROTO_UDP组合。当第三个参数为**0**时，会自动选择第二个参数类型对应的默认协议。

```
_tcp_socket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
if (_tcp_socket == INVALID_SOCKET){
	perror("socket error !");
	WSACleanup();
}
```

###sockaddr_in 

我更喜欢将变量定义为xxx_config，因为在我看来，这很大程度上就相当于一个配置文件，
```
struct sockaddr_in {
      short   sin_family;
      u_short sin_port;
      struct  in_addr sin_addr;
      char    sin_zero[8];
};
```
- sin_family 地址家族，通常大多用的是都是AF_INET,代表TCP/IP协议族。
- sin_port 端口，这里必须采用网络数据格式，
 - htons ，将端口号由主机字节序转换为网络字节序的整数值。
- sin_addr 存储IP地址，这里服务端和客户端是有所区别的。
 - inet_addr  **客户端使用**，将一个IP字符串转化为一个网络字节序的整数值，用于sockaddr_in.sin_addr.s_addr。
 - htonl(INADDR_ANY) **服务端使用**，INADDR_ANY表示任何IP地址。
 
```
SOCKADDR_IN remote_config;
remote_config.sin_port = htons(port);
remote_config.sin_family = AF_INET;
remote_config.sin_addr.S_un.S_addr = inet_addr(addr);
```
###connect

TCP客户端使用它，通过调用connect建立一个连接（虚电路），

```
int PASCAL FAR connect (
    _In_ SOCKET s,
    _In_reads_bytes_(namelen) const struct sockaddr FAR *name,
    _In_ int namelen);
```

无需赘言，按照以下方式调用。如果调用失败（SOCKET_ERROR），不要重新调用connect，最好closesocket然后socket重新得到一个新的，继而进行connect。

如果客户端通过WSAGetLastError获得返回值（10061），可能是，

 - 服务端没有运行。
 - IP地址设置有误。

另外，一般UDP客户端不需要调用调用connect，但也并不是不可以。这里，如果调用两次及以上次数connect函数，将造成（10048）错误。

```
if (connect(_tcp_socket, (sockaddr *)&remote_config, sizeof(remote_config)) == SOCKET_ERROR){
	perror("connect error !");
	closesocket(_tcp_socket);
	WSACleanup();
}
```

##server

###bind

作为服务端，必须绑定一个固定的端口，以等待客户端的连接。
```
int PASCAL FAR bind (
    _In_ SOCKET s,
    _In_reads_bytes_(namelen) const struct sockaddr FAR *addr,
    _In_ int namelen);
```

这里再次比较一下，客户端与服务端配置文件的写法，

|客户端|服务端|
|--|--|
|SOCKADDR_IN remote_config;|SOCKADDR_IN local_config;|
|remote_config.sin_port = htons(port);|local_config.sin_port = htons(port);|
|remote_config.sin_family = AF_INET;|local_config.sin_family = AF_INET;|
|remote_config.sin_addr.S_un.S_addr = inet_addr(addr);|local_config.sin_addr.S_un.S_addr = htonl(INADDR_ANY);|


特别注意到sin_addr.S_un.S_addr的转换关系，
如果写错了，将导致通信失败。
```
if (bind(_tcp_socket_listen, (struct sockaddr *)&local_config, sizeof(local_config)) == SOCKET_ERROR){
	perror("bind error !");
	closesocket(_tcp_socket_listen);
	WSACleanup();
	return;
}
```

###listen

UDP服务端无需做任何准备，因为这一切都市通过sendto/recvfrom建立起来的。但是TCP就不同了，必须使用listen为接纳来自客户端的数据做好准备。

```
int PASCAL FAR listen (
    _In_ SOCKET s,
    _In_ int backlog);
```

 - backlog，为等待连接的队列长度，也就是当几个客户端同一时间请求时，将放入该队列中，按先后顺序逐个处理。（注意，该参数**并不是**服务端可接受的最大连接数）。

```
#define SOMAXCONN       5
```
系统内部，定义了一个最大**正在连接**，队列长度数宏SOMAXCONN。

```
if (listen(_tcp_socket_listen, SOMAXCONN) == SOCKET_ERROR){
	perror("listen error !");
	closesocket(_tcp_socket_listen);
	WSACleanup();
	return;
}
```

###accept

accept函数，用来接受客户端对服务器发起的连接请求。accept函数牵涉到两个socket，

- 一个是accept函数的第一个参数（_In_ SOCKET s），这里应该传入的是服务端本身的SOCKET。
- 二个是accept函数的返回值，它一般代表着新来的客户端SOCKET，除非返回值为SOCKET_ERROR之外。

这里 (struct sockaddr*)&client_config，并不需要服务端自己配置，而是直接填充接收到的客户端的参数。

```
SOCKET PASCAL FAR accept (
       _In_ SOCKET s,
       _Out_writes_bytes_opt_(*addrlen) struct sockaddr FAR *addr,
       _Inout_opt_ int FAR *addrlen);
```

正是由于服务端存在多个SOCKET，除了接受到的N个客户端之外，服务端本身还有一个自己的SOCKET，在做善后工作时，记得closesocket服务端本身。

```
SOCKADDR_IN client_config;
int addr_len = sizeof(client_config);
SOCKET client_tcp_socket = accept(_tcp_socket_listen, (struct sockaddr*)&client_config, &addr_len);
if (client_tcp_socket == SOCKET_ERROR){
	perror("accept socket error !");
	break;
}
```

##send 与 sendto

以前，我总是混用，或者是不知其所以然的错误使用这两个函数。正是因为我不知道他们有什么区别，直到我阅读了《[Windows Sockets网络编程](http://blog.csdn.net/williamaiden/article/details/72859325)》一书，

一言蔽之，

- send 在“TCP”上发送数据。
- sendto 在“UDP”上发送数据。

注意使用时相互搭配即可，send与recv互补，而sendto与recvfrom是搭档。虽然，send与sendto区别不大，除了以下两点，

- 所允许的“socket”状态。
- 从哪里获得“socket”配置。

UDP认为任何socket都是有效的，使用sendto之前不需要做任何准备。而从哪里获取socket配置呢？UDP将sendto中的参数，作为发送数据的目的地址，即使在之前你调用了connect（所以UDP根本不需要调用connect，虽然不会报错）。而TCP恰好相反，如果TCP中使用sendto发送数据，好像也不会报错，但是他会使用connect时的socket配置，而忽略sendto参数中的配置。

正是如此，总是在TCP上使用send/recv。而在UDP上使用sendto/recvfrom。

```
int Server::sendTcpData(SOCKET* socket, const char* data, const int dataLength){
	if (data != NULL && *socket != INVALID_SOCKET && dataLength > 0){
		return send(*socket, data, dataLength, 0);
	}
	return 0;
}
```
- send返回值， 一般表示实际发送出去的字节数，除非发送失败。
- recv返回值，一般表示实际接收到的字节数，除非链路异常。

```
int Server::recvTcpData(SOCKET* socket, char* data, int bufferLength){
	int ret = 0;
	if (data != NULL && *socket != INVALID_SOCKET && bufferLength > 0){
		ret = recv(*socket, data, bufferLength, 0);
		if (ret > 0){
			data[ret] = '\0';
			printf("%s\n", data);
		}
	}
	return ret;
}
```


##closesocket

TCP中，如果需要优雅的关闭，可以去研究一下shutdown函数，粗暴一点就直接调用closesocket吧。（UDP中，应该始终是只调用closesocket。虽然也可以调用shutdown但没有任何意义）。

closesocket函数一般是不会失败的，即使重复调用。因为它只是将socket资源还给协议栈，这里需要注意closesocket函数不是阻塞的，总是一调用直接就返回，不管是真的关闭了。返回值只是表示closesocket该函数有没有调用成功。

如果socket已经成功关闭，再执行任何socket调用，都会得到（10038）错误。

##WSACleanup

与WSAStartup是孪生兄弟，用于加载套接字DLL库，让所有socket函数可以有效（无效）的调到，一般只需要调用一次即可。

- WSAStartup 加载套接字库。
- WSACleanup 卸载套接字库，释放所有资源，后续所有socket都会失效。



这两个函数，一般不会失败。


#client code

client.h

```
#pragma once

#include <windows.h>
#pragma comment(lib,"ws2_32.lib") 

class Client
{
public:
	Client(char* addr, int port);
	~Client();
	int sendTcpData(const char* data, const int dataLength);
	int recvTcpData(char* data, int bufferLength);
private:
	SOCKET _tcp_socket;
};


```


client.cpp

```
#include "Client.h"
#include <stdio.h>

Client::Client(char* addr, int port)
{
	WSADATA wsa;
	if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0){
		perror("WSASartup error !");
		return;
	}
	_tcp_socket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (_tcp_socket == INVALID_SOCKET){
		perror("socket error !");
		WSACleanup();
		return;
	}
	SOCKADDR_IN remote_config;
	remote_config.sin_port = htons(port);
	remote_config.sin_family = AF_INET;
	remote_config.sin_addr.S_un.S_addr = inet_addr(addr);
	if (connect(_tcp_socket, (sockaddr *)&remote_config, sizeof(remote_config)) == SOCKET_ERROR){
		perror("connect error !");
		closesocket(_tcp_socket);
		WSACleanup();
		return;
	}
}


Client::~Client()
{
	if (_tcp_socket != INVALID_SOCKET){
		if(closesocket(_tcp_socket) == 0){
			_tcp_socket = INVALID_SOCKET;
			WSACleanup();
			return;
		}
		perror("close socket error.");
	}
}

int Client::sendTcpData(const char* data, const int dataLength){
	if (data != NULL && _tcp_socket != INVALID_SOCKET && dataLength > 0){
		return send(_tcp_socket, data, dataLength, 0);
	}
	return 0;
}

int Client::recvTcpData(char* data, int bufferLength){
	int ret = 0;
	if (data != NULL && _tcp_socket != INVALID_SOCKET && bufferLength > 0){
		ret = recv(_tcp_socket, data, bufferLength, 0);
		if (ret > 0){
			data[ret] = '\0';
			printf("%s\n",data);
		}
	}
	return ret;
}

int main(){
	Client c("127.0.0.1", 8086);

	char* strNum = "1234567890";
	c.sendTcpData(strNum, strlen(strNum));
	char recv[1024];
	c.recvTcpData(recv, sizeof(recv));

	char* strStr = "abcdefghi";
	c.sendTcpData(strStr, strlen(strStr));
	c.recvTcpData(recv, sizeof(recv));

	return 0;
}
```

#server code

server.h

```
#pragma once

#include <windows.h>
#pragma comment(lib,"ws2_32.lib") 

class Server
{
public:
	Server(int port);
	~Server();
	void acceptTcpSocket();
	int sendTcpData(SOCKET* socket, const char* data, const int dataLength);
	int recvTcpData(SOCKET* socket, char* data, int bufferLength);
private:
	SOCKET _tcp_socket_listen;
};


```

server.cpp

```
#include "Server.h"
#include <stdio.h>

Server::Server(int port)
{
	WSADATA wsa;
	if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0){
		perror("WSASartup error !");
		return;
	}
	_tcp_socket_listen = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (_tcp_socket_listen == INVALID_SOCKET){
		perror("socket error !");
		WSACleanup();
		return;
	}
	SOCKADDR_IN local_config;
	local_config.sin_port = htons(port);
	local_config.sin_family = AF_INET;
	local_config.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
	if (bind(_tcp_socket_listen, (struct sockaddr *)&local_config, sizeof(local_config)) == SOCKET_ERROR){
		perror("bind error !");
		closesocket(_tcp_socket_listen);
		WSACleanup();
		return;
	}
	if (listen(_tcp_socket_listen, SOMAXCONN) == SOCKET_ERROR){
		perror("listen error !");
		closesocket(_tcp_socket_listen);
		WSACleanup();
		return;
	}
}


Server::~Server()
{
	if (_tcp_socket_listen != INVALID_SOCKET){
		if (closesocket(_tcp_socket_listen) == 0){
			_tcp_socket_listen = INVALID_SOCKET;
			WSACleanup();
			return;
		}
		perror("close socket error.");
	}
}

void Server::acceptTcpSocket(){
	bool flag = true;
	while (flag){
		SOCKADDR_IN client_config;
		int addr_len = sizeof(client_config);
		SOCKET client_tcp_socket = accept(_tcp_socket_listen, (struct sockaddr*)&client_config, &addr_len);
		if (client_tcp_socket == SOCKET_ERROR){
			perror("accept socket error !");
			break;
		}
		while (flag){
			char recv[1024];
			if (recvTcpData(&client_tcp_socket, recv, sizeof(recv)) <= 0){
				perror("recv error !");
				break;
			}
			char* data = "Hi, I'm William Aiden !";
			if (sendTcpData(&client_tcp_socket, data, strlen(data)) <= 0){
				perror("send error !");
				break;
			}
		}
	}
}

int Server::sendTcpData(SOCKET* socket, const char* data, const int dataLength){
	if (data != NULL && *socket != INVALID_SOCKET && dataLength > 0){
		return send(*socket, data, dataLength, 0);
	}
	return 0;
}

int Server::recvTcpData(SOCKET* socket, char* data, int bufferLength){
	int ret = 0;
	if (data != NULL && *socket != INVALID_SOCKET && bufferLength > 0){
		ret = recv(*socket, data, bufferLength, 0);
		if (ret > 0){
			data[ret] = '\0';
			printf("%s\n", data);
		}
	}
	return ret;
}

int main(){
	Server s(8086);
	s.acceptTcpSocket();

	return 0;
}
```

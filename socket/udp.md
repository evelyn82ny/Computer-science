# UDP (User Datagram Protocol)

UDP도 TCP와 같은 통신 프로토콜이다.<br>

TCP는 소켓끼리의 1대 1 연결이고, 소켓 전달시 SEQ, ACK 같은 신호로 데이터의 전송을 보장해준다.<br>
하지만 UDP는 데이터의 전송도 보장해주지 않고 그저 빠른 속도만 보장한다. 즉, 안전성보다 성능이 중요하다면 UDP를 사용하면된다.<br>

- 데이터의 전송을 보장하지 않아 SEQ, ACK 같은 신호가 없다.
- 데이터의 흐름을 제어하지 않고 일단 보낸다.(= flow control 없음)
- 빠르게 보내는 것만 제공하므로 TCP보다 구조가 단순하다.
- 전송 체크를 따로 하지 않기 때문에 속도가 빠르다.
- 1대 1 방식의 TCP와 다르게 여러 host와 동시에 통신이 가능하다. (broadcast, multicast)
- 여러 host와 동시에 통신이 가능하므로 connection 이라는 개념이 적용되지 않아 통신할 때마다 수신자를 지정해야한다.


## UDP의 데이터 송수신 방법

TCP는 소켓끼리의 1대 1 연결이지만 UDP에는 연결이라는 개념이 존재하지 않는다. 누구와도 연결이 가능하기 때문에 서버 소켓, 클라이언트 소켓이라는 구분이 없다.<br>
즉, 하나의 소켓으로 둘 이상의 영역과 데이터 송수신이 가능하다.<br>

### sendto

```c
#include <sys/socket.h>

ssize_t sendto(int sock, void *buff, size_t nbytes, int flags,
                struct sockaddr *to, socklen_t addrlen);
```

- sock : 데이터 전송에 사용될 UDP 소켓의 파일 디스크립터
- buff : 전송할 데이터를 저장하고 있는 버퍼 주소 값
- nbytes : 전송할 데이터 크기(바이트 단위)
- flags : 옵션 지정에 사용되는 매개변수로 지정할 옵션 없으면 0 전달
- to : 목적지 주소정보를 담고 있는 sockaddr 구조체 변수의 주소 값
- addrlen : 매개변수 to로 전달된 주소 값의 구조체 변수 크기(value)

UDP는 여러 소켓과 연결이 가능하므로 데이터를 전송할 때마다 **to** 매개변수에 목적지에 대한 정보를 전달해야한다.<br>
sendto() 성공 시 ```return 보낸 문자 수(byte)```, 실패 시 ```return -1``` <br>

### recvfrom

```c
#include <sys/socket.h>

ssize_t recvfrom(int sock, void *buff, size_t nbytes, int flags,
                    struct sockaddr *from, socklen_t *addrlen);
```

- sock : 데이터 수신에 사용될 UDP 소켓의 파일 디스크립터
- buff : 데이터 수신에 사용될 버퍼 주소 값
- nbytes : 수신할 최대 바이트 수 전달. buff가 가리키는 버퍼의 크기를 넘을 수 없음
- flags : 옵션 지정에 사용되는 매개변수로 지정할 옵션 없으면 0 전달
- from : 발신지 정보를 채워 넣을 sockaddr 구조체 변수의 주소 값
- addrlen : sockaddr 구조체의 크기를 할당한 pointer

from에 비어있는 sockaddr 구조체를 넘겨주면 커널에서 발신자 주소를 알아서 채워준다.<br>
recvfrom() 성공 시 ```return 받은 문자 수(byte)```, 실패 시 ```return -1``` <br>

### TCP / UDP 포트번호 관리

하나의 PC에 TCP 소켓과 UDP 소켓의 Port 번호가 같아도 상관없다.<br>
같은 PC에 존재하는 소켓이므로 IP는 같다. 하지만 Port 번호까지 같아도 상관없는 이유는 TCP/UDP 따로 관리되기 때문이다.<br><br>

## UDP 기반 echo server/client

### UDP echo server

```c
1  serv_sock = socket(PF_INET, SOCK_DGRAM, 0);
2  if(bind(serv_sock, (struct sockaddr*) &serv_adr, sizeof(serv_addr)) == -1)
3       error_handling("bind() error");
4   
5  while(1) {
6    
7       clnt_adr_sz = sizeof(clnt_adr);
8       str_len = recvfrom(serv_sock, message, BUF_SIZE, 0,
9                        (struct sockaddr*)&clnt_adr, &clnt_adr_sz);
10
11      message[str_len] = 0; // null 추가
12      printf("%s", message);
13
14      sendto(serv_sock, message, str_len, 0,   // null 전까지 전송
15              (struct sockaddr*)&clnt_adr, clnt_adr_sz);
16 }
```

TCP는 데이터의 경계가 없어 받고 싶은 만큼만 받아도 되지만 UDP는 데이터의 경계가 존재한다. 즉, 한번에 모두 읽기위해 recvfrom() 이 1번 호출된다.(line 8 )<br>

### UDP echo client

```c
1  while(1) {
2       
3       fputs("Insert message(q to quit) : ", stdout);
4       fgets(message, sizeof(message), stdin);
5       if(!strcmp(message, "q\n") || !strcmp(message, "Q\n")) break;    
6       
7       sendto(sock, message, strlen(message), 0,
8                  (struct sockaddr*)&serv_adr, sizeof(serv_adr));
9       
10      adr_sz = sizeof(from_adr);
11      str_len = recvfrom(sock, message, BUF_SIZE, 0,
12                 (struct sockaddr*)&from_adr, &adr_sz);
13 
14      message[str_len] = 0;
15      printf("Message from server : %s", message);
16 }
```

echo client에서 서버로 데이터를 보내기 위해 IP, Port 번호를 설정해야 하는데 이는 connect()를 호출할 때 커널에서 자동으로 할당해주기 때문에 bind()를 호출할 필요없다.<br>
하지만 server는 bind() 를 호출해야 하는데 외부에서 서버의 주소를 알아야 하기 때문에 bind() 호출로 주소를 명시한다.<br><br>
UDP는 소켓끼리 1대 1 연결이 아니므로 데이터를 전송하기 위해 sendto()를 호출할 때마다 상대방 주소를 작성해야하지만, 자신의 IP, Port 주소는 sendto() 첫 호출때만 할당된다. 즉, sendto()를 호출했는데 클라이언트의 IP,Port 번호가 이미 할당된 상태라면 넘어간다.<br>
echo server와 마찬가지로 UDP는 데이터의 경계가 존재해 모든 데이터를 한번에 읽기 위해 recvfrom()를 1번 호출한다.<br>

이렇게 UDP는 여러 소켓과 통신이 가능하므로 sendto()를 호출할 때마다 UDP 소켓에 목적지의 IP, Port번호를 등록하고 전송이 끝나면 목적지 주소를 삭제한다. 이를 **unconnected UDP**라고 한다.<br><br>

## connected UDP socket

```c
1  sock = socket(PF_INET, SOCK_DGRAM, 0);
2  if(sock == -1)
3       error_handling("sock() error");
4  
5  memset(&serv_adr, 0, sizeof(serv_adr));  
6  serv_adr.sin_family = AF_INET;
7  serv_adr.sin_addr.s_addr = inet_addr(argv[1]);
8  serv_adr.sin_port = htons(atoi(argv[2]));
9 
10 connect(sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr));
```

UDP 소켓은 여러 UDP 소켓과 통신이 가능하므로 sendto()를 호출할 때마다 상대방의 주소를 작성해야한다. 하지만 connect()를 사용하면 UDP 소켓에 상대방의 IP, Port 주소가 고정 등록되어 sendto()가 아닌 read()/write()로 데이터 송수신이 가능하다.<br>
즉, sendto()를 사용하지 않으니 데이터를 보낼 때마다 상대방 주소를 작성할 필요가 없다. 이와 같은 소켓을 connected UDP 소켓이라 한다.<br>

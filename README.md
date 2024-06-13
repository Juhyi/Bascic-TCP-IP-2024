# Basic-Network-2024
리눅스 기반 네트워크 프로그래밍-2024 레포지토리

## 1일차(2024-06-11)
* 네트워크 프로그래밍, 소켓
    - 소켓 
        + 물리적으로 연결된 네크워크 상에서의 데이터 송수신에 사용할 수 있는 소프트웨어적인 장치,
        + 소켓을 이용하여 데이터를 주고 받을 수 있음(네트워크 프로그래밍을 소켓 프로그래밍이라고 부르는 이유) 

    - 소켓 구현
        **핵심(server 구현을 위해 4개의 함수를 호출해야함)**
        + 소 -> 소켓(socket) : 휴대폰 구입 | 소켓 생성
        + 말 -> 바인드 (bind) : 전화번호 할당 | IP주소와 PORT번호 할당
        + 리 -> 리슨 (listen) : 개통 | 연결요청 가능상태로 변경
        + 아 -> 엑셉트 (accept)  : 전화를 받음. ㅣ (연결 요청에 대한 수락)

            + socket() 
                - 서버와 클라이언트가 통신을 하기 위해서는 소켓 라이브러리 사용, 소켓을 만들기 위해서 사용하는 함수
                - 함수 형태 

                ```C
                 #include <sys/socket.h>
                 int socket(int domain, int type, int protocol);
                ```
                
                - 반환 : 해당 소켓을 가리키는 소켓 디스크립터(socket descriptor,시스템이 특정 파일에 할당해준 정수값)를 반환, -1이 반환되면 생성 실패, 0 이상의 값이 나오면 socket descriptor 반환.
                - domain : 어떤 영역에서 통신할 것이니지에 대한 영역을 지정. 프로토콜 family를 지정해 주는 것 -> AF_UNIX(프로토콜 내부에서), AF_INEF(IPv4), AF_INET6(IPv6)
                - type : 어떤 타입의 프로토콜을 사용할 것인지 대해 설정. -> SOCK_STREAM(TCP), AF_DGRAM(UDP), SOCK_RAM(사용자 정의)
                - protocol : 어떤 프로토콜의 값을 결정하는 것 -> 대부분 0 그 외의 경우 IPPROTO_TCP, IPPROTO_UDP
            + bind() 
                - 소켓에 주소를 할당하는 함수
                - 함수 형태

                ```C
                #include <sys/socket.h>
                int bind(int sockfd, struct sockaddr *addr, socklen_t addrlen);
                ```
                
                - 반환 : 성공시 0, 실패시 -1
                - 3가지 인자를 전달하여 소켓에 주소를 할당할 수 있다. 즉 socket()함수로 받아온 디스크립터 sockfd가 존재하는데, 이 디시크립터 파일에 해당하는 소켓에 serv_addr 주소를 할당하겠다는 의미
                - sockfd : socket()함수를 통해 배정받은 디스크립터 번호 serv_sock
                - *addr : IP주소와 PORT번호를 지정한 serv_addr 구조체
                - addrlen : 주소정보를 담은 변수의 길이
            + listen() 
                -  연결요청을 대기하는 함수.  

                ```C
                #include <sys/socket.h>
                int listen(int sock, int backlog);
                ```
                
                - bind를 통해 하나의 소켓에 ip주소와 port 번호를 할당했으니, 이제 클라이언트가 해당 소켓에 연결할 수 있도록 요청을 대기하는 상태로 만들어줘야함. 즉, listen함수가 호출된 이후부터 클라이언트에서 connect를 호출할 수 있음.
                - 반환 : 성공시 0, 실패시 -1 
                - sock : 소켓 디스크립터 번호
                - backlog : 연결요청을 대기하는 큐의 크기
                    + 지정한 디스크립터의 소켓이 리스닝소케이 되며, backlog만큼의 큐 공간을 가짐
            + accept()
                - 연결 요청을 수락하는 함수

                ```C
                #include <sys/socket.h>
                int accept(int sock, struct sockaddr*addr, socklen_t *addrlen);
                ```
                
                - 대기중인 클라이언트의 요청을 차례로 수락함으로써 데이터를 주고 받을 수 있게 됨. 
                - 반환 : 성공시 새로운 디스크립트 번호, 실패시 -1 
                - (주의. 앞서 이용하던 서버소켓(리스닝소켓)은 연결요청을 대기시키는 과정까지를 담당. accept()를 통해 새로 할당받은 소켓을 이용해 데이터 송수신을 할 수 있는 것)
                - sock : 서버소켓(리스닝 소켓)의 디스크립터 번호
                - addr : 대기 큐를 참조해 얻은 클라이언트의 주소 정보
                - addrlen : addr변수의 크기
                - *cf) clnt_sock에 새로운 소켓 디스크립터를 반환할 때에는 대기 큐에서 첫번째로 대기중인 연결요청을 참조한다. 이 때, 대기 큐가 비어있는 상황이라면 새로운 요청이 올 때까지 accept값은 반환되지 않고 대기(blocking)한다.*
            + connect()
                - client socket -> 전화를 거는 함수

                ```C
                #include <sys/socket.h>
                int connect(int sockfd, struct sockaddr *serv_addr, socklen_t addrlen)
                ```
                
                - 클라이언트에서 connect() 시스템 콜을 호출하면 listen()|accept()로 대기중이던 서버에 연결이 요청됨.서버는 accept() 함수를 정상 리턴함으로써 두 호스트간 연결이 성립됨.
                - sockfd : sockfd의 디스크립터 번호
                - *addr : 연결을 요청할 서버의 구조체 주소
                - addrlen : 서버 구조체 주소 변수의 길이

    - 리눅스 기반에서 실행하기
        + 실행하기 위해서는 서버를 먼저 실행해야 함
            - ./(실행파일명) 9190(포트번호)
        + 클라이언트 프로그램 실행
            - ./(실행파일명) 127.0.0.1(루프백주소(loopback address). 컴퓨터 자신의 ip주소를 의미. 한대의 컴퓨터 내에서 서버와 클라이언트 프로그램을 실행시키는 경우 사용.) 9190(포트번호)
            - 다른 컴퓨터에서 각각 실행할 경우에는 루프백 주소를 대신에 서버의 ip주소(외부 ip주소)를 입력하면 됨.
            - 외부 ip주소 : 통신회사에서 들어오는 외부 인터넷 회선에 할당되는 ip주소
            - 내부 ip주소 : 외부 ip주소를 받아 공유기에 연결된 각각의 컴퓨터와 휴대폰에 할당되는 ip주소

* 리눅스 기반 파일 조작
    - 리눅스에서의 소켓조작은 파일조작과 동일함. 파일 입출력 함수를 소켓 입축력(네트워크상에서의 데이터 송수신)에 사용할 수 있음.
    *cf)윈도우는 구분함*

    - 리눅스가 독립적으로 제공하는 파일 디스크립터 
        + 별도의 생성과정을 거치지 않아도 프로그램이 실행되면 자동으로 할당됨.
        + 0 - 표준입력 : Standard Input
        + 1 - 표준출력 : Standard Output
        + 2 - 표준에러 : Standard Error

    - 파일 열기
    - 파일 닫기
    - 파일 데이터 쓰기
    - 파일에 저장된 데이터 읽기
    - fd_seri.c 예제 

* 소켓의 프로토콜, 데이터 전송 특성
    - 프로토콜 - 컴퓨터 상호간의 대화에 필요한 통신규약(약속)
        + 프로토콜 체계(Protocol Family) - 프로토콜의 부류정보
            + PF_INET(IPv4 인터넷의 프로토콜 체계), PF_INET6(IPv6 인터넷의 프로토콜 체계) ...
        + 소켓의 타입(Type) - 소켓의 데이터 전송방식
            + 연결지향형 소켓 (SOCK_STREAM, TCP방식)
                - 중간에 데이터가 소멸되지 않고 목적지로 전송
                - 전송 순서대로 데이터가 수신됨
                - 전송되는 데이터의 경계(Boundary)가 존재하지 않는다. (read함수의 호출횟수와 write함수의 호출횟수는 연결지향형 소켓의 경우 큰 의미x)
                - 연결지향형 소켓은 자신과 연결된 상대 소켓의 상태를 파악해가며 데이터를 전송함. 데이터가 제대로 전송되지 않으면 데이터를 재전송. 소켓에 존재하는 버퍼가 다 차면 전송하지 않음
            + 비 연결지향형 소켓(SOCK_DGRAM, UDP방식)
                - 전송된 순서에 상관없이 가장 빠른 전송을 지향
                - 전송된 데이터는 손실의 우려가 있고, 파손의 우려가 있다.
                - 전송되는 데이터의 경계(Boundary)가 존재한다.
        
* 소켓에 할당되는 ip주소, port 번호
    - IP주소 (인터넷 주소)
        + IPv4 기준의 4바이트 IP주소는 네트워크 주소와 호스트(컴퓨터) 주소로 나뉨.
        + 주소의 형태에 따라서 A(0~127, 첫번째비트 0), B(128~191, 첫번째비트 10), C(192~223, 첫번째비트 110), D, E클래스로 분류됨. (E는 일반적이지 않은 예약된 주소 체계)
        
        ![IPv4 주소체계](https://raw.githubusercontent.com/Juhyi/Bascic-Network-Programming-2024/main/imges/net001.png)

    - Port (포트번호)
        + 클라이언트 프로그램이 네트워크 상의 특정 서버 프로그램을 지정하는 방법으로 사용. 지금 실행되고 있는 프로그램
        + 포트번호는 중복으로 소켓에 할당할 수 없다. 
        + 데이터 전송방식이 다를경우에는 port 번호가 같아도 상관없다.
        + 16비트 (2바이트, 0 ~ 65535)              개인적으로 사용할때는 10000번 이상의 번호를 사용

* IPv4 기반의 주소표현을 위한 구조체
    - bind 함수에 주소정보를 전달하는 용도로 사용

    ```C
    struct sockaddr_in
    {
        sa_family_t sin_family // 주소 체계
        uint16_t sin_port // 16비트 TCP/UDP PORT 번호
        struct in_addr // 32비트 IP주소
        char sin_zero[8] // 사용되지 않음, 반드시 0으로 채워야함
    }
    ```

* 네트워크 바이트 순서와 인터넷 주소 변환
    - 바이트(Order) 순서와 네트워크 바이트 순서
        + 빅 엔디안 : 상위바이트의 값을 작은 번지수에 저장하는 방식 
        + 리틀 엔디안 : 상위바이트의 값을 큰 번지수에 저장하는 방식
        + CPU의 데이터 저장방식을 의미하는 '호스트 바이트 순서(Host Byte Order)'는 CPU마다 다름
        + 문제해결을 위해 네트워크상으로 데이터를 전송할 때 데이터의 배열을 "빅 엔디안" 방식으로 통일(네트워크 바이트 순서)
        + 리틀 엔디안 시스템에서는 데이터를 전송하기 전에 빅 엔디안 정렬방식으로 재정렬 해야함.

        ![빅/리틀 에디안](https://raw.githubusercontent.com/Juhyi/Bascic-Network-Programming-2024/main/imges/net002.png)


    - 바이트 순서의 변환(Endian Conversions)
        + unsigned short htons(usigned short); shott형 데이터를 호스트 바이트 순서에서 네트워크 바이트 순서로 변환
            + unsigned short ntohs(usigned short);
            + unsigned long htons(usigned long);
            + unsigned long ntohs(usigned long);
            + 변환하는 것 - h:host, n:network s:shoty l:long  to:to 


## 2일차(2024-06-12)
* 네트워크 바이트 순서와 인터넷 주소 변환
    - 문자열 정보 -> 정수 변환
        + inet_addr() 
            ```C
            in_addr_t inet_addr(const char * string);
            ```
            
            + 성공 시 빅엔디안으로 변환된 32bit 정수값, 실패 시 INADDR_NONE 반환.
            + (e.g) 211.214.107.88와 같이 점이 찍힌 10진수로 표현된 문자열을 전달하면, 해당 문자열 정보를 참조해서 IP주소 정보를 32biyt 정수형으로 변환.
            + 변환과정에서 네트워크 바이트 순서로 변환도 동시에 진행.
            + inet_addr()는 변환된 IP주소 정보를 구조체 sockaddr_in에 선언되어 있는 in_addr 구조체 변수에 대입하는 과정을 거쳐야 함.
            
        + inet_aton()

            ```C 
            int inet_aton(const char * string, struct in_addr * addr);
            ```
            + 기능상으로 int_addr 함수와 동이하나 in_addr형 구조체 변수에 변환의 결과가 저장된다는 차이가 있음
            + 활욛도 측면에선 inet_aton이 높음.
            + string : 변환할 IP주소 정보를 담고 있는 문자열의 주소 값 
            + addr : 변환된 정보를 저장할 in_addr 구조체 변수의 주소 값 
            + 별도의 대입 과정을 거칠 필요 없이 인자로 in_addr 구조체 변수의 주소값을 전달하여 변환된 값이 자동으로 in_addr 구조체 변수에 저장됨.
        
        + inet_ntoa()
            ```C
            char * inet_ntoa(struct in_addr adr);
            ```
            + 성공 시 반환된 문자열의 주소 값, 실패 시 -1 
            + inet_aton 함수의 반대 기능을 제공
            + 네트워크 바이트 순서로 정렬된 정수형 IP 주소 정보를 우리가 쉽게 인식할 수 있는 문자열의 형태로 반환
            + 사용시 주의 - 함수가 재호출되기 전까지만 반환된 문자열의 주소값이 유효함. 오랫동안 문자열 정보를 유지하려면 별도의 메모리 공간에 복사를 해두어야 함.

    - 인터넷 주소의 초기화
        + 소켓생성과정에서 흔히 보이는 인터넷 정보의 초기화 방법
            ```C
            struct sockaddr_in_addr;    
            char *serv_ip = "211.217.168.13";    // ip주소 문자열 선언 
            char *serv_port="9190";     //port번호 문자열 선언
            memset(&addr, 0, sizeof(addr));     // 구조체 변수 addr의 모든 멤버 0으로 초기화
            addr.sin_family = AF_INET;  // 주소체계 지정
            addr.sin_s_addr=inet_addr(serv_ip);     // 문자열 기반의 ip주소 초기화
            addr.sin_port=htons(atol(serv_port));   // 문자열 기반의 port번호 초기화   
            /* 문자열로 표현된 IP주소와 PORT번호를 기반으로 하는 sockaddr_in 구조체 변수의 초기화 과정*/
            ```
            + memset 함수 
                + 동일한 값으로 바이트 단위 초기화를 할 때 호출하는 함수, 초기화 대상은 첫번째 인자인 구조체 변수 addr의 주소값, 변수 addr
                + addr을 0으로 초기화하는 이유 - sockaddr_in 구조체 멤버 sin_zero를 0으로 초기화하기
            + atoi 함수
                + 문자열로 표현되어 있는 값을 정수로 변환해서 반환

    - 클라이언트의 주소정보 초기화
        + INADDR_ANY
            
            ```c
            struct sockaddr_in addr;
            char *serv_port="9190";
            memset(&addr, 0, sizeof(addr));
            addr.sin_family=AF_INET;
            addr.sin_addr.s_addr=htonl(INADDR_ANY);
            addr.sin_port=htons(atoi(serv_port))
            ```
            + 현재 실행중인 컴퓨터의 ip를 소켓에 부여할 때 사용
            + 컴퓨터 내에 두 개 이상의 IP를 할당받아서 사용하는 경우(Multi-homed 컴퓨터, 라우터), 할당받은 ip중 어떤 주소를 통해서 데이터가 들어오더라도 PORT번호만 일치하면 수신할 수 있게 됨.

    - 소켓에 인터넷 주소 할당
        + 초기화된 주소정보 소켓에 할당 -> bind()  
            ```C
            // 실제 사용 (원형은 1일차에 정리해둠)
            if(bind(serv_sock, (struct sockaddr*) &serv_addr, sizeof(serv_addr))== -1) { ... }
            ```
            + 함수 호출이 성공하면, 첫번째 인자에 해당하는 소켓에 두번째 인자로 전달된 주소정보가 할당됨

* TCP/ UDP
    - TCP/ip 프로토콜 스택
        + 인터넷 기반의 데이터 송수신을 목적으로 설계된 스텍
        + '인터넷 기반의 효율적인 데이터 전송'이라는 커다란 하나의 문제를 하나의 덩치 큰 프로토콜 설계로 해결하지 않고 문제를 작게 나누어 계층화하려는 노력으로 탄생
        + 각각의 계층을 담당하는 것은 운영체제와 같은 소프트웨어이기도, NIC와 같은 물리적인 장치이기도 함.
        + 각 스텍 별 영역을 전문화하고 표준화
        + OSI 7 Layer - 7계층으로 세분화 되지만 4개층으로 구분짓기도 함.
        + 개방형 시스템 - 여러 개의 표준을 근거로 설계된 시스템, TCP/IP프로토콜 스택이 개방형 시스템. 표준이 존재한다 == 빠른 속도의 기술발전이 가능함   -> 시스템을 개방형으로 설계하는 이유

    - Link & IP 계층
        + Link 계층
            + 물리적인 영역의 표준화에 대한 결과
            + LAN,WAN,MAN과 같은 네트워크 표준과 관련된 프로토콜을 정의하는 영역

        + IP 계층
            + 목적지로 데이터를 전송하기 위해서 중간에 어떤 경로를 거쳐갈 것인지의 문제를 해결하는 계층
            + 이 계층에서 사용하는 프로토콜이 IP
                + IP 자체는 비 연결지향적이며 신뢰할 수 없는 프로토콜, 데이터를 전송할 때마다 거쳐야 할 경로를 선택해주지만, 그 경로가 일정하지 않음. 오류
                + 오류 발생에 대한 대비가 되어있지 않은 프로토콜

    - TCP/UDT 계층
        + 실제 데이터의 송수신과 관련 있는 계층
        + 전송(Transport) 계층이라고도 함
        + TCP는 데이터의 전송을 보장하는 프로토콜(신뢰성 있는 프로토콜), UDP는 보장하지 않는 프로토콜
        + TCP는 신뢰성을 보장하기 때문에 UDP에 비해 복잡한 프로토콜
            + 패킷이 전송되는 과정은 IP에 의해서 진행 -> 전송의 순서, 전송 자체를 신뢰할 수 x
            +  TCP 프로토콜이 추가되어 데이터를 송수신하면 데이터를 주고받는 관정에서 서로 데이터의 주고받음을 확인하고, 그리고 분실된 데이터에 대해서 재전송해준다. 따라서 데이터의 전송을 신뢰할 수 있게 된다.
    - APPLICATION 계층
        + 클라이언트와 서버 간의 데이터 송수신에 대한 약속
        + 응용프로그램의 프로토콜을 구성하는 계층
        + 소켓을 기반으로 완성하는 프로토콜을 의미함
        + 소켓을 생성하면, 앞서 보인 LINK, IP, TCP/UDP 계층에 대한 내용은 감춰진다.
        + 소켓을 기반으로 완성하는 프로토콜을 의미함
        + 응용 프로그래머는 APPLICATION 계층의 완성에 집중하게 된다. 

* TCP 기반 서버/클라이언트
    - TCP기반 서버 구현
        - TCP 서버의 기본적이 함수호출 순서
            + socket() -> bind() -> listen() -> accept() -> read()/write() <데이터 송수신> -> close() 
        - 연결요청 대기 상태로 진입
            + listen() 함수
            + 연결요청 대기상태에 있다는 것은 클라이언트가 연결요청을 했을 때 연결이 수락될 떄까지 연결 요청 자체를 대기시킬 수 있는 상태에 있다는 것
            + 연결요청 또한 데이터 전송 -> 연결요청을 받아들이기 위한 소켓 필요, 이 소켓이 서버소켓(리스닝 소켓)임. listen() 함수가 호출되면 서버소켓이 만들어짐.
            + *연결요청을 대기하는 큐(Queue)란? 우선 '연결요청을 대기'한다는 것은 클라이언트가 연결을 요청했을 때, 그 요청을 대기시킬 수 있음을 의미한다. 그리고 큐는 쉽게말해 대기실이라고 이해할 수 있는데, 시스템에서 순서대로(tcp인경우) 클라이언트를 연결시킬 수 있도록 큐에 모아놓는 것이다.*
        - 클라이언트의 연결 요청 수락
            + 클라이언트와 데이터를 주고받을 수 있는 상태가 됨.
            + 데이터를 주고받을 소켓 필요, accept() 함수가 호출 성공 시 내부적으로 데이터 입출력에 사용할 소켓을 생성하고 그 소켓의 파일 디스크립터를 반환

    - TCP기반 클라이언트 구현
        - TCP 클라이언트의 기본적인 함수호출 순서  
            + socket() -> connect() -> read()/write() -> close()
        - 클라이언트의 연결 요청
            +  서버는 listen 함수를 호출한 이후부터 연결요청 대기 큐를 만들어 놓음. 따라서 그 이후부터 클라이언트는 연결요청을 할 수 있음. 
            + connect() 함수가 호출되면 둘 중 하나의 상황이 되어야 반환됨. 
                1. 서버에 의해 연결요청이 접수 (연결요청 정보가 서버의 연결요청 대기 큐에 등록된 상황)
                    - connect 함수가 반환되더라도 당장 서비스가 안이루어질 수 있음. 
                2. 네트워크 단절 등 오류상황이 발생해서 연결요청이 중단 
            + 클라이언트 소켓의 주소정보는 bind를 통해 소켓에 직접 할당하지 않아도 connect함수 호출시 자동으로 할당됨.

    - 정리
        + 서버는 소켓 생성 이후에 bind, listen 함수의 연이은 호출을 통해 대기상태에 들어감
        +  클라이언트는 connect 함수호출을 통해서 연결 요청을 하게 됨.
        +  특히 클라이언트는 서버소켓의 listen 함수호출 이후에 connect 함수호출이 가능
        + 클라이언트가 connect 함수를 호출하기에 앞서 서버가 accept 함수를 먼저 호출할 수 있다. ->클라이언트가 connect 함수를 호출할 때까지 서버는 accept 함수가 호출된 위치에서 블로킹 상태.

* Iterative 기반의 서버, 클라이언트 구현
* 에코 클라이언트 구현
* TCP 이론
* UDP
* UDP 기반 서버/클라이언트 구현




* 서버- 클라이언트 실습
    - 공유기 관리자모드로 들어가기 (192.168.0.1) 접속 -> 포트포워드 설정에서  내부 ip주소(rpi ip주소), 외부포트 설정
    - client ip주소 210.119.12.--(고정 ip주소) 서버 ip 입력, 포트 번호


## 3일차 (2024-06-13)
* 157~

* reseadr_eserver.c 실행파일 - ch.4 echo_client.c의 실행파일 eclient와 같이 실행하여 결과 확인하기 
    - client에서 먼저 연결 종료를 요청하는 경우에는 문제 발생 X 
    - But Server에서 먼저 연결 종료시 같은 포트번호로 재실행시 bind() error가 발생함. 
     
- Clinet가 먼저 연결 종료를 요청한 경우에는 왜 오류가 안생길까 ??
 - server가 client를 찾아가는 경우에 당연하게 ip + port 번호가 필요.
 - client가 사용하는 포트 번호는 서버에서 랜덤으로 부여하기 때문에 문제가 발생하지 않음
 - 서버에서 연결을 끊었을 경우에는 서버의 포트가 time- way에 걸리기 때문에 약 3분간 연결이 되지 않는다.

 * Nagle 알고리즘
    - 앞에 전송한 ack 메세지를 받아야 다음 데이터를 전송하는 알고리즘
    - 단점 : 용량이 큰 파일 데이터의 전송의 경우에는 Nagle의 적용 여부에 따른 트래픽 차이가 크지않으면서도 적용하는 것보다 데이터 전송이 빠름.

* 멀티프로세스 기반의 서버 구현 (이전까지는 기본적인 설명이였고 이제 진짜 서버같은거 배우는거!)
    - 프로세스 : 메모리 공간을 차지한 상태에서 실행중인 프로그램


    - 프로세스 생성
        - fork() 
        ```C
        #include <unistd.h>
        pid_t fork(void);
        ```
        - 반환 : 성공 시 프로세스 ID, 실패 시 -1 
    
    - fork함수는 복사본을 생성 -> 새로운 프로세스가 아닌 이미 실행중인 fork함수를 호출한 프로세스를 복사 
        - 부모 프로세스 - fork 함수를 호출한 프로세스, fork함수의 반환 값은 자식 프로세스의 ID
        - 자식 프로세스 : fork 함수의 호출에 의해 생성된 프로세스, fork함수의 반환 값은 0 
    - fork 함수의 반환 이후에 프로세스가 각각 독립적으로 실행됨.

* 좀비 프로세스
    - 실행이 완료된 후에도 소멸되지 않은 프로세스
    - 프로세스도 main함수가 반환되면 소멸되어야 함.
    - 소멸되지 않았다 == 사용한 리소스가 메모리 공간에 존재
    - 생성 원인
        - 자식 프로세스가 종료되면서 반환하는 상태 값이 부모 프로세스에게 전달되지 않을 경우!
        1. 인자를 전달하면서 exit를 호출하는 경우
        2. main 함수에서 return문을 실행하면서 값을 반환하는 경우
        - -> 자식 프로세스의 종료 상태 값이 운영체젱 전달되는 경로
    
    - 좀비 프로세스 생성 확인
        - 현재 실행중인 프로세스 확인 - cmd에서 ps au 입력
        - 자식 프로세스의 종료 값을 반환 받을 부모 프로세스가 소멸되기 전에 좀비의 생성 확인해야함.
            - 부모 프로세스가 종료되면 좀비 상태였던 자식 프로세스도 함께 소멸하기 때문.
        ![좀비프로세스 생성 전](https://raw.githubusercontent.com/Juhyi/Bascic-Network-Programming-2024/main/imges/net003.png)

        ![좀비프로세스 생성 확인](https://raw.githubusercontent.com/Juhyi/Bascic-Network-Programming-2024/main/imges/net004.png)



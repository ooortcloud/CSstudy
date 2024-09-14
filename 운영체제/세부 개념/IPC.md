**프로세스 간 통신(IPC, Inter-Process Communication)**은 서로 다른 프로세스가 데이터를 주고받거나, 동기화할 수 있는 방법을 말합니다. 운영체제는 여러 프로세스가 동시에 실행될 수 있도록 지원하는데, 각 프로세스는 독립적인 메모리 공간을 가지고 있기 때문에, 직접적인 데이터 공유는 불가능합니다. 이 때문에 프로세스들이 상호작용하거나 데이터를 교환하기 위해서는 IPC 메커니즘이 필요합니다.

IPC는 주로 여러 프로세스가 협력 작업을 할 때나, 같은 시스템에서 실행되는 다른 프로세스들 간에 정보를 주고받을 때 사용됩니다. IPC는 일반적으로 **데이터 전송**, **프로세스 동기화**, **자원 공유**를 처리하는 데 사용됩니다.

### IPC의 주요 기법

1. **파이프 (Pipes)**
2. **메시지 큐 (Message Queues)**
3. **공유 메모리 (Shared Memory)**
4. **소켓 (Sockets)**
5. **세마포어 (Semaphores)**
6. **시그널 (Signals)**

이제 각각의 IPC 방식에 대해 자세히 살펴보겠습니다.

---

### 1. 파이프 (Pipes)

**파이프(Pipe)**는 한 프로세스에서 다른 프로세스로 데이터 스트림을 전송하는 반이중 통신 방식입니다. 하나의 시간에서, 한 프로세스는 데이터를 쓰고 다른 프로세스는 데이터를 읽습니다. 파이프는 두 프로세스 간의 통로로서 데이터를 주고받을 수 있는 통신 채널을 제공합니다. 통신 채널은 통신을 위한 메모리 공간(버퍼)으로 이루어져 있다. 파이프는 주로 부모 프로세스와 자식 프로세스 간의 통신에 사용됩니다.

#### 파이프 종류:
- **익명 파이프 (Anonymous Pipe)**: 같은 부모를 가진 프로세스들 사이에서만 사용할 수 있습니다.
- **이름 있는 파이프 (Named Pipe)**: 부모 자식 관계가 아닌 서로 관련이 없는 프로세스들 간에도 통신이 가능합니다. FIFO(First In, First Out) 방식으로 작동하며, 파일 시스템을 통해 생성됩니다. 이름이 있는 파일을 사용하기 때문에 가능함.

#### 파이프 사용 예제 (C 언어)

**익명 파이프를 이용한 부모-자식 프로세스 간의 통신**

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    int fd[2];
    pid_t pid;
    char buf[100];

    // 파이프 생성
    if (pipe(fd) == -1) {
        perror("pipe");
        return 1;
    }

    // 자식 프로세스 생성
    pid = fork();
    if (pid < 0) {
        perror("fork");
        return 1;
    }

    // 자식 프로세스
    if (pid == 0) {
        close(fd[1]); // 쓰기쪽 닫기
        read(fd[0], buf, 100);
        printf("Child process received: %s\n", buf);
        close(fd[0]);
    } else { // 부모 프로세스
        close(fd[0]); // 읽기쪽 닫기
        write(fd[1], "Hello from parent!", 17);
        close(fd[1]);
    }

    return 0;
}
```

**코드 설명:**

1. **파이프 생성:** `pipe(fd)` 함수를 이용하여 파이프를 생성하고, 파일 디스크립터 배열 `fd`에 읽기용(fd[0])과 쓰기용(fd[1]) 파일 디스크립터를 저장합니다.
2. **자식 프로세스 생성:** `fork()` 함수를 이용하여 자식 프로세스를 생성합니다.
3. **자식 프로세스:** 자식 프로세스에서는 쓰기용 파일 디스크립터를 닫고 읽기용 파일 디스크립터를 통해 부모 프로세스에서 보낸 메시지를 읽습니다.
4. **부모 프로세스:** 부모 프로세스에서는 읽기용 파일 디스크립터를 닫고 쓰기용 파일 디스크립터를 통해 자식 프로세스에게 메시지를 보냅니다.

**명명 파이프 사용 예제**

명명 파이프는 익명 파이프와 달리 파일 시스템에 이름을 가지고 있어서, 서로 관련 없는 프로세스들 간에도 통신이 가능합니다. 이는 부모-자식 관계가 아닌 프로세스들 간의 데이터 교환에 유용하게 활용될 수 있습니다.

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    int fd;
    char * myfifo = "/tmp/myfifo";

    // 파이프 생성 (만약 이미 존재하면 오류)
    mkfifo(myfifo, 0666);

    // 파이프 열기 (서버)
    fd = open(myfifo, O_RDONLY);
    char buf[100];
    read(fd, buf, 100);
    printf("Received: %s\n", buf);
    close(fd);

    // 파이프 삭제
    unlink(myfifo);

    return 0;
}
```

**코드 설명:**

1. **파이프 생성:** `mkfifo` 함수를 사용하여 `/tmp/myfifo`라는 이름의 파이프를 생성합니다.
2. **파이프 열기:** `open` 함수를 사용하여 읽기 모드로 파이프를 엽니다.
3. **데이터 읽기:** `read` 함수를 사용하여 파이프에서 데이터를 읽습니다.
4. **파이프 닫기:** `close` 함수를 사용하여 파이프를 닫습니다.
5. **파이프 삭제:** `unlink` 함수를 사용하여 파이프를 삭제합니다.

**클라이언트 코드 (별도의 프로세스)**

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    int fd;
    char * myfifo = "/tmp/myfifo";

    // 파이프 열기 (쓰기 모드)
    fd = open(myfifo, O_WRONLY);
    write(fd, "Hello from client!", 17);
    close(fd);

    return 0;
}
```

**클라이언트 코드 설명:**

1. **파이프 열기:** `open` 함수를 사용하여 쓰기 모드로 파이프를 엽니다.
2. **데이터 쓰기:** `write` 함수를 사용하여 파이프에 데이터를 씁니다.
3. **파이프 닫기:** `close` 함수를 사용하여 파이프를 닫습니다.

**주의 사항:**

* **파이프 생성 시 권한:** `mkfifo` 함수의 두 번째 인자는 파이프에 대한 권한을 설정합니다. 위 예제에서는 `0666`으로 설정하여 모든 사용자가 읽고 쓸 수 있도록 했습니다.
* **동기화:** 여러 프로세스가 동시에 파이프에 접근할 경우, 데이터 손실이나 오류가 발생할 수 있습니다. 이를 방지하기 위해 적절한 동기화 기법을 사용해야 합니다.
* **파이프 삭제:** 파이프 사용이 끝나면 `unlink` 함수를 사용하여 파이프를 삭제해야 합니다.

**명명 파이프의 활용 예시**

* **다중 프로세스 간 통신:** 서로 다른 프로세스 간에 데이터를 주고받을 때 사용할 수 있습니다.
* **데이터 스트리밍:** 큰 데이터를 실시간으로 전송할 때 사용할 수 있습니다.
* **데이터 필터링:** 파이프를 통해 데이터를 전달하면서 필터링 작업을 수행할 수 있습니다.

**익명 파이프와의 차이점**

* **생성:** 익명 파이프는 `pipe` 함수를 사용하여 메모리 내에 생성되지만, 명명 파이프는 `mkfifo` 함수를 사용하여 파일 시스템에 생성됩니다.
* **수명:** 익명 파이프는 프로세스가 종료되면 자동으로 사라지지만, 명명 파이프는 `unlink` 함수로 명시적으로 삭제해야 합니다.
* **접근:** 익명 파이프는 부모-자식 프로세스 간에만 사용할 수 있지만, 명명 파이프는 서로 관련 없는 프로세스들 간에도 사용할 수 있습니다.


#### 파이프의 장단점

* **장점:**
  * 구현이 간단하고 사용하기 쉽다.
  * 부모-자식 프로세스 간의 통신에 효과적이다.
* **단점:**
  * 단방향 통신만 지원하거나 양방향 통신을 위해 두 개의 파이프를 사용해야 한다.
  * 데이터 전달 시 버퍼링 문제가 발생할 수 있다.
  * 프로세스 간의 관계가 복잡해질 경우 관리가 어려울 수 있다.

#### 파이프의 활용 예시

* **쉘 파이프라인:** Unix/Linux 쉘에서 파이프 기호 `|`를 사용하여 여러 명령어의 표준 출력과 표준 입력을 연결하는 기능
* **프로세스 간 데이터 전달:** 부모 프로세스에서 생성된 데이터를 자식 프로세스로 전달하거나, 자식 프로세스에서 처리한 결과를 부모 프로세스로 전달
* **필터 프로그램:** 파이프를 이용하여 데이터를 필터링하는 프로그램을 구현

```bash
$ ls | grep .txt
```
위 명령어에서 `ls` 명령어의 출력이 파이프를 통해 `grep` 명령어로 전달됩니다.



#### 추가 고려 사항

* **데이터 형식:** 파이프를 통해 전달되는 데이터는 일반적으로 바이트 스트림 형태입니다.
* **동기화:** 여러 프로세스가 동시에 파이프에 접근할 때 발생할 수 있는 문제를 해결하기 위해 적절한 동기화 기법을 사용해야 합니다.
* **버퍼링:** 파이프에는 버퍼가 존재하므로, 데이터를 읽고 쓸 때 버퍼의 상태를 고려해야 합니다.


---

### 2. 메시지 큐 (Message Queues)

**메시지 큐(Message Queue)**는 운영체제가 제공하는 큐(Queue)를 통해 프로세스 간에 데이터를 주고받는 방식입니다. 이는 우체통과 비슷하게 생각할 수 있는데, 한 프로세스가 메시지를 우체통에 넣으면 다른 프로세스가 그 메시지를 꺼내 읽는 방식입니다. 메시지 큐는 FIFO(First In, First Out) 방식으로 데이터를 처리하며, 서로 독립적인 프로세스가 데이터를 주고받는 데 사용됩니다.

#### 주요 특징:
- **비동기 통신**: 메시지를 보내는 프로세스와 받는 프로세스가 서로 독립적으로 동작할 수 있습니다. 메시지를 보내고 나서, 받는 프로세스가 즉시 데이터를 처리할 필요가 없습니다.
- **구조화된 데이터 전송**: 메시지는 구조화된 데이터(예: 문자열 또는 숫자)로 구성됩니다.
- **FIFO 방식**: 메시지는 큐에 순서대로 쌓이며, 먼저 들어온 메시지가 먼저 처리됩니다.
- **다중 프로세스 간 통신:** 여러 개의 프로세스가 하나의 메시지 큐를 공유하여 통신할 수 있습니다.
- **시스템 호출:** 메시지 큐는 운영체제의 시스템 호출을 이용하여 구현되므로, 상대적으로 오버헤드가 크고 복잡한 편입니다.
- **메시지 타입:** 각 메시지는 고유한 타입을 가질 수 있어서, 수신자가 원하는 타입의 메시지만 선택적으로 받을 수 있습니다.

#### 메시지 큐 사용 절차:
1. 프로세스 A가 메시지 큐에 데이터를 추가.
2. 프로세스 B가 메시지 큐에서 데이터를 읽어옴.


#### C 언어를 이용한 메시지 큐 구현 예제 (POSIX)

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

#define MAX_TEXT 512

// 메시지 구조체
struct msgbuf {
    long mtype;       // 메시지 타입
    char mtext[MAX_TEXT]; // 메시지 내용
};

int main() {
    int msgid;
    struct msgbuf buf;

    // 메시지 큐 생성 (키 값을 이용하여 생성)
    key_t key = ftok(".", 'B');
    msgid = msgget(key, 0666 | IPC_CREAT);
    if (msgid == -1) {
        perror("msgget");
        exit(1);
    }

    // 메시지 보내기
    buf.mtype = 1;
    strcpy(buf.mtext, "Hello, world!");
    msgsnd(msgid, &buf, sizeof(buf.mtext), 0);

    // 메시지 받기
    msgrcv(msgid, &buf, MAX_TEXT, 1, 0);
    printf("Received: %s\n", buf.mtext);

    // 메시지 큐 삭제
    msgctl(msgid, IPC_RMID, NULL);

    return 0;
}
```

**코드 설명**
* **msgget:** 메시지 큐를 생성하거나 얻는 함수입니다. `key` 값은 메시지 큐를 식별하는 데 사용됩니다.
* **msgsnd:** 메시지를 메시지 큐에 보내는 함수입니다.
* **msgrcv:** 메시지 큐에서 메시지를 받는 함수입니다.
* **msgctl:** 메시지 큐를 제어하는 함수로, 여기서는 메시지 큐를 삭제하는 데 사용됩니다.

**메시지 큐의 주요 함수**
* **msgget:** 메시지 큐 생성 또는 접속
* **msgsnd:** 메시지 큐에 메시지 보내기
* **msgrcv:** 메시지 큐에서 메시지 받기
* **msgctl:** 메시지 큐 제어 (생성, 삭제, 속성 변경 등)


#### 메시지 큐의 활용 예시
* **멀티 프로세스 시스템:** 다양한 프로세스들이 메시지를 주고받으며 협력하는 시스템
* **분산 시스템:** 서로 다른 컴퓨터에 있는 프로세스 간의 통신
* **데이터 공유:** 여러 프로세스가 공유하는 데이터를 메시지 큐를 통해 교환

#### 주의 사항
* **메시지 큐 생성 및 삭제:** 메시지 큐는 사용 후 반드시 삭제해야 합니다.
* **메시지 크기:** 메시지 크기는 시스템에 따라 제한될 수 있습니다.
* **오류 처리:** 메시지 큐 관련 함수 호출 시 오류가 발생할 수 있으므로, 적절한 오류 처리를 해야 합니다.


---

### 3. 공유 메모리 (Shared Memory)

**공유 메모리(Shared Memory)**는 두 개 이상의 프로세스가 동일한 메모리 공간을 공유하여 데이터를 주고받는 방식입니다. IPC 기법 중에서 가장 빠른 데이터 전송 방식을 제공하지만, 동기화 문제를 해결해야 하는 부담이 있습니다. 공유 메모리는 빠른 데이터 전달이 필요하고, 여러 프로세스가 동일한 데이터를 공유해야 하는 경우에 유용하게 사용될 수 있습니다.

#### 주요 특징:
- **빠른 통신**: 다른 IPC 방식보다 빠릅니다. 프로세스들이 메모리 공간을 공유하기 때문에, 파일 시스템이나 커널을 경유하지 않아도 됩니다.
- **동기화 필요**: 프로세스 간에 동시에 메모리에 접근하는 문제를 해결하기 위해 **세마포어** 또는 **뮤텍스** 같은 동기화 기법이 필요합니다.
- **동일한 메모리 접근**: 프로세스들이 메모리를 공유하기 때문에 프로세스 간 데이터를 전달하는 데 매우 효율적입니다.

#### 예시:
프로세스 A와 프로세스 B가 같은 메모리 공간을 사용하여 데이터를 읽고 쓰는 경우, 프로세스 A가 데이터를 쓰면, 프로세스 B는 이를 곧바로 읽을 수 있습니다.

#### C 언어를 이용한 공유 메모리 예제

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define SHMSIZE 27

int main() {
    int shmid;
    key_t key;
    char *shm, *s;

    // 공유 메모리 키 생성
    key = ftok(".", 'B');

    // 공유 메모리 생성 (또는 연결)
    shmid = shmget(key, SHMSIZE, IPC_CREAT | 0666);
    if (shmid < 0) {
        perror("shmget");
        exit(1);
    }

    // 공유 메모리에 연결
    shm = shmat(shmid, NULL, 0);
    if (shm == (char *) -1) {
        perror("shmat");
        exit(1);
    }

    // 공유 메모리에 데이터 쓰기 (프로세스 1)
    strcpy(shm, "Hello, world!");

    // 공유 메모리에서 데이터 읽기 (프로세스 2)
    s = shm;
    printf("%s\n", s);

    // 공유 메모리 해제
    shmdt(shm);

    // 공유 메모리 삭제
    shmctl(shmid, IPC_RMID, NULL);

    return 0;
}
```

**코드 설명**
* **shmget:** 공유 메모리 세그먼트를 생성하거나 이미 존재하는 세그먼트를 얻습니다.
* **shmat:** 프로세스의 주소 공간에 공유 메모리 세그먼트를 연결합니다.
* **shmdt:** 프로세스의 주소 공간에서 공유 메모리 세그먼트를 분리합니다.
* **shmctl:** 공유 메모리 세그먼트를 제어하는 함수입니다.

**공유 메모리 사용 시 주의사항**
* **동기화:** 위 예제에서는 간단하게 구현했지만, 실제 환경에서는 여러 프로세스가 동시에 공유 메모리에 접근하기 때문에 세마포어 등을 이용하여 동기화를 해야 합니다.
* **오류 처리:** 각 함수 호출 시 오류 발생 여부를 반드시 확인해야 합니다.
* **메모리 누수:** shmdt 함수를 호출하여 공유 메모리를 해제하지 않으면 메모리 누수가 발생할 수 있습니다.

**추가 고려 사항**
* **공유 메모리 키:** 공유 메모리를 식별하기 위한 고유한 값입니다.
* **공유 메모리 크기:** shmget 함수의 두 번째 인자를 통해 공유 메모리의 크기를 지정합니다.
* **공유 메모리 권한:** shmget 함수의 세 번째 인자를 통해 공유 메모리에 대한 접근 권한을 설정합니다.

#### 공유 메모리의 장단점
* **장점:**
    * 메시지 큐보다 빠른 데이터 전달 속도
    * 대량의 데이터를 효율적으로 전달 가능
* **단점:**
    * 동기화 문제 발생 가능성
    * 메모리 관리에 대한 부담
    * 직접 메모리를 조작하기 때문에 오류 발생 가능성 증가



---

### 4. 소켓 (Sockets)

**소켓(Socket)**은 네트워크를 통해 프로세스 간에 통신할 때 사용하는 기법입니다. 소켓은 프로세스가 네트워크 상의 다른 프로세스와 데이터를 주고받을 수 있는 양방향 통신 방식입니다. 인터넷을 기반으로 한 **TCP/IP** 소켓을 주로 사용합니다.

#### 주요 특징:
- **양방향 통신**: 데이터를 주고받을 수 있습니다. TCP나 UDP를 기반으로 데이터를 송수신합니다.
- **네트워크 기반 통신**: 같은 컴퓨터 내의 프로세스뿐만 아니라, 네트워크 상의 원격 프로세스와도 통신할 수 있습니다.
- **표준화된 프로토콜**: TCP(연결 지향적)와 UDP(비연결 지향적) 프로토콜을 사용합니다.

#### 예시:
클라이언트-서버 모델에서, 클라이언트가 소켓을 통해 서버에 요청을 보내고, 서버가 응답을 처리하여 클라이언트로 다시 전송하는 방식입니다.

IPC(Inter-Process Communication) 소켓 프로그래밍의 예제를 Unix 도메인 소켓을 사용하여 C 언어로 구현한 예시를 제공해 드리겠습니다. 이 예제는 서버와 클라이언트 프로그램으로 구성되어 있습니다.

먼저 서버 프로그램입니다:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/un.h>
#include <unistd.h>

#define SOCKET_NAME "/tmp/example_socket"
#define BUFFER_SIZE 128

int main() {
    struct sockaddr_un name;
    int down_flag = 0;
    int ret;
    int connection_socket;
    int data_socket;
    int result;
    char buffer[BUFFER_SIZE];

    /* 소켓 생성 */
    connection_socket = socket(AF_UNIX, SOCK_STREAM, 0);
    if (connection_socket == -1) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    /* 기존 소켓 파일 삭제 */
    unlink(SOCKET_NAME);

    /* 서버 주소 설정 */
    memset(&name, 0, sizeof(struct sockaddr_un));
    name.sun_family = AF_UNIX;
    strncpy(name.sun_path, SOCKET_NAME, sizeof(name.sun_path) - 1);

    /* 소켓에 주소 바인드 */
    ret = bind(connection_socket, (const struct sockaddr *) &name,
               sizeof(struct sockaddr_un));
    if (ret == -1) {
        perror("bind");
        exit(EXIT_FAILURE);
    }

    /* 연결 대기 */
    ret = listen(connection_socket, 20);
    if (ret == -1) {
        perror("listen");
        exit(EXIT_FAILURE);
    }

    /* 메인 루프 */
    while (1) {
        /* 클라이언트 연결 수락 */
        data_socket = accept(connection_socket, NULL, NULL);
        if (data_socket == -1) {
            perror("accept");
            exit(EXIT_FAILURE);
        }

        /* 클라이언트로부터 데이터 수신 및 응답 */
        while (1) {
            ret = read(data_socket, buffer, BUFFER_SIZE);
            if (ret == -1) {
                perror("read");
                exit(EXIT_FAILURE);
            }

            /* 종료 메시지 확인 */
            if (!strncmp(buffer, "DOWN", BUFFER_SIZE)) {
                down_flag = 1;
                break;
            }

            /* 에코 응답 */
            sprintf(buffer, "Server received: %s", buffer);
            ret = write(data_socket, buffer, BUFFER_SIZE);
            if (ret == -1) {
                perror("write");
                exit(EXIT_FAILURE);
            }
        }

        /* 소켓 닫기 */
        close(data_socket);

        if (down_flag) {
            break;
        }
    }

    close(connection_socket);
    unlink(SOCKET_NAME);
    exit(EXIT_SUCCESS);
}
```

다음은 클라이언트 프로그램입니다:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/un.h>
#include <unistd.h>

#define SOCKET_NAME "/tmp/example_socket"
#define BUFFER_SIZE 128

int main() {
    struct sockaddr_un addr;
    int ret;
    int data_socket;
    char buffer[BUFFER_SIZE];

    /* 소켓 생성 */
    data_socket = socket(AF_UNIX, SOCK_STREAM, 0);
    if (data_socket == -1) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    /* 서버 주소 설정 */
    memset(&addr, 0, sizeof(struct sockaddr_un));
    addr.sun_family = AF_UNIX;
    strncpy(addr.sun_path, SOCKET_NAME, sizeof(addr.sun_path) - 1);

    /* 서버에 연결 */
    ret = connect(data_socket, (const struct sockaddr *) &addr,
                  sizeof(struct sockaddr_un));
    if (ret == -1) {
        fprintf(stderr, "The server is down.\n");
        exit(EXIT_FAILURE);
    }

    /* 메인 루프 */
    while (1) {
        printf("Enter message to server: ");
        fgets(buffer, BUFFER_SIZE, stdin);
        buffer[strcspn(buffer, "\n")] = 0;  // 개행 문자 제거

        /* 서버에 메시지 전송 */
        ret = write(data_socket, buffer, BUFFER_SIZE);
        if (ret == -1) {
            perror("write");
            break;
        }

        /* 종료 메시지 확인 */
        if (!strcmp(buffer, "DOWN")) {
            break;
        }

        /* 서버로부터 응답 수신 */
        ret = read(data_socket, buffer, BUFFER_SIZE);
        if (ret == -1) {
            perror("read");
            break;
        }

        printf("Server response: %s\n", buffer);
    }

    /* 소켓 닫기 */
    close(data_socket);
    exit(EXIT_SUCCESS);
}
```

이 예제에서:

1. 서버는 Unix 도메인 소켓을 생성하고, 지정된 파일 경로에 바인드합니다.
2. 서버는 클라이언트의 연결을 기다립니다.
3. 클라이언트가 연결하면, 서버는 클라이언트로부터 메시지를 받아 에코 응답을 보냅니다.
4. 클라이언트는 사용자 입력을 받아 서버에 전송하고, 서버의 응답을 출력합니다.
5. "DOWN" 메시지를 보내면 서버와 클라이언트 모두 종료됩니다.

이 프로그램들을 컴파일하고 실행하면, 로컬 시스템에서 프로세스 간 통신을 테스트할 수 있습니다. 이 예제는 기본적인 IPC 소켓 통신을 보여주며, 실제 사용 시에는 오류 처리, 보안, 동시성 등을 고려해야 합니다.

---

### 5. 세마포어 (Semaphores)

**세마포어(Semaphore)**는 여러 프로세스가 공유 자원에 접근할 때 발생하는 **경쟁 상태(race condition)**를 방지하기 위한 동기화 기법입니다. 세마포어는 주로 공유 자원에 대한 **진입 허가**(lock)를 관리합니다.

#### 주요 특징:
- **동기화 기법**: 프로세스들이 공유 자원에 동시에 접근하지 않도록 관리합니다.
- **카운팅 세마포어**: 여러 프로세스가 자원을 사용할 수 있는 수를 관리합니다.
- **이진 세마포어**: 하나의 프로세스만 자원에 접근할 수 있도록 제어합니다(뮤텍스와 유사).

#### 예시:
A 프로세스가 자원을 사용하고 있을 때, B 프로세스는 세마포어를 통해 자원이 해제될 때까지 대기하다가, A 프로세스가 작업을 마치면 자원에 접근합니다.

---

### 6. 시그널 (Signals)

**시그널(Signal)**은 프로세스에 특정 이벤트가 발생했음을 알리기 위한 기법입니다. 운영체제는 시그널을 통해 프로세스에게 이벤트를 전달하고, 프로세스는 이를 처리하는 방식으로 동작합니다.

#### 주요 특징:
- **비동기 이벤트 처리**: 시그널은 특정 이벤트가 발생했을 때 프로세스에 알립니다. 예를 들어, 키보드 입력이나 타이머 만료 등.
- **단일 비동기 이벤트 전송**: 시그널은 주로 간단한 이벤트 알림에 사용됩니다.

#### 예시:
`SIGINT`는 사용자가 `Ctrl+C`를 눌렀을 때 프로세스에 전달되는 시그널입니다. 이 시그널을 받으면 프로세스는 실행을 종료합니다.

---

### IPC의 문제점 및 해결 방법

IPC를 사용할 때 발생할 수 있는 주요 문제는 **동기화**와 **경쟁 상태**입니다. 여러 프로세스가 동시에 자원에 접근하면 데이터 일관성이 깨질 수 있습니다. 이를 해결하기 위해 동기화 기법이 필요합니다.

#### 1. **동기화 문제**
- 공유 자원에 동시에 접근할 때 데이터가 충돌하거나, 잘못된 상태에 빠질 수 있습니다. 이를 해결하기 위해 **세마포어** 또는 **뮤텍스**와 같은 동기화 메커니즘을 사용하여, 한 번에 하나의 프로세스만 자원에 접근하도록 제어합니다.

#### 2. **경쟁 상태**
- 여러 프로세스가 자원에 동시에 접근하려 할 때 발생하는 문제입니다. 이 문제를 해결하기 위해 **락(lock)** 또는 **세마포어**를 사용하여 자원 접근을 제어할 수 있습니다.

---

### IPC의 활용 사례

IPC는 여러 응용 프로그램에서 다양한 형태로 사용됩니다. 예를 들어, 웹 브라우저와 웹 서버 간의 통신은 **소켓**을 통해 이루어집니다. 또, 운영체제에서 실행 중인 여러 프로세스는 **파이프**나 **메시지 큐**를 사용해 데이터를 주고받으며 협력 작업을 수행합니다. **공유 메모리**는 실시간 시스템이나 빠른 데이터 전송이 필요한 애플리케이션에서 많이 사용됩니다.

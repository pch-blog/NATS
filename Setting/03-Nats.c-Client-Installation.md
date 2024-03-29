# 03-Nats.c-Client-Installation
- C언어로 작성된 NATS 프로젝트의 클라이언트 라이브러리인 nats.c의 설치 과정 정리.

- nats.c 라이브러리 설치 후 프로그램 개발 과정 정리.

<br>

## Table of Contents
- [nats.c 다운로드 및 설치](#natsc-다운로드-및-설치)

- [nats.c 설치 오류](#natsc-설치-오류)

- [nats.c 라이브러리로 프로그램 개발](#natsc-라이브러리로-프로그램-개발)

<br>

## nats.c 다운로드 및 설치
### TLS OFF, Streaming OFF 상태 설치
- 해당 설치 과정은 CentOS 7에서 진행하였고, 두 기능을 OFF 후 컴파일하면 다른 문제 없이 컴파일 및 설치 가능

  ```shell
  $ git clone https://github.com/nats-io/nats.c.git
  $ cd nats.c
  $ mkdir build; cd build
  $ cmake .. -DNATS_BUILD_WITH_TLS=OFF -DNATS_BUILD_STREAMING=OFF
  $ sudo make install
  ```

- 각 기능을 OFF한 이유는 다음과 같음
  - 두 기능은 기본적으로 ON
  - <span style="color:red"><b>TLS : CentOS 7 기준 openssl-devel 패키지 버전이 낮아서 컴파일 에러</b></span>
  - <span style="color:red"><b>Streaming : CentOS 7 기준 protobuf-c 버전이 낮고 Nats.c가 요구하는 protobuf-c 최소 요구 버전 및 최신 버전 컴파일에 문제가 있음</b></span>

### TLS ON, Streaming ON  상태 설치
- 해당 설치 과정은 RHEL 8.9와 Rocky 9.2에서 진행한 과정

- 필요한 패키지 설치 및 업데이트
  ```shell
  $ sudo yum -y install protobuf protobuf-c protobuf-c-devel
  # OR
  $ sudo yum -y update protobuf protobuf-c protobuf-c-devel
  ```
- protobuf-c 관련 define은 변경될 수 있으며, 각각 위치에 대해 파악이 필요하다.
- <b>Rocky 9.2 기준 "protobuf-c-devel" 패키지는 안나오기 때문에 [오류2](#오류2-protobuf-c-헤더를-못-찾는-경우)의 방법으로 해결한다.</b>

  ```shell
  $ git clone https://github.com/nats-io/nats.c.git
  $ cd nats.c
  $ mkdir build; cd build
  $ cmake .. -DNATS_BUILD_WITH_TLS=ON -DNATS_BUILD_STREAMING=ON \
  -DNATS_PROTOBUF_DIR=/usr/lib64 \
  -DNATS_PROTOBUF_LIBRARY=/usr/lib64/libprotobuf-c.so \
  -DNATS_PROTOBUF_INCLUDE_DIR=/usr/include/protobuf-c
  $ sudo make install
  ```

### CMake 옵션
- 설치 과정 중 사용한 CMake 옵션
  ```shell
  # TLS 사용여부 (default - ON)
  -DNATS_BUILD_WITH_TLS=OFF | ON
  # Streaming 사용여부
  -DNATS_BUILD_STREAMING=OFF | ON
  # 기본 라이브러리 경로에서 찾지 못하면 libprotobuf-c 라이브러리 디렉토리 경로 지정
  -DNATS_PROTOBUF_DIR=
  # 정적 라이브러리를 찾지 못하면 libprotobuf-c .so 라이브러리 경로 지정
  -DNATS_PROTOBUF_LIBRARY=
  # protobuf-c의 헤더파일 디렉토리 경로 추가
  -DNATS_PROTOBUF_INCLUDE_DIR=
  ```

<br>

## nats.c 설치 오류
### 오류1. CMake(Cross-platform make system) 버전이 낮은 경우
- CentOS 7에서 설치할 때 나온 오류로 수동으로 CMake 버전 업데이트
  ```shell
  $ wget https://github.com/Kitware/CMake/releases/download/v3.26.5/cmake-3.26.5.tar.gz
  $ tar -zxvf cmake-3.26.5.tar.gz
  $ cd cmake-3.26.5
  $ ./bootstrap
  $ gmake
  $ sudo make install
  ```

- OS가 CentOS 8 또는 RHEL 8.9 일 경우 yum update 시도 후 수동으로 버전 업데이트
  ```shell
  $ sudo yum update cmake
  ```
### - 오류2. protobuf-c 헤더를 못 찾는 경우
- Rocky 9.2에서 yum search를 하면 protobuf-c-devel 패키지를 찾을 수 없음.

- protobuf-c 코드를 github에서 clone
  ```shell
  $ git clone https://github.com/protobuf-c/protobuf-c.git
  ```

- 해당 경로 지정
  ```shell
  $ cmake ..  -D....... -DNATS_PROTOBUF_INCLUDE_DIR="github에서 protobuf-c를 clone한 경로"
  ```

<br>

## nats.c 라이브러리로 프로그램 개발
- [Nats.c Documentation](http://nats-io.github.io/nats.c/topics.html)
### Makefile 라이브러리 추가
- Makefile 수정

  ```make
  # 동적 라이브러리
  NATSLIBS = -lnats
  # 정적 라이브러리
  NATSLIBS = -lnats_static
  # Streaming ON
  NATSLIBS = -lnats_static -lprotobuf-c
  # TLS ON
  NATSLIBS = -lnats_static -lssl -lcrypto
  # Streaming ON & TLS ON
  NATSLIBS = -lnats_static -lprotobuf-c -lssl -lcrypto
  ```

### 클라이언트 개발 기타 설명
- 전체적인 함수 목록이나 설명은 [여기](http://nats-io.github.io/nats.c/topics.html)를 참조한다.

- 전반적으로 스레드가 처리하는 것으로 보이며 "ps -efL | grep 프로세스명" 명령으로 확인하면 총 6개(메인1, 스레드5개) 확인.

- nats-server를 다운 시킨 후 확인하면 스레드 4개로 확인.

- nats-server가 다운된 경우 클라이언트에서는 재연결 시도하며 이 부분도 스레드인 것으로 추측.

- 재연결 시도하는 동안 발생한 데이터는 버려지나 메세지 발행 함수는 정상을 리턴함.

- 설정된 연결 시도 횟수가 끝나면 메세지 발행 함수에서 에러 리턴.

- natsOptions_SetAllowReconnect 함수를통해 재연결허용을 해제하면 메세지 발행 함수가 바로 에러 리턴.

- 재연결 기능을 옵션 설정을 통해 끄고 별도의 재연결 시도 로직을 만들어서 처리

### pthread 관련 처리 (오픈 소스 수정 영역)
- NATS C클라이언트 코드 내에서 pthread_create, 등 phtread 함수 확인
- 메인 스레드만 특정 시그널을 받기 위해 다른 스레드에서 특정 시그널 블럭 시킬 방법 필요

### 연결 함수 차이
- natsConnection_ConnectTo : 기본 옵션 사용

- natsConnection_Connect : 옵션 객체가 NULL이면 natsConnection_ConnectTo 호출 후 옵션 clone

### - 기본 옵션 DEFINE
- nats.c/src/opts.h

  ```c
  #define NATS_OPTS_DEFAULT_MAX_RECONNECT         (60)
  #define NATS_OPTS_DEFAULT_TIMEOUT               (2 * 1000)          // 2 seconds
  #define NATS_OPTS_DEFAULT_RECONNECT_WAIT        (2 * 1000)          // 2 seconds
  #define NATS_OPTS_DEFAULT_PING_INTERVAL         (2 * 60 * 1000)     // 2 minutes
  #define NATS_OPTS_DEFAULT_MAX_PING_OUT          (2)
  #define NATS_OPTS_DEFAULT_IO_BUF_SIZE           (32 * 1024)         // 32 KB
  #define NATS_OPTS_DEFAULT_MAX_PENDING_MSGS      (65536)
  #define NATS_OPTS_DEFAULT_RECONNECT_BUF_SIZE    (8 * 1024 * 1024)   // 8 MB
  #define NATS_OPTS_DEFAULT_RECONNECT_JITTER      (100)               // 100 ms
  #define NATS_OPTS_DEFAULT_RECONNECT_JITTER_TLS  (1000)              // 1 second
  ```

### - 연결 상태 확인 함수
- natsConnStatus natsConnection_Status(natsConnection *nc)

- nats.c/src/status.h

  ```c
  typedef enum
  {
      NATS_CONN_STATUS_DISCONNECTED = 0, ///< The connection has been disconnected
      NATS_CONN_STATUS_CONNECTING,       ///< The connection is in the process or connecting
      NATS_CONN_STATUS_CONNECTED,        ///< The connection is connected
      NATS_CONN_STATUS_CLOSED,           ///< The connection is closed
      NATS_CONN_STATUS_RECONNECTING,     ///< The connection is in the process or reconnecting
      NATS_CONN_STATUS_DRAINING_SUBS,    ///< The connection is draining subscriptions
      NATS_CONN_STATUS_DRAINING_PUBS,    ///< The connection is draining publishers
  } natsConnStatus;
  ```
<br>
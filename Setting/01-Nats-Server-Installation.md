# 01-Nats-Server-Installation
- NATS 프로젝트의 nats-server 설치 과정 정리
<br>

## Table of Contents
- [nats-server 서버 설치](#nats-server-서버-설치)
<br><br>

## nats-server 서버 설치
- NATS.io의 [다운로드페이지](https://github.com/nats-io/nats-server/releases)에서 상황에 맞는 버전을 결정한다.
- 이 문서에서는 사용할 버전의 RPM 다운로드 링크를 복사하여 wget으로 다운로드한다.
```shell
# RPM 다운로드 및 설치
$ wget https://github.com/nats-io/nats-server/releases/download/v2.10.7/nats-server-v2.10.7-amd64.rpm
$ rpm -i nats-server-v2.10.7-amd64.rpm
```
```shell
# 위치 확인
$ which nats-server
/usr/local/bin/nats-server
```
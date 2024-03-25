# 02-Nats-Server-Config
- [01-nats-server-installation](01-nats-server-installation.md)에서 설치한 nats-server 프로그램을 실행하기 위해 필요한 환경설정 과정 정리

## Table of Contents
- [nats-server 서버 환경설정](#nats-server-서버-환경설정)
- [nats-server 클러스터링 설정](#nats-server-클러스터링-설정)
<br><br>

## nats-server 서버 환경설정
- [Configuring NATS Server](https://docs.nats.io/running-a-nats-service/configuration)
- 필요한 설정값을 추가하여 환경설정 파일을 작성한다.
### 환경설정 파일 작성
- PORT 설정
```shell
# nats-server의 기본 포트
port: 4222
# nats-server의 모니터링 포트
http_port: 8222
```
- LOG 설정
```shell
# 로그파일 위치
log_file: "로그 경로 및 파일명"
# 로그 시간 표시 여부
logtime: true | false
# 로그 파일 사이즈 설정
logfile_size_limit: 1G
```
- DEBUG 설정
```shell
# DEBUG 수준의 로그 설정
debug: true | false
```
### 환경설정 파일 예시
- "nats.cfg" 작성
```shell
# PORT
port: 4222
http_port: 8222

# DEBUG
debug: true

# LOG
log_file: /tmp/nats-server.log
logtime: true
logfile_size_limit: 1G
```
### 환경설정 파일 테스트
```shell
$ nats-server -t "환경설정 파일"
```
<br>

## nats-server 클러스터링 설정
- [NATS Server Clustering](https://docs.nats-server/running-a-nats-service/configuration/clustering)
- seed(main) 환경설정 파일에 내용 추가
```shell
cluster {
  port:클러스터의 포트
}
```
- cluster 환경설정 파일에 내용 추가
```shell
cluster {
  port:클러스터의 포트
  routes = [
    nats://시드의 IP 주소:시드의 클러스터 포트
  ]
}
```
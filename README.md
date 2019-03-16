# nginx
동시 접속 처리에 특화된 웹 서버 프로그램
* Apache보다 단순하며, 전달자 역할만을 수행하므로 동시 접속 처리에 특화되어 있음
* 동시접속자가 700명 이상일 경우 서버를 증설하거나 nginx 환경을 권장
* AWS상에서 시장 점유율 44%에 달할 정도로 가볍고 성능이 좋은 엔진으로 인정받음
* 비동기 처리 방식(Event-Driven)을 사용

## nginx 웹 서버의 역할
### 정적 파일을 처리하는 HTTP 서버로서의 역할
기본적인 웹 서버의 역할로, HTML, CSS, JavaScript, 이미지와 같은 정보를 브라우저로 전송

### 응용프로그램 서버에 요청을 보내는 리버스 프록시로서의 역할
클라이언트가 가짜 서버에게 요청을 전송하면, 프록시 서버가 배후 서버(reverse server)로부터 데이터를 가져옴
* **사용 이유** : 요청에 대한 버퍼링 때문에 요청을 배분하고자 사용
* **방법** : nginx.conf 파일에서 `location` 지시어를 사용해서 요청 배분

클라이언트가 직접 App 서버에 요청할 경우 App 서버의 프로세스 1개가 응답 대기 상태가 되어야 하기 때문에
  
## nginx 설치(macOS)

    brew install nginx
  
## nginx 설정하기
* `nginx.conf` : nginx 설정파일

```
worker_processes  1;
events {
    worker_connections  1024;
}
http { 
    include       mime.types;
    server {
        listen       80;
        location / {
            root   html;
            index  index.html index.htm;
        }
    }
}
```

### Core 모듈
설정 파일의 최상단에 위치하면서 nginx의 기본적인 동작 방식을 정의
* 여기에서 사용되는 지시어들은 다른 곳에서 사용되지 않음
* 위 예제의 `worker_process`와 같은 것
* 참고링크 - [코어모듈 지시어 사전](http://opentutorials.org/module/384/4533)

#### 지시어 목록
* `user` : worker process와 cache manager process를 구동하는 부분
  * nginx를 구동하면 master process, worker process, cache manager process 3개를 실행함
  * master process는 root 유저로 기동함
* `worker_process` : 몇 개의 프로세스를 실행할지 설정
  * nginx는 싱글스레드이므로, core 수를 맞춰서 설정함
  * core 수보다 높은 숫자를 지정해도 문제는 없고, auto로 하면 알아서 지정해줌
* `error_log` : 에러로그의 위치를 지정해주는 지시어, http나 server 블록에서도 지정 가능
* `pid` : pid의 저장 위치 설정

### http 블록
server, location 블록이 상속할 루트 블록
* 여러 개를 사용할 수 있지만, 관리상의 이슈로 한 번만 사용하는 것을 권장
* http 블록의 내용은 server의 기본 값이, server의 내용은 location의 기본 값이 됨(계층구조)

#### 지시어 목록
* `server_tokens` : response-header에 nginx 버전을 쓸지 결정
  * 보안을 위해 일반적으로 off로 설정함
* `include` : 
* `default_type` :
* `sendfile` :
* `access_log` :

### server 블록
하나의 웹 사이트를 선언하는데 사용(가상 호스팅의 개념)
* 하나의 서버로 여러 주소를 동시에 운영하고 싶을 때 사용
* 참고링크 - [가상 호스팅](http://opentutorials.org/module/384/4529)

### location 블록
server 블록 안에 위치하면서 특정 URL을 처리하는 방법을 정의
* `http://myhomepage.com/module/1`과 `http://myhomepage.com/course/1`을 다르게 처리하고 싶을 때

### events 블록
네트워크 동작 방법과 관련된 설정 값
* 이벤트 블록의 지시어는 이벤트 블록 내에서만 사용 가능
* http, server, location과 상속 관계를 가지지 않음
* 참고링크 - [이벤트 모듈 지시어 사전](http://opentutorials.org/module/384/4534)5

#### 지시어 목록
* `worker_connections` : 하나의 worker가 동시에 처리할 수 있는 접속 수
* `multi_accept` : on으로 지정할 경우 한 번에 복수의 접속을 처리
* `use` : epoll 또는 queue로 설정
  * **epoll** : linux 커널 2.6 이상일 경우
  * **queue** : BSD의 경우

## 설정 변경 내용 반영하기

    sudo service nginx reload
    
restart를 해도 되지만 권장되지 않음 

## 관련 명령어

### 구동하기

    nginx
    
아무 응답이 없으면 nginx가 잘 실행된 것

### 정지하기

    nginx -s stop
    nginx -s quit
    
stop은 프로세스를 바로 종료, quit는 현재 커넥션이 모두 종료된 후에 종료

### 리로드하기

    nginx -s reload
    
설정 파일을 수정한 후 서버에 반영할 때, 아무 문구가 없으면 정상 반영

***
## 참고링크
* [Nginx 이해하기 및 기본 환경설정 세팅(AWS)](https://whatisthenext.tistory.com/123)
* [opentutorials - NGINX 환경설정](https://opentutorials.org/module/384/4526)
* [nginx 설정 정리](http://bong8nim.com/post/programming/etc/nginx-config-manual/)
* [Nginx 주요 설정(nginx.conf)](https://sarc.io/index.php/nginx/61-nginx-nginx-conf)
* [명령어로 NGINX 컨트롤하기](https://gongzza.github.io/linux/how-to-control-nginx/)

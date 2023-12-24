# 배포

서버 배포는 복잡한 영역으로, 어떤 종류의 서비스에 Uvicorn을 배포할 것인지에 따라 달라집니다.

일반적인 규칙으로, 당신은 아마도 다음을 원할 것입니다:

* 로컬 개발을 위해 명령줄에서 `uvicorn --reload`를 실행합니다.
* 프로덕션을 위해 `gunicorn -k uvicorn.workers.UvicornWorker`를 실행합니다.
* 또한 자체 호스팅을 위해 Nginx 뒤에서 실행합니다.
* 마지막으로, 캐싱 지원과 DDOS 보호를 위해 모든 것을 CDN 뒤에서 실행합니다.

## 명령줄에서 실행

일반적으로는 명령줄에서 `uvicorn`을 실행합니다.

```bash
$ uvicorn main:app --reload --port 5000
```

ASGI 애플리케이션은 `path.to.module:instance.path` 형식으로 지정해야 합니다.

로컬에서 실행하는 경우에는 `--reload`를 사용하여 자동 로딩을 킬 수 있습니다.

`--reload` 및 `--workers` 매개변수는 **상호 배타적**입니다.

사용 가능한 전체 옵션 세트를 보려면 `uvicorn --help`를 사용하세요:

<!-- :cli_usage: -->
```
$ uvicorn --help
사용법: uvicorn [OPTIONS] APP

Options:
  --host TEXT                     소켓을 이 호스트에 바인딩합니다.  [기본 값:
                                  127.0.0.1]
  --port INTEGER                  소켓을 이 포트에 바인딩합니다. 0이면 사용 가능한
                                  포트가 선택됩니다.  [기본 값: 8000]
  --uds TEXT                      UNIX 도메인 소켓에 바인딩합니다.
  --fd INTEGER                    이 파일 디스크립터의 소켓에 바인딩합니다.
  --reload                        자동 새로 고침을 활성화합니다.
  --reload-dir PATH               현재 작업 디렉토리를 사용하는 대신 명시적으로 자동
                                  새로 고침 디렉토리를 설정합니다.
  --reload-include TEXT           파일을 감시할 때 포함할 glob 패턴을 설정합니다.
                                  기본적으로 '*.py'를 포함합니다; 이 기본 값은
                                  '--reload-exclude'로 재정의할 수 있습니다.
                                  이 옵션은 watchfiles가 설치되지 않았다면 효과가
                                  없습니다.
  --reload-exclude TEXT           파일을 감시할 때 제외할 glob 패턴을 설정합니다.
                                  기본적으로 '.*, .py[cod], .sw.*,~*'를 포함
                                  합니다; 이 기본 값은 '--reload-include'로
                                  재정의 할 수 있습니다. 이 옵션은 watchfiles가
                                  설치되지 않았다면 효과가 없습니다.
  --reload-delay FLOAT            이전 검사와 다음 검사 사이의 지연은 애플리케이션이
                                  필요한 경우에 설정됩니다. 기본 값은 0.25초입니다.
                                  [기본 값: 0.25]
  --workers INTEGER               워커 프로세스의 수. 가능한 경우 $WEB_CONCURRENCY
                                  환경 변수를 기본 값으로 사용하거나, 1을 사용합니다.
                                  --reload와 함께 사용할 수 없습니다.
  --loop [auto|asyncio|uvloop]    이벤트 루프 구현  [기본 값: auto]
  --http [auto|h11|httptools]     HTTP 프로토콜 구현  [기본 값:
                                  auto]
  --ws [auto|none|websockets|wsproto]
                                  웹소켓 프로토콜 구현
                                  [기본 값: auto]
  --ws-max-size INTEGER           웹소켓 최대 메시지 크기(바이트 단위)
                                  [기본 값: 16777216]
  --ws-max-queue INTEGER          웹소켓 메시지 큐의 최대 길이
                                  [기본 값: 32]
  --ws-ping-interval FLOAT        웹소켓 핑 간격  [기본 값: 20.0]
  --ws-ping-timeout FLOAT         웹소켓 핑 타임아웃  [기본 값: 20.0]
  --ws-per-message-deflate BOOLEAN
                                  웹소켓 단일 메시지 압축 설정
                                  [기본 값: True]
  --lifespan [auto|on|off]        수명 주기 구현  [기본 값: auto]
  --interface [auto|asgi3|asgi2|wsgi]
                                  애플리케이션 인터페이스로 ASGI3, ASGI2, 또는
                                  WSGI를 선택.  [기본 값: auto]
  --env-file PATH                 환경 설정 파일을 지정.
  --log-config PATH               로그 구성 파일을 지정. 지원되는 형식:
                                  .ini, .json, .yaml.
  --log-level [critical|error|warning|info|debug|trace]
                                  로깅 레벨. [기본 값: info]
  --access-log / --no-access-log  엑세스 로그를 활성화/비활성화
  --use-colors / --no-use-colors  컬러 로그를 활성화/비활성화
  --proxy-headers / --no-proxy-headers
                                  원격 주소 정보를 채우기 위해 X-Forwarded-Proto
                                  X-Forwarded-For, X-Forwarded-Port를
                                  활성화/비활성화합니다.
  --server-header / --no-server-header
                                  기본 server 헤더를 활성화/비활성화.
  --date-header / --no-date-header
                                  기본 날짜 헤더를 활성화/비활성화.
  --forwarded-allow-ips TEXT      프록시 헤더를 신뢰할 수 IP 목록을 쉼표로 구분하여
                                  설정합니다. 가능한 경우 $FORWARDED_ALLOW_IPS
                                  환경 변수를 기본값으로 사용하거나, '127.0.0.1'을
                                  사용합니다.
  --root-path TEXT                특정 URL 경로 아래에 마운트된 애플리케이션을 위해
                                  ASGI 'root_path'를 설정합니다.
  --limit-concurrency INTEGER     HTTP 503 응답을 발행하기 전에 허용하는 동시 연결
                                  또는 작업의 최대 수를 설정합니다.
  --backlog INTEGER               대기열에서 보유할 최대 연결 수
  --limit-max-requests INTEGER    프로세스를 종료하기 전에 처리할 요청의 최대 수를
                                  설정합니다.
  --timeout-keep-alive INTEGER    이 제한 시간 내에 새 데이터가 수신되지 않으면 연결
                                  유지를 종료  [기본 값:
                                  5]
  --timeout-graceful-shutdown INTEGER
                                  자동 종료를 기다리는 최대 시간(초)
  --ssl-keyfile TEXT              SSL 키 파일
  --ssl-certfile TEXT             SSL 인증서 파일
  --ssl-keyfile-password TEXT     SSL 키 파일 패스워드
  --ssl-version INTEGER           사용할 SSL 버전 (stdlib ssl 모듈 참조)
                                  [기본 값: 17]
  --ssl-cert-reqs INTEGER         클라이언트 인증서 필요 여부 (stdlib
                                  ssl 모듈 참조)  [기본 값: 0]
  --ssl-ca-certs TEXT             CA 인증서 파일
  --ssl-ciphers TEXT              사용할 암호화 (stdlib ssl 모듈 참조)
                                  [기본 값: TLSv1]
  --header TEXT                   Name:Value 쌍으로 사용자 지정 기본 HTTP 응답
                                  헤더를 지정합니다.
  --version                       uvicorn 버전을 표시하고 종료합니다.
  --app-dir TEXT                  지정된 디렉토리에서 APP을 찾습니다. 이것을
                                  PYTHONPATH에 추가합니다.
                                  기본 값은 현재 작업 디렉토리입니다.
  --h11-max-incomplete-event-size INTEGER
                                  h11의 경우, 완료되지 않은 이벤트의 버퍼에 대한
                                  버퍼의 최대 바이트 수.
  --factory                       APP을 애플리케이션 팩토리, 즉 
                                  () -> <ASGI app> 호출 가능한 객체로 취급합니다.
  --help                          이 메시지를 표시하고 종료합니다.
```


uvicorn에서 지원되는 옵션에 대한 자세한 내용은 [설정 설명서](settings.md)를 참고하세요.

## 프로그래밍 방식으로 실행

Python 프로그램 내에서 직접 실행하려면 `uvicorn.run(app, **config)`을 사용해야 합니다. 예를 들어:

```py title="main.py"
import uvicorn

class App:
    ...

app = App()

if __name__ == "__main__":
    uvicorn.run("main:app", host="127.0.0.1", port=5000, log_level="info")
```

구성 옵션 집합은 명령줄 도구와 동일합니다.

앱 가져오기 문자열 대신 애플리케이션 인스턴스 자체가 전달될 수 있다는 점을 유의하세요.

```python
uvicorn.run(app, host="127.0.0.1", port=5000, log_level="info")
```

그러나 이 스타일은 멀티프로세싱(`workers=NUM`) 또는 리로딩(`reload=True`)을 사용하지 않는 경우에만 작동하므로, 
가져오기 문자열 스타일을 사용하는 것을 추천합니다.

또한 이 경우, 메인 모듈에서 `uvicorn.run`을 `if __name__ == '__main__'`절에 넣어야 한다는 점을 유의하세요.

!!! note
    `reload`와 `workers` 매개변수는 **상호 배타적**입니다.

## 프로세스 관리자 사용법

프로세스 관리자를 사용하여 Uvicorn을 실행하면 여러 프로세스를 견고하게 실행할 수 있으며, 요청을 놓치지 않고 서버 업그레이드를 수행할 수 있습니다.

프로세스 관리자는 소켓 설정을 처리하고, 여러 서버 프로세스를 시작하며, 프로세스 생존을 모니터링하고, 프로세스 재시작, 종료 또는 실행 중인 프로세스 수를 조정하기 위한 신호를 감지합니다.

Uvicorn은 예를 들어 `--workers 4`와 같이 여러 워커 프로세스를 가볍게 실행할 수 있는 방법을 제공하지만, 프로세스 모니터링은 제공하지 않습니다..

### Gunicorn

Gunicorn은 프로덕션 환경에서 Uvicorn을 실행하고 관리하는 가장 간단한 방법일 것입니다. Uvicorn에는 Gunicorn worker 클래스가 포함되어 있어 설정이 거의 필요하지 않습니다.

다음은 네 개의 worker 프로세스로 Gunicorn을 시작합니다:

`gunicorn -w 4 -k uvicorn.workers.UvicornWorker`

`UvicornWorker` 구현은 `uvloop` 및 `httptools` 구현을 사용합니다. PyPy에서 실행하려면 대신 순수 파이썬 구현을 사용하는 것이 좋습니다. `UvicornH11Worker` 클래스를 사용하면 됩니다.

`gunicorn -w 4 -k uvicorn.workers.UvicornH11Worker`

Gunicorn은 Uvicorn과 다른 구성 옵션 집합을 제공하므로 Gunicorn을 실행할 때 `--limit-concurrency`와 같은 일부 옵션은 아직 지원되지 않습니다.

Uvicorn의 구성 인자를 Gunicorn worker에 전달해야 하는 경우, `UvicornWorker`를 서브 클래싱해야 합니다:

```python
from uvicorn.workers import UvicornWorker

class MyUvicornWorker(UvicornWorker):
    CONFIG_KWARGS = {"loop": "asyncio", "http": "h11", "lifespan": "off"}
```

### Supervisor

`supervisor`를 프로세스 관리자로 사용하려면 다음 중 하나를 선택해야 합니다:

* 소켓을 uvicorn에 파일 디스크립터를 사용하여 넘겨줍니다. Supervisor는 이것을 항상 0으 로 사용 가능하게 하며, 이것은 `fcgi-program` 섹션에서 설정해야 합니다.
* 또는 각 `uvicorn` 프로세스에 대해 UNIX 도메인 소켓을 사용하세요.

간단한 Supervisor 구성은 다음과 같습니다:

```ini title="supervisord.conf"
[supervisord]

[fcgi-program:uvicorn]
socket=tcp://localhost:8000
command=venv/bin/uvicorn --fd 0 main:App
numprocs=4
process_name=uvicorn-%(process_num)d
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
```

그런 다음 `supervisord -n`으로 실행합니다.

### Circus

`circus`를 프로세스 관리자로 사용하기 위해서는 다음 중 하나를 수행해야 합니다:

* 소켓을 uvicorn에 파일 디스크립터를 사용하여 넘겨줍니다. circus는 이것을 `$(circus.sockets.web)`로 사용 가능하게 합니다.
* 또는 각 uvicorn 프로세스에 대해 UNIX 도메인 소켓을 사용하세요.

간단한 circus 구성은 다음과 같은 모습일 수 있습니다:

```ini title="circus.ini"
[watcher:web]
cmd = venv/bin/uvicorn --fd $(circus.sockets.web) main:App
use_sockets = True
numprocesses = 4

[socket:web]
host = 0.0.0.0
port = 8000
```

그런 다음 `circusd circus.ini`으로 실행합니다.

## Nginx 뒤에서 실행하기

Nginx를 Uvicorn 프로세스 앞의 프록시로 사용하는 것은 필수는 아니지만 추가적인 복원력을 위해 권장됩니다. Nginx는 정적 미디어를 제공하고 느린 요청을 버퍼링하여 애플리케이션 서버의 부하를 최대한 줄일 수 있습니다.

`Heroku`와 같은 관리형 환경에서는 서버 프로세스가 이미 로드 밸런싱 프록시 뒤에서 실행되고 있기 때문에 일반적으로 Nginx를 구성할 필요는 없습니다.

Nginx에서 프록시하는 데 권장되는 구성은 Nginx와 Uvicorn 실행에 사용되는 프로세스 관리자 사이에 UNIX 도메인 소켓을 사용하는 것입니다. 
이 경우 도메인 소켓이 헤더를 프록시할 소스로서 신뢰할 수 있도록 `--forwarded-allow-ips='*'`로 Uvicorn을 실행해야 한다는 점에 유의하세요.

프록시 서버로 애플리케이션을 전달할 때는 애플리케이션이 들어오는 연결의 클라이언트 주소를 올바르게 확인할 수 있도록 프록시가 헤더를 설정하는지, 연결이 `http` 또는 `https`를 통해 이루어졌는지 확인해야 합니다.

프록시에서 X-Forwarded-For 및 X-Forwarded-Proto 헤더가 설정되어 있는지, 그리고 `--proxy-headers` 설정을 사용하여 Uvicorn이 실행되는지 확인해야 합니다. 이렇게 하면 ASGI scope에 올바른 `client` 및 `scheme` 정보가 포함되도록 할 수 있습니다.

다음은 간단한 Nginx 구성의 모습입니다. 이 예제에는 프록시 헤더 설정과 UNIX 도메인 소켓을 사용하여 애플리케이션 서버와 통신하는 방법이 포함되어 있습니다.

또한 웹소켓 연결을 전달하기 위한 몇 가지 기본 구성도 포함되어 있습니다. 이에 대한 자세한 내용은 [Nginx 권장 사항][nginx_websocket]을 참조하세요.

```conf
http {
  server {
    listen 80;
    client_max_body_size 4G;

    server_name example.com;

    location / {
      proxy_set_header Host $http_host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      proxy_redirect off;
      proxy_buffering off;
      proxy_pass http://uvicorn;
    }

    location /static {
      # path for static files
      root /path/to/app/static;
    }
  }

  map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
  }

  upstream uvicorn {
    server unix:/tmp/uvicorn.sock;
  }

}
```

다양한 헤더 조합을 사용하는 복잡한 프록시 구성이나 애플리케이션이 둘 이상의 프록시 서비스 뒤에서 실행되는 경우 Uvicorn의 `--proxy-headers` 동작이 충분하지 않을 수 있습니다.

이러한 경우 ASGI 미들웨어를 사용하여 요청 헤더에 따라 `client` 및 `scheme`를 설정할 수 있습니다.

## CDN 뒤에서 실행하기

Cloudflare 또는 Cloud Front와 같은 콘텐츠 전송 네트워크(CDN) 뒤에서 운영하는 것은 DDoS 공격에 대한 강력한 보호층을 제공합니다. 서비스는 대규모의 트래픽을 처리하기 위해 설계된 거대한 프록시 및 로드 밸런서 클러스터 뒤에서 운영되며, DDoS 공격에서 오는 연결을 감지하고 차단하는 기능을 가지고 있습니다.

캐시 제어 헤더를 올바르게 사용하면 CDN이 항상 요청을 서버로 전달하지 않고도 대량의 데이터를 제공할 수 있습니다.

콘텐츠 전송 네트워크는 또한 HTTPS 종료를 제공하는 데 들이는 노력이 적은 방법일 수 있습니다.

## HTTPS와 함께 실행하기

https로 uvicorn을 실행하려면 인증서와 개인 키가 필요합니다. 
권장되는 인증서를 얻는 방법은[Let's Encrypt][letsencrypt]를 사용하는 것입니다.

https를 사용하는 로컬 개발의 경우 [mkcert][mkcert]를 
사용하여 유효한 인증서와 비공개 키를 생성할 수 있습니다. 

```bash
$ uvicorn main:app --port 5000 --ssl-keyfile=./key.pem --ssl-certfile=./cert.pem
```

### gunicorn worker로 실행하기

Uvicorn worker와 함께 인증서를 Gunicorn에 사용할 수도 있습니다.

```bash
$ gunicorn --keyfile=./key.pem --certfile=./cert.pem -k uvicorn.workers.UvicornWorker main:app
```

[nginx_websocket]: https://nginx.org/en/docs/http/websocket.html
[letsencrypt]: https://letsencrypt.org/
[mkcert]: https://github.com/FiloSottile/mkcert

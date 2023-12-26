<p align="center">
  <img width="320" height="320" src="../../uvicorn.png" alt='uvicorn'>
</p>

<p align="center">
<em>Python용 ASGI 웹 서버.</em>
</p>

<p align="center">
<a href="https://github.com/encode/uvicorn/actions">
    <img src="https://github.com/encode/uvicorn/workflows/Test%20Suite/badge.svg" alt="Test Suite">
</a>
<a href="https://pypi.org/project/uvicorn/">
    <img src="https://badge.fury.io/py/uvicorn.svg" alt="Package version">
</a>
<a href="https://pypi.org/project/uvicorn" target="_blank">
    <img src="https://img.shields.io/pypi/pyversions/uvicorn.svg?color=%2334D058" alt="Supported Python versions">
</a>
</p>

---

# 소개

Uvicorn은 Python용 ASGI 웹 서버를 구현한 것입니다.

최근까지 Python은 비동기 프레임워크를 위한 최소한의 저수준 서버/애플리케이션 인터페이스가 부족했습니다. 
[ASGI 사양][asgi]은 이 공백을 채워주며, 
이제 우리는 모든 비동기 프레임워크에서 사용할 수 있는 공통 도구 세트를 구축하기 시작할 수 있게 되었습니다.

Uvicorn은 현재 **HTTP/1.1**과 **웹소켓**을 지원하고 있습니다.

## 빠른 시작

`pip`를 사용하여 설치하기:

```shell
$ pip install uvicorn
```

위와 같이 하면 최소한의(순수 파이썬) 종속성으로 uvicorn이 설치됩니다.

```shell
$ pip install 'uvicorn[standard]'
```

위와 같이 하면 가능한 경우 'Cython 기반' 의존성과 다른 '선택적 추가 기능'이 포함된 uvicorn을 설치합니다.

이 문맥에서, 'Cython 기반'은 다음을 의미합니다:

- 가능한 경우 이벤트 루프 `uvloop`가 설치되어 사용됩니다.
  - uvloop은 내장 asyncio 이벤트 루프의 빠르고 간단한 대체제입니다. 이것은 Cython으로 구현되었습니다. [여기서][uvloop_docs] 더 읽어보세요.
  - 내장 asyncio 이벤트 루프는 쉽게 읽을 수 있는 참조 구현으로, 순수 Python 기반으로 되어 있어 쉽게 디버깅할 수 있습니다.
- 가능한 경우 HTTP 프로토콜은 `httptools`에 의해 처리될 것입니다.
  - `h11`과의 비교에 대해서는 [여기][httptools_vs_h11]에서 더 읽어보세요.

또한, '선택적 추가 기능'은 다음을 의미합니다:

- 가능한 경우 웹소켓 프로토콜은 `websockets`에 의해 처리될 것입니다(만약 `wsproto`를 사용하고 싶다면 수동으로 설치해야 합니다).
- 개발 모드에서의 `--reload` 플래그는 `watchfiles`를 사용할 것입니다.
- 윈도우 사용자는 컬러 로그를 위해 `colorama`가 설치될 것입니다.
- `--env-file` 옵션을 사용하고 싶다면 `python-dotenv`가 설치될 것입니다.
- 원한다면 `--log-config`에  `.yaml` 파일을 제공하기 위해  `PyYAML`이 설치될 것입니다.

애플리케이션을 만듭니다:

```python title="main.py"
async def app(scope, receive, send):
    assert scope['type'] == 'http'

    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            [b'content-type', b'text/plain'],
        ],
    })
    await send({
        'type': 'http.response.body',
        'body': b'Hello, world!',
    })
```

서버를 가동합니다:

```shell
$ uvicorn main:app
```

---

## 사용법

uvicorn 명령줄 도구는 애플리케이션을 실행하는 가장 쉬운 방법입니다.

### 명령줄 옵션(원래는 전부 영어로 출력됨. 아래에 나오는 한글은 모두 번역된 것. -역자)¶

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

자세한 내용은 [설정 설명서](settings.md)를 참고하세요.

### 프로그래밍 방식으로 실행

애플리케이션에서 직접 uvicorn을 실행하는 방법에는 여러 가지가 있습니다.

#### `uvicorn.run`

`uvicorn` 명령줄 인터페이스와 동등한 프로그래밍 방식의 인터페이스를 찾고 있다면 `uvicorn.run()`을 사용하세요:

```py title="main.py"
import uvicorn

async def app(scope, receive, send):
    ...

if __name__ == "__main__":
    uvicorn.run("main:app", port=5000, log_level="info")
```

#### `Config` 및 `Server` 인스턴스

구성 및 서버 수명 주기를 더 자세히 제어하려면 `uvicorn.Config` 및 `uvicorn.Server`를 사용하세요:

```py title="main.py"
import uvicorn

async def app(scope, receive, send):
    ...

if __name__ == "__main__":
    config = uvicorn.Config("main:app", port=5000, log_level="info")
    server = uvicorn.Server(config)
    server.run()
```

이미 실행 중인 비동기 환경에서 `uvicorn`을 실행하려면 `uvicorn.Server.serve()`을 대신 사용하세요:

```py title="main.py"
import asyncio
import uvicorn

async def app(scope, receive, send):
    ...

async def main():
    config = uvicorn.Config("main:app", port=5000, log_level="info")
    server = uvicorn.Server(config)
    await server.serve()

if __name__ == "__main__":
    asyncio.run(main())
```

### Gunicorn과 함께 실행하기

[Gunicorn][gunicorn]은 모든 기능을 갖춘 성숙한 서버 및 프로세스 관리자입니다.

Uvicorn은 ASGI 애플리케이션을 실행할 수 있는 Gunicorn 워커 클래스를 포함하고 있어, 
Uvicorn의 성능 이점을 모두 활용하면서도 Gunicorn의 완벽한 프로세스 관리 기능을 제공합니다.

이를 통해 워커 프로세스의 수를 실시간으로 증가시키거나 감소시키고, 
워커 프로세스를 원활하게 재시작하거나, 서버 업그레이드를 다운타임 없이 수행할 수 있습니다.

프로덕션 배포에는 uvicorn 워커 클래스와 함께 gunicorn을 사용하는 것을 추천합니다.

```
gunicorn example:app -w 4 -k uvicorn.workers.UvicornWorker
```

[PyPy][pypy]와 호환되는 구성의 경우 `uvicorn.workers.UvicornH11Worker`를 사용하세요.

자세한 내용은 [배포 설명서](deployment.md)를 참조하세요.

### 애플리케이션 팩토리

`--factory` 플래그를 사용하면 애플리케이션 인스턴스가 아닌 팩토리 함수에서 애플리케이션을 직접 로드할 수 있습니다. 
팩토리는 인자 없이 호출되며 ASGI 애플리케이션을 반환해야 합니다.

```py title="main.py"
def create_app():
    app = ...
    return app
```

```shell
$ uvicorn --factory main:create_app
```

## ASGI 인터페이스

Uvicorn은 애플리케이션과 상호작용하기 위해 [ASGI 사양][asgi]을 사용합니다.

애플리케이션은 세 개의 인수를 받는 비동기 콜러블을 노출해야 합니다:

* `scope` - 들어오는 연결에 대한 정보가 포함된 딕셔너리입니다.
* `receive` - 서버로부터 들어오는 메시지를 받기 위한 채널입니다.
* `send` - 서버로 나가는 메시지를 보내기 위한 채널입니다.

두 가지 일반적인 패턴을 사용할 수 있는 데, 하나는 함수 기반 애플리케이션입니다:

```python
async def app(scope, receive, send):
    assert scope['type'] == 'http'
    ...
```

또는 인스턴스 기반 애플리케이션입니다:

```python
class App:
    async def __call__(self, scope, receive, send):
        assert scope['type'] == 'http'
        ...

app = App()
```

애플리케이션이 처리하지 않는 범위 유형에 대해서는 예외를 발생시키는 것이 좋습니다.

`scope` 인수의 내용과, `receive` 및 `send`에서 예상되는 메시지는 사용 중인 프로토콜에 따라 달라집니다.

HTTP 메시지 형식은 [ASGI HTTP 메시지 형식][asgi-http]에 설명되어 있습니다.

### HTTP Scope

들어오는 HTTP 요청의 연결 `scope`는 다음과 같을 수 있습니다:

```python
{
    'type': 'http',
    'scheme': 'http',
    'root_path': '',
    'server': ('127.0.0.1', 8000),
    'http_version': '1.1',
    'method': 'GET',
    'path': '/',
    'headers': [
        (b'host', b'127.0.0.1:8000'),
        (b'user-agent', b'curl/7.51.0'),
        (b'accept', b'*/*')
    ]
}
```

### HTTP 메시지

인스턴스 코루틴은 `send` 코루틴에 메시지를 보내 서버와 다시 통신합니다.

```python
await send({
    'type': 'http.response.start',
    'status': 200,
    'headers': [
        [b'content-type', b'text/plain'],
    ]
})
await send({
    'type': 'http.response.body',
    'body': b'Hello, world!',
})
```

### 요청 & 응답

다음은 수신 요청에 사용된 메서드와 경로를 표시하는 예제입니다:

```python
async def app(scope, receive, send):
    """
    메서드와 경로를 HTTP 응답으로 되돌려 보냅니다.
    """
    assert scope['type'] == 'http'

    body = f'Received {scope["method"]} request to {scope["path"]}'
    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            [b'content-type', b'text/plain'],
        ]
    })
    await send({
        'type': 'http.response.body',
        'body': body.encode('utf-8'),
    })
```

### 요청 본문 읽기

`send` 코루틴에서 메시지를 가져와서 비동기 작업 풀을 차단하지 않고 요청 본문을 스트리밍할 수 있습니다.

```python
async def read_body(receive):
    """
    수신되는 ASGI 메시지에서 본문 전체를 읽고 반환합니다.
    """
    body = b''
    more_body = True

    while more_body:
        message = await receive()
        body += message.get('body', b'')
        more_body = message.get('more_body', False)

    return body


async def app(scope, receive, send):
    """
    요청 본문을 HTTP 응답으로 되돌려 보냅니다.
    """
    body = await read_body(receive)
    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            (b'content-type', b'text/plain'),
            (b'content-length', str(len(body)).encode())
        ]
    })
    await send({
        'type': 'http.response.body',
        'body': body,
    })
```

### 응답 스트리밍

`send` 코루틴에 여러 개의 `http.response.body` 메시지를 전송하여 응답을 스트리밍할 수 있습니다.

```python
import asyncio


async def app(scope, receive, send):
    """
    클라이언트에게 점진적으로 데이터를 스트리밍하는 HTTP 응답을 보냅니다.
    """
    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            [b'content-type', b'text/plain'],
        ]
    })
    for chunk in [b'Hello', b', ', b'world!']:
        await send({
            'type': 'http.response.body',
            'body': chunk,
            'more_body': True
        })
        await asyncio.sleep(1)
    await send({
        'type': 'http.response.body',
        'body': b'',
    })
```

---

## 왜 ASGI인가?

잘 정립된 대부분의 Python 웹 프레임워크는 WSGI 기반 프레임워크로 시작되었습니다.

WSGI 애플리케이션은 요청을 받아 응답을 반환하는 단일 동기 호출 가능 객체입니다. 
이것은 WSGI가 잘 지원하지 않는, 긴 지속성을 가진 연결(예를 들어, 긴 폴링 HTTP 또는 WebSocket 연결)을 
허용하지 않습니다.

비동기 동시성 모델을 가짐으로써 가벼운 백그라운드 작업과 같은 옵션을 허용하며, 
느린 HTTP 요청과 같이 네트워크 I/O에 오랫동안 차단되는 엔드포인트에 대한 
제한 요소가 줄어들 수 있습니다.

---

## 대체 ASGI 서버들

ASGI 프로토콜의 강점은 서버 구현을 애플리케이션 프레임워크에서 분리한다는 점입니다. 
이를 통해 상호 운용되는 웹 서버와 애플리케이션 프레임워크의 생태계를 구축할 수 있습니다.

### Daphne

[Daphne webserver][daphne]은 원래 Django Channels을 구동하기 위해 개발된, 최초의 ASGI 서버 구현입니다.

이 서버는 프로덕션 환경에서 광범위하게 실행되고 있으며 HTTP/1.1, HTTP/2 및 웹소켓을 지원합니다.

여기서 제공된 예제 애플리케이션 중 어느 것이든 `daphne`을 사용하여 똑같이 실행할 수 있습니다.

```
$ pip install daphne
$ daphne app:App
```

### Hypercorn

[Hypercorn][hypercorn]은 처음에 Quart 웹 프레임워크의 일부였으나 독립형 ASGI 서버로 분리되었습니다.

Hypercorn HTTP/1.1, HTTP/2, HTTP/3과 웹소켓을 지원합니다.

```
$ pip install hypercorn
$ hypercorn app:App
```

---

## ASGI 프레임워크들

Uvicorn, Daphne, 또는 Hypercorn을 사용하여 모든 ASGI 프레임워크를 실행할 수 있습니다.

소규모 서비스의 경우 ASGI 애플리케이션을 직접 작성할 수도 있습니다.

### Starlette

[Starlette](https://github.com/encode/starlette)은 경량 ASGI 프레임워크/툴킷입니다.

고성능 비동기 서비스를 구축하는 데 이상적이며 HTTP와 웹소켓 모두 지원합니다.

### Django Channels

ASGI 사양은 원래 [Django Channels](https://channels.readthedocs.io/en/latest/)과 함께 사용하도록 설계되었습니다.

Channels는 비동기 프론트엔드를 스레드 기반 백엔드에 제공한다는 점에서 다른 ASGI 프레임워크와 약간 다릅니다. 
이를 통해 Django는 웹소켓, 백그라운드 작업 및 장시간 실행되는 연결을 지원할 수 있으며, 
응용 프로그램 코드는 여전히 표준 스레드 컨텍스트에서 실행됩니다.

### Quart

[Quart](https://pgjones.gitlab.io/quart/)는 Flask와 유사한 ASGI 웹 프레임워크입니다.

### FastAPI

[**FastAPI**](https://github.com/tiangolo/fastapi)은 Starlette과 Pydantic을 기반으로 한, 이전 서버 버전인 APIStar에서 크게 영감을 받은 API 프레임워크입니다.

Python 3.6+ 타입 선언을 사용하여 API 함수 매개변수를 작성하면 자동 데이터 변환, 데이터 유효성 검사, OpenAPI 스키마(JSON 스키마 포함) 및 대화형 API 문서화 UI를 얻을 수 있습니다.

### BlackSheep

[BlackSheep](https://www.neoteroi.dev/blacksheep/)는 Flask와 ASP.NET Core에서 영감을 받은 ASGI 기반의 웹 프레임워크입니다.

이 프레임워크의 가장 독특한 특징은 의존성 주입에 대한 내장 지원, 요청 핸들러의 타입 주석에 의한 매개변수 자동 바인딩, OpenAPI 문서 및 Swagger UI의 자동 생성입니다.

### Falcon

[Falcon](https://falconframework.org)은 Python을 위한 최소주의적인 REST 및 애플리케이션 백엔드 프레임워크로, 신뢰성, 정확성 및 대규모 성능에 중점을 둡니다.

### Muffin

[Muffin](https://github.com/klen/muffin)은 Python 3을 위한 빠르고 가벼우며 비동기적인 ASGI 웹 프레임워크입니다.

### Litestar

[**Litestar**](https://litestar.dev)는 강력하고 가벼우며 유연한 ASGI 프레임워크입니다.

이 프레임워크에는 데이터 직렬화 및 검증부터 웹소켓, ORM 통합, 세션 관리, 인증 등 현대적인 API를 구축하는 데 필요한 모든 것이 포함되어 있습니다.


[uvloop]: https://github.com/MagicStack/uvloop
[httptools]: https://github.com/MagicStack/httptools
[gunicorn]: https://gunicorn.org/
[pypy]: https://pypy.org/
[asgi]: https://asgi.readthedocs.io/en/latest/
[asgi-http]: https://asgi.readthedocs.io/en/latest/specs/www.html
[daphne]: https://github.com/django/daphne
[hypercorn]: https://gitlab.com/pgjones/hypercorn
[uvloop_docs]: https://uvloop.readthedocs.io/
[httptools_vs_h11]: https://github.com/python-hyper/h11/issues/9

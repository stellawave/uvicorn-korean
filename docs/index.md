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

이렇게 하면 같이 하면 최소한의(순수 파이썬) 종속성으로 uvicorn이 설치됩니다.

```shell
$ pip install 'uvicorn[standard]'
```

이렇게 하면 가능한 경우 'Cython 기반' 의존성과 다른 '선택적 추가 기능'이 포함된 uvicorn을 설치합니다.

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

Uvicorn uses the [ASGI specification][asgi] for interacting with an application.

The application should expose an async callable which takes three arguments:

* `scope` - A dictionary containing information about the incoming connection.
* `receive` - A channel on which to receive incoming messages from the server.
* `send` - A channel on which to send outgoing messages to the server.

Two common patterns you might use are either function-based applications:

```python
async def app(scope, receive, send):
    assert scope['type'] == 'http'
    ...
```

Or instance-based applications:

```python
class App:
    async def __call__(self, scope, receive, send):
        assert scope['type'] == 'http'
        ...

app = App()
```

It's good practice for applications to raise an exception on scope types
that they do not handle.

The content of the `scope` argument, and the messages expected by `receive` and `send` depend on the protocol being used.

The format for HTTP messages is described in the [ASGI HTTP Message format][asgi-http].

### HTTP Scope

An incoming HTTP request might have a connection `scope` like this:

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

### HTTP Messages

The instance coroutine communicates back to the server by sending messages to the `send` coroutine.

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

### Requests & responses

Here's an example that displays the method and path used in the incoming request:

```python
async def app(scope, receive, send):
    """
    Echo the method and path back in an HTTP response.
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

### Reading the request body

You can stream the request body without blocking the asyncio task pool,
by fetching messages from the `receive` coroutine.

```python
async def read_body(receive):
    """
    Read and return the entire body from an incoming ASGI message.
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
    Echo the request body back in an HTTP response.
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

### Streaming responses

You can stream responses by sending multiple `http.response.body` messages to
the `send` coroutine.

```python
import asyncio


async def app(scope, receive, send):
    """
    Send a slowly streaming HTTP response back to the client.
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

## Why ASGI?

Most well established Python Web frameworks started out as WSGI-based frameworks.

WSGI applications are a single, synchronous callable that takes a request and returns a response.
This doesn’t allow for long-lived connections, like you get with long-poll HTTP or WebSocket connections,
which WSGI doesn't support well.

Having an async concurrency model also allows for options such as lightweight background tasks,
and can be less of a limiting factor for endpoints that have long periods being blocked on network
I/O such as dealing with slow HTTP requests.

---

## Alternative ASGI servers

A strength of the ASGI protocol is that it decouples the server implementation
from the application framework. This allows for an ecosystem of interoperating
webservers and application frameworks.

### Daphne

The first ASGI server implementation, originally developed to power Django Channels, is [the Daphne webserver][daphne].

It is run widely in production, and supports HTTP/1.1, HTTP/2, and WebSockets.

Any of the example applications given here can equally well be run using `daphne` instead.

```
$ pip install daphne
$ daphne app:App
```

### Hypercorn

[Hypercorn][hypercorn] was initially part of the Quart web framework, before
being separated out into a standalone ASGI server.

Hypercorn supports HTTP/1.1, HTTP/2, HTTP/3 and WebSockets.

```
$ pip install hypercorn
$ hypercorn app:App
```

---

## ASGI frameworks

You can use Uvicorn, Daphne, or Hypercorn to run any ASGI framework.

For small services you can also write ASGI applications directly.

### Starlette

[Starlette](https://github.com/encode/starlette) is a lightweight ASGI framework/toolkit.

It is ideal for building high performance asyncio services, and supports both HTTP and WebSockets.

### Django Channels

The ASGI specification was originally designed for use with [Django Channels](https://channels.readthedocs.io/en/latest/).

Channels is a little different to other ASGI frameworks in that it provides
an asynchronous frontend onto a threaded-framework backend. It allows Django
to support WebSockets, background tasks, and long-running connections,
with application code still running in a standard threaded context.

### Quart

[Quart](https://pgjones.gitlab.io/quart/) is a Flask-like ASGI web framework.

### FastAPI

[**FastAPI**](https://github.com/tiangolo/fastapi) is an API framework based on **Starlette** and **Pydantic**, heavily inspired by previous server versions of **APIStar**.

You write your API function parameters with Python 3.6+ type declarations and get automatic data conversion, data validation, OpenAPI schemas (with JSON Schemas) and interactive API documentation UIs.

### BlackSheep

[BlackSheep](https://www.neoteroi.dev/blacksheep/) is a web framework based on ASGI, inspired by Flask and ASP.NET Core.

Its most distinctive features are built-in support for dependency injection, automatic binding of parameters by request handler's type annotations, and automatic generation of OpenAPI documentation and Swagger UI.

### Falcon

[Falcon](https://falconframework.org) is a minimalist REST and app backend framework for Python, with a focus on reliability, correctness, and performance at scale.

### Muffin

[Muffin](https://github.com/klen/muffin) is a fast, lightweight and asynchronous ASGI web-framework for Python 3.

### Litestar

[**Litestar**](https://litestar.dev) is a powerful, lightweight and flexible ASGI framework.

It includes everything that's needed to build modern APIs - from data serialization and validation to websockets, ORM integration, session management, authentication and more.


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

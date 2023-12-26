<p align="center">
  <img width="320" height="320" src="https://raw.githubusercontent.com/tomchristie/uvicorn/master/docs/uvicorn.png" alt='uvicorn'>
</p>

<p align="center">
<em>Python을 위한 ASGI 웹 서버</em>
</p>

---

[![Build Status](https://github.com/encode/uvicorn/workflows/Test%20Suite/badge.svg)](https://github.com/encode/uvicorn/actions)
[![Package version](https://badge.fury.io/py/uvicorn.svg)](https://pypi.python.org/pypi/uvicorn)
[![Supported Python Version](https://img.shields.io/pypi/pyversions/uvicorn.svg?color=%2334D058)](https://pypi.org/project/uvicorn)

**공식 문서**: [https://www.uvicorn.org](https://www.uvicorn.org)

**필수 요구 사항**: Python 3.8+

Uvicorn은 Python용 ASGI 웹 서버를 구현한 것입니다.

최근까지 Python은 비동기 프레임워크를 위한 최소한의 저수준 서버/애플리케이션 인터페이스가 부족했습니다.
async frameworks. [ASGI 사양][asgi] 이 공백을 채워주며, 이제 우리는 모든 비동기 프레임워크에서 사용할 수 있는 공통 도구 세트를 구축하기 시작할 수 있게 되었습니다.

Uvicorn은 현재 HTTP/1.1과 웹소켓을 지원하고 있습니다.
## Uvicorn-korean
**Uvicorn-korean은 Uvicorn의 비공식 한국어 번역입니다.**

한국어 학습자를 위해 문서, 주석에 있는 영어를 최대한 번역하는 것을 목표로 합니다.
uvicorn Version 0.25.0을 기반으로 번역되었으며 학습 목적으로 사용하시는 것을 추천합니다. 

<br/>

---

<br/>

## 빠른 시작

`pip`를 사용하여 설치:

```shell
$ pip install uvicorn
```

아래와 같이 하면 최소한의(순수 파이썬) 종속성으로 uvicorn이 설치됩니다.

```shell
$ pip install 'uvicorn[standard]'
```

이것은 가능한 경우 'Cython 기반' 의존성과 다른 '선택적 추가 기능'이 포함된 uvicorn을 설치합니다.

이 문맥에서, 'Cython 기반'은 다음을 의미합니다:

- 가능한 경우 이벤트 루프 `uvloop`가 설치되어 사용됩니다.
- 가능한 경우 HTTP 프로토콜은 `httptools`에 의해 처리될 것입니다.

또한, '선택적 추가 기능'은 다음을 의미합니다:

- 가능한 경우 웹소켓 프로토콜은 `websockets`에 의해 처리될 것입니다(만약 `wsproto`를 사용하고 싶다면 수동으로 설치해야 합니다).
- 개발 모드에서의 `--reload` 플래그는 `watchfiles`를 사용할 것입니다.
- 윈도우 사용자는 컬러 로그를 위해 `colorama`가 설치될 것입니다.
- `--env-file` 옵션을 사용하고 싶다면 `python-dotenv`가 설치될 것입니다.
- 원한다면 `--log-config`에  `.yaml` 파일을 제공하기 위해  `PyYAML`이 설치될 것입니다.

`example.py`에서 애플리케이션을 만듭니다:

```python
async def app(scope, receive, send):
    assert scope['type'] == 'http'

    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            (b'content-type', b'text/plain'),
        ],
    })
    await send({
        'type': 'http.response.body',
        'body': b'Hello, world!',
    })
```

서버를 가동합니다:

```shell
$ uvicorn example:app
```

---

## 왜 ASGI인가?

잘 정립된 대부분의 Python 웹 프레임워크는 WSGI 기반 프레임워크로 시작되었습니다.

WSGI 애플리케이션은 요청을 받아 응답을 반환하는 단일 동기 호출 가능 객체입니다. 
이것은 WSGI가 잘 지원하지 않는, 긴 지속성을 가진 연결(예를 들어, 롱 폴링 HTTP 또는 WebSocket 연결)을 허용하지 않습니다.

비동기 동시성 모델을 가짐으로써 가벼운 백그라운드 작업과 같은 옵션을 허용하며, 느린 HTTP 요청과 같이 네트워크 I/O에 오랫동안 차단되는 엔드포인트에 대한 제한 요소가 줄어들 수 있습니다.

---

## 대체 ASGI 서버들

ASGI 프로토콜의 강점은 서버 구현을 애플리케이션 프레임워크에서 분리한다는 점입니다. 
이를 통해 상호 운용되는 웹 서버와 애플리케이션 프레임워크의 에코 시스템을 구축할 수 있습니다.

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

Hypercorn은 HTTP/1.1, HTTP/2 및 웹소켓을 지원합니다.

또한 `asyncio`의 대안으로 [우수한 비동기 프레임워크 `trio`][trio]를 지원합니다.

```
$ pip install hypercorn
$ hypercorn app:App
```

### Mangum

[Mangum][mangum]은 AWS Lambda 및 API Gateway에서 ASGI 애플리케이션을 사용하기 위한 어댑터입니다.

---

<p align="center"><i>Uvicorn은 <a href="https://github.com/encode/uvicorn/blob/master/LICENSE.md">BSD licensed</a>입니다.<br/>세심하게 디자인되고 제작되었습니다.</i><br/>&mdash; 🦄  &mdash;</p>

[asgi]: https://asgi.readthedocs.io/en/latest/
[daphne]: https://github.com/django/daphne
[hypercorn]: https://github.com/pgjones/hypercorn
[mangum]: https://mangum.io
[trio]: https://trio.readthedocs.io

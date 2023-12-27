# 설정

명령줄에서 Uvicorn을 실행할 때 다음 옵션들을 사용하여 Uvicorn을 구성하세요.

프로그래밍 방식으로 `uvicorn.run(...)`을 사용하여 실행하는 경우, 동등한 키워드 인자를 사용하세요. 
예를 들어, `uvicorn.run("example:app", port=5000, reload=True, access_log=False)`와 같이 사용합니다.
이 경우 `reload=True`나 `workers=NUM`을 사용한다면, 
메인 모듈에서 `if __name__ == '__main__'` 절 안에 `uvicorn.run`을 넣어야 한다는 점을 유의하세요.

또한 UVICORN_ 접두사를 가진 환경 변수를 사용하여 `Uvicorn`을 구성할 수도 있습니다.
예를 들어, 앱을 포트 `5000`에서 실행하고 싶다면, 환경 변수 `UVICORN_PORT`를 `5000`으로 설정하세요.

!!! note
    CLI 옵션과 `uvicorn.run()`의 인수는 환경 변수보다 우선합니다.

    또한 환경 설정 파일 내에서는 `UVICORN_*` 접두사가 붙은 설정은 사용할 수 없습니다. 환경 설정 파일에 `--env-file` 플래그를 사용하는 것은 uvicorn 자체를 구성하는 것이 아니라 uvicorn이 실행하는 ASGI 애플리케이션을 구성하기 위한 것입니다.

## 애플리케이션

* `APP` - 실행할 ASGI 애플리케이션을 `"<module>:<attribute>"` 형식으로 입력합니다.
* `--factory` - `APP`을 애플리케이션 팩토리, 즉 `() -> <ASGI app>` 콜러블로 취급합니다.

## 소켓 바인딩

* `--host <str>` - 소켓을 해당 호스트에 바인딩합니다. 로컬 네트워크에서 애플리케이션을 사용할 수 있게 하려면`--host 0.0.0.0`을 사용합니다. IPv6 주소가 지원합니다. 예를 들어: `--host '::'`. **기본값:** *'127.0.0.1'*.
* `--port <int>` - 소켓을 해당 포트에 바인딩합니다. **기본값:** *8000*.
* `--uds <path>` - UNIX 도메인 소켓에 바인딩합니다(예: `--uds /tmp/uvicorn.sock`). 역방향 프록시 뒤에서 Uvicorn을 실행하려는 경우에 유용합니다. 
* `--fd <int>` - 해당 파일 디스크립터에 소켓을 바인딩합니다. 프로세스 관리자 내에서 Uvicorn을 실행하려는 경우 유용합니다.

## 개발

* `--reload` - 자동 리로드 활성화. Uvicorn은 이 옵션으로 활성화되는 두 가지 버전의 자동 새로 고침 동작을 지원합니다. 이 두 가지 버전에는 중요한 차이점이 있습니다.
* `--reload-dir <path>` - 파이썬 파일 변경을 감시하는 디렉터리를 지정합니다. 여러 번 사용할 수 있습니다. 사용하지 않으면 기본적으로 현재 디렉터리 전체가 감시됩니다. 프로그래밍 방식으로 실행하는 경우 `reload_dirs=[]`를 사용하고 문자열 리스트를 전달하세요.

### watchfiles 없이 리로드하기

Uvicorn이 런타임에 [watchfiles](https://pypi.org/project/watchfiles/)를 불러올 수 없는 경우, 모니터링하는 디렉터리 내의 모든 `*.py` 파일(`*.py` 파일만 해당)에 대한 수정 시간 변경을 주기적으로 찾을 겁니다. `reload-dir` 옵션을 참조하세요. watchfiles가 설치되어 있지 않으면 다른 파일 확장자는 지원되지 않습니다. 자세한 내용은 `--reload-include` 및 `--reload-exclude` 옵션을 참조하세요.

### watchfiles을 사용하여 리로드하기

파일 변경이 리로드를 트리거하는 것을 더 세밀하게 제어하려면, 종속 요소로 watchfiles을 포함하는 `uvicorn[standard]`를 설치하세요. 또는 Uvicorn이 감지할 수 있는 곳에 [watchfiles](https://pypi.org/project/watchfiles/)를 설치하세요.

watchfiles와 함께 Uvicorn을 사용하면 다음 옵션들이 활성화됩니다(그렇지 않으면 무시됩니다).

* `--reload-include <glob-pattern>` - 변경 사항이 감시될 파일이나 디렉토리에 매치되는 glob 패턴을 지정하세요. 여러 번 사용할 수 있습니다. 기본적으로 다음 패턴들이 포함됩니다: `*.py`. 이 기본값들은 `--reload-exclude`에 포함시켜 덮어쓸 수 있습니다.
* `--reload-exclude <glob-pattern>` - 변경 사항 감시에서 제외될 파일이나 디렉토리에 매치되는 glob 패턴을 지정하세요. 여러 번 사용할 수 있습니다. 기본적으로 다음 패턴들이 제외됩니다: `.*, .py[cod], .sw.*, ~*`. 이 기본값들은 `--reload-include`에 포함시켜 덮어쓸 수 있습니다.

!!! tip
    [WSL](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux)을 통해 Uvicorn을 사용할 때, 파일 변경이 리로드를 트리거하도록 하려면 `WATCHFILES_FORCE_POLLING` 환경 변수를 설정해야 할 수도 있습니다.
자세한 내용은 [watchfiles 문서](https://watchfiles.helpmanual.io/api/watch/)를 참조하세요.

## 프로덕션

* `--workers <int>` - 여러 워커 프로세스를 사용합니다. 가능한 경우 `$WEB_CONCURRENCY` 환경 변수를 기본값으로 사용하거나, 그렇지 않으면 1로 설정됩니다.

## 로깅

* `--log-config <path>` - 로깅 구성 파일입니다. **옵션:** *`dictConfig()` 형식: .json, .yaml*. 다른 형식은 `fileConfig()`를 사용하여 처리됩니다. 자동 감지된 동작을 덮어쓰려면 `formatters.default.use_colors` 및 `formatters.access.use_colors` 값을 설정하세요.
    * 로깅 구성에 YAML 파일을 사용하려면 프로젝트에 PyYAML을 종속성으로 포함하거나 `[standard]` 선택적 추가 기능을 사용하여 uvicorn을 설치해야 합니다.
* `--log-level <str>` - 로그 레벨을 설정합니다. **옵션:** *'critical', 'error', 'warning', 'info', 'debug', 'trace'.* **기본값:** *'info'*.
* `--no-access-log` - 로그 레벨을 변경하지 않고, 액세스 로그를 비활성화합니다.
* `--use-colors / --no-use-colors` - 로그 기록의 색상 형식을 활성화/비활성화합니다. 이 옵션이 설정되지 않으면 자동 감지됩니다. 이 옵션은 `--log-config` CLI 옵션을 사용하는 경우 무시됩니다.

## 구현

* `--loop <str>` - 이벤트 루프 구현을 설정합니다. uvloop 구현은 더 높은 성능을 제공하지만, Windows나 PyPy와 호환되지 않습니다. **옵션:** *'auto', 'asyncio', 'uvloop'.* **기본값:** *'auto'*.
* `--http <str>` - HTTP 프로토콜 구현을 설정합니다. httptools 구현은 더 높은 성능을 제공하지만, PyPy와 호환되지 않습니다. **옵션:** *'auto', 'h11', 'httptools'.* **기본값:** *'auto'*.
* `--ws <str>` - 웹소켓 프로토콜 구현을 설정합니다. `websockets` 및 `wsproto` 패키지 모두 지원됩니다. 모든 웹소켓 요청을 무시하려면 `'none'`을 사용하세요. **옵션:** *'auto', 'none', 'websockets', 'wsproto'.* **기본값:** *'auto'*.
* `--ws-max-size <int>` - 웹소켓의 최대 메시지 크기를 바이트 단위로 설정합니다. 이 옵션은 기본 `websockets` 프로토콜에서만 사용할 수 있습니다.
* `--ws-max-queue <int>` - 웹소켓 들어오는 메시지 큐의 최대 길이를 설정합니다. 이 옵션은 기본 `websockets` 프로토콜에서만 사용할 수 있습니다.
* `--ws-ping-interval <float>` - 웹소켓의 핑 간격을 초 단위로 설정합니다. 이 옵션은 기본 `websockets` 프로토콜에서만 사용할 수 있습니다. **기본값:** *20.0*
* `--ws-ping-timeout <float>` - 웹소켓의 핑 타임아웃을 초 단위로 설정합니다. 이 옵션은 기본 `websockets` 프로토콜에서만 사용할 수 있습니다. **기본값:** *20.0*
* `--lifespan <str>` - Lifespan 프로토콜 구현을 설정합니다. **옵션:** *'auto', 'on', 'off'.* **기본값:** *'auto'*.
* `--h11-max-incomplete-event-size <int>` - 완료되지 않은 이벤트의 버퍼에 저장할 최대 바이트 수를 설정합니다.  이 옵션은 `h11` HTTP 프로토콜 구현에만 사용 가능합니다. **기본값:** *'16384'* (16 KB).

## 애플리케이션 인터페이스

* `--interface` - 애플리케이션 인터페이스로 ASGI3, ASGI2, 또는 WSGI를 선택합니다.
WSGI 모드는 WSGI 인터페이스에서 지원되지 않기 때문에 항상 웹소켓 지원을 비활성화합니다.
**옵션:** *'auto', 'asgi3', 'asgi2', 'wsgi'.* **기본값:** *'auto'*.

!!! 경고
    Uvicorn의 네이티브 WSGI 구현은 더 이상 사용되지 않으므로, [a2wsgi](https://github.com/abersheeran/a2wsgi)로 전환해야 합니다 (`pip install a2wsgi`).

## HTTP

* `--root-path <str>` - 특정 URL 경로 아래에 서브마운트된 애플리케이션을 위한 ASGI `root_path`를 설정합니다.
* `--proxy-headers` / `--no-proxy-headers` - X-Forwarded-Proto, X-Forwarded-For, X-Forwarded-Port를 활성화/비활성화하여 원격 주소 정보를 채웁니다. 기본적으로 활성화되어 있지만, `forwarded-allow-ips` 설정에 있는 연결 IP만 신뢰합니다.
* `--forwarded-allow-ips` <comma-separated-list> 프록시 헤더로 신뢰할 수 있는 IP 목록을 쉼표로 구분하여 설정합니다. 기본값은 `$FORWARDED_ALLOW_IPS` 환경 변수(사용 가능한 경우) 또는 '127.0.0.1'입니다. 와일드카드 '*'는 항상 신뢰함을 의미합니다.
* `--server-header` / `--no-server-header` - 기본 `Server` 헤더를 활성화/비활성화합니다.
* `--date-header` / `--no-date-header` - 기본 `Date` 헤더를 활성화/비활성화합니다.

!!! 주의
    `--no-date-header` 플래그는 `websockets` 구현에 영향을 주지 않습니다.

## HTTPS

다음 옵션으로 [SSL 컨텍스트](https://docs.python.org/3/library/ssl.html#ssl.SSLContext)를 구성할 수 있습니다:

* `--ssl-keyfile <path>` - SSL 키 파일입니다.
* `--ssl-keyfile-password <str>` - SSL 키를 해독하는 데 필요한 비밀번호입니다.
* `--ssl-certfile <path>` - SSL 인증서 파일입니다.
* `--ssl-version <int>` - 사용할 SSL 버전입니다.
* `--ssl-cert-reqs <int>` - 클라이언트 인증서가 필요한지 여부입니다.
* `--ssl-ca-certs <str>` - CA 인증서 파일입니다.
* `--ssl-ciphers <str>` - 사용할 암호 알고리즘입니다.

SSL 컨텍스트 옵션에 대해 더 자세히 알고 싶다면 [Python 문서](https://docs.python.org/3/library/ssl.html)를 참조하세요.

## 리소스 제한

* `--limit-concurrency <int>` - HTTP 503 응답을 발행하기 전에 허용하는 동시 연결 또는 작업의 최대 수입니다. 리소스 과부하 상태에서도 알려진 메모리 사용 패턴을 보장하는 데 유용합니다.
* `--limit-max-requests <int>` - 프로세스를 종료하기 전에 서비스할 요청의 최대 수입니다. 프로세스 관리자와 함께 실행할 때, 장기간 실행되는 프로세스에 영향을 미치는 메모리 누수를 방지하는 데 유용합니다.
* `--backlog <int>` - 대기열에서 보유할 최대 연결 수입니다. 많은 들어오는 트래픽에 관련됩니다. **기본값:** *2048*

## 타임아웃

* `--timeout-keep-alive <int>` - 이 타임아웃 내에 새 데이터가 수신되지 않으면 Keep-Alive 연결을 닫습니다. **기본값:** *5*.
* `--timeout-graceful-shutdown <int>` - 원활한 종료를 위해 기다리는 최대 초 수입니다. 이 타임아웃 후에 서버는 요청을 종료하기 시작합니다.
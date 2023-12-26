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

### watchfiles 없이 리로딩하기

Uvicorn이 런타임에 [watchfiles](https://pypi.org/project/watchfiles/)를 로드할 수 없는 경우, 모니터링하는 디렉터리 내의 모든 `*.py` 파일(`*.py` 파일만 해당)에 대한 수정 시간 변경을 주기적으로 찾을 겁니다. `reload-dir` 옵션을 참조하세요. watchfiles가 설치되어 있지 않으면 다른 파일 확장자는 지원되지 않습니다. 자세한 내용은 `--reload-include` 및 `--reload-exclude` 옵션을 참조하세요.

### watchfiles을 사용하여 리로딩하기

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
* `--no-access-log` - 로그 레벨을 변경하지 않고 액세스 로그를 비활성화합니다.
* `--use-colors / --no-use-colors` - 로그 기록의 색상 형식을 활성화/비활성화합니다. 이 옵션이 설정되지 않으면 자동 감지됩니다. 이 옵션은 `--log-config` CLI 옵션을 사용하는 경우 무시됩니다.

## Implementation

* `--loop <str>` - Set the event loop implementation. The uvloop implementation provides greater performance, but is not compatible with Windows or PyPy. **Options:** *'auto', 'asyncio', 'uvloop'.* **Default:** *'auto'*.
* `--http <str>` - Set the HTTP protocol implementation. The httptools implementation provides greater performance, but it not compatible with PyPy. **Options:** *'auto', 'h11', 'httptools'.* **Default:** *'auto'*.
* `--ws <str>` - Set the WebSockets protocol implementation. Either of the `websockets` and `wsproto` packages are supported. Use `'none'` to ignore all websocket requests. **Options:** *'auto', 'none', 'websockets', 'wsproto'.* **Default:** *'auto'*.
* `--ws-max-size <int>` - Set the WebSockets max message size, in bytes. Please note that this can be used only with the default `websockets` protocol.
* `--ws-max-queue <int>` - Set the maximum length of the WebSocket incoming message queue. Please note that this can be used only with the default `websockets` protocol.
* `--ws-ping-interval <float>` - Set the WebSockets ping interval, in seconds. Please note that this can be used only with the default `websockets` protocol. **Default:** *20.0*
* `--ws-ping-timeout <float>` - Set the WebSockets ping timeout, in seconds. Please note that this can be used only with the default `websockets` protocol. **Default:** *20.0*
* `--lifespan <str>` - Set the Lifespan protocol implementation. **Options:** *'auto', 'on', 'off'.* **Default:** *'auto'*.
* `--h11-max-incomplete-event-size <int>` - Set the maximum number of bytes to buffer of an incomplete event. Only available for `h11` HTTP protocol implementation. **Default:** *'16384'* (16 KB).

## Application Interface

* `--interface` - Select ASGI3, ASGI2, or WSGI as the application interface.
Note that WSGI mode always disables WebSocket support, as it is not supported by the WSGI interface.
**Options:** *'auto', 'asgi3', 'asgi2', 'wsgi'.* **Default:** *'auto'*.

!!! warning
    Uvicorn's native WSGI implementation is deprecated, you should switch
    to [a2wsgi](https://github.com/abersheeran/a2wsgi) (`pip install a2wsgi`).

## HTTP

* `--root-path <str>` - Set the ASGI `root_path` for applications submounted below a given URL path.
* `--proxy-headers` / `--no-proxy-headers` - Enable/Disable X-Forwarded-Proto, X-Forwarded-For, X-Forwarded-Port to populate remote address info. Defaults to enabled, but is restricted to only trusting
connecting IPs in the `forwarded-allow-ips` configuration.
* `--forwarded-allow-ips` <comma-separated-list> Comma separated list of IPs to trust with proxy headers. Defaults to the `$FORWARDED_ALLOW_IPS` environment variable if available, or '127.0.0.1'. A wildcard '*' means always trust.
* `--server-header` / `--no-server-header` - Enable/Disable default `Server` header.
* `--date-header` / `--no-date-header` - Enable/Disable default `Date` header.

!!! note
    The `--no-date-header` flag doesn't have effect on the `websockets` implementation.

## HTTPS

The [SSL context](https://docs.python.org/3/library/ssl.html#ssl.SSLContext) can be configured with the following options:

* `--ssl-keyfile <path>` - The SSL key file.
* `--ssl-keyfile-password <str>` - The password to decrypt the ssl key.
* `--ssl-certfile <path>` - The SSL certificate file.
* `--ssl-version <int>` - The SSL version to use.
* `--ssl-cert-reqs <int>` - Whether client certificate is required.
* `--ssl-ca-certs <str>` - The CA certificates file.
* `--ssl-ciphers <str>` - The ciphers to use.

To understand more about the SSL context options, please refer to the [Python documentation](https://docs.python.org/3/library/ssl.html).

## Resource Limits

* `--limit-concurrency <int>` - Maximum number of concurrent connections or tasks to allow, before issuing HTTP 503 responses. Useful for ensuring known memory usage patterns even under over-resourced loads.
* `--limit-max-requests <int>` - Maximum number of requests to service before terminating the process. Useful when running together with a process manager, for preventing memory leaks from impacting long-running processes.
* `--backlog <int>` - Maximum number of connections to hold in backlog. Relevant for heavy incoming traffic. **Default:** *2048*

## Timeouts

* `--timeout-keep-alive <int>` - Close Keep-Alive connections if no new data is received within this timeout. **Default:** *5*.
* `--timeout-graceful-shutdown <int>` - Maximum number of seconds to wait for graceful shutdown. After this timeout, the server will start terminating requests.

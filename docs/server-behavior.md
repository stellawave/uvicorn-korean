# 서버 동작 방식

Uvicorn은 강력한 서버 구현을 제공하기 위해 연결 및 리소스 관리에 특히 주의를 기울여 설계되었습니다. 서버 또는 클라이언트 오류가 발생해도 정상적으로 작동하고, 클라이언트 동작 불량 또는 서비스 거부 공격에 대한 복원력을 보장하는 것을 목표로 합니다.

## HTTP 헤더

`Server` 및 `Date` 헤더가 모든 발신 요청에 추가됩니다.

만약 `Connection: Close` 헤더가 포함된 경우, Uvicorn은 응답 후 연결을 닫습니다. 그렇지 않으면 연결이 열려 있는 상태로 유지되며 연결 유지 시간 초과를 기다립니다.

`Content-Length` 헤더가 포함된 경우, Uvicorn은 응답 본문의 콘텐츠 길이가 헤더의 값과 일치하는지 확인하고 그렇지 않은 경우 오류를 발생시킵니다.

`Content-Length` 헤더가 포함되지 않은 경우, Uvicorn은 응답 본문에 청크 인코딩을 사용하며, 필요한 경우 `Transfer-Encoding` 헤더를 설정합니다.

`Transfer-Encoding` 헤더가 포함된 경우, `Content-Length` 헤더는 무시됩니다.

HTTP 헤더는 대소문자를 구분하지 않아야 합니다. Uvicorn은 항상 소문자로만 응답 헤더를 전송합니다.

---

## 흐름 제어

Proper flow control ensures that large amounts of data do not become buffered on the transport when either side of a connection is sending data faster than its counterpart is able to handle.

### 쓰기 흐름 제어

If the write buffer passes a high water mark, then Uvicorn ensures the ASGI `send` messages will only return once the write buffer has been drained below the low water mark.

### 읽기 흐름 제어

Uvicorn will pause reading from a transport once the buffered request body hits a high water mark, and will only resume once `receive` has been called, or once the response has been sent.

---

## Request and Response bodies

### Response completion

Once a response has been sent, Uvicorn will no longer buffer any remaining request body. Any later calls to `receive` will return an `http.disconnect` message.

Together with the read flow control, this behavior ensures that responses that return without reading the request body will not stream any substantial amounts of data into memory.

### Expect: 100-Continue

The `Expect: 100-Continue` header may be sent by clients to require a confirmation from the server before uploading the request body. This can be used to ensure that large request bodies are only sent once the client has confirmation that the server is willing to accept the request.

Uvicorn ensures that any required `100 Continue` confirmations are only sent if the ASGI application calls `receive` to read the request body.

Note that proxy configurations may not necessarily forward on `Expect: 100-Continue` headers. In particular, Nginx defaults to buffering request bodies, and automatically sends `100 Continues` rather than passing the header on to the upstream server.

### HEAD requests

Uvicorn will strip any response body from HTTP requests with the `HEAD` method.

Applications should generally treat `HEAD` requests in the same manner as `GET` requests, in order to ensure that identical headers are sent in both cases, and that any ASGI middleware that modifies the headers will operate identically in either case.

One exception to this might be if your application serves large file downloads, in which case you might wish to only generate the response headers.

---

## Timeouts

Uvicorn provides the following timeouts:

* Keep-Alive. Defaults to 5 seconds. Between requests, connections must receive new data within this period or be disconnected.

---

## Resource Limits

Uvicorn provides the following resource limiting:

* Concurrency. Defaults to `None`. If set, this provides a maximum number of concurrent tasks *or* open connections that should be allowed. Any new requests or connections that occur once this limit has been reached will result in a "503 Service Unavailable" response. Setting this value to a limit that you know your servers are able to support will help ensure reliable resource usage, even against significantly over-resourced servers.
* Max requests. Defaults to `None`. If set, this provides a maximum number of HTTP requests that will be serviced before terminating a process. Together with a process manager this can be used to prevent memory leaks from impacting long running processes.

---

## Server Errors

Server errors will be logged at the `error` log level. All logging defaults to being written to `stdout`.

### Exceptions

If an exception is raised by an ASGI application, and a response has not yet been sent on the connection, then a `500 Server Error` HTTP response will be sent.

### Invalid responses

Uvicorn will ensure that ASGI applications send the correct sequence of messages, and will raise errors otherwise. This includes checking for no response sent, partial response sent, or invalid message sequences being sent.

---

## Graceful Process Shutdown

Graceful process shutdowns are particularly important during a restart period. During this period you want to:

* Start a number of new server processes to handle incoming requests, listening on the existing socket.
* Stop the previous server processes from listening on the existing socket.
* Close any connections that are not currently waiting on an HTTP response, and wait for any other connections to finalize their HTTP responses.
* Wait for any background tasks to run to completion, such as occurs when the ASGI application has sent the HTTP response, but the asyncio task has not yet run to completion.

Uvicorn handles process shutdown gracefully, ensuring that connections are properly finalized, and all tasks have run to completion. During a shutdown period Uvicorn will ensure that responses and tasks must still complete within the configured timeout periods.

---

## HTTP Pipelining

HTTP/1.1 provides support for sending multiple requests on a single connection, before having received each corresponding response. Servers are required to support HTTP pipelining, but it is now generally accepted to lead to implementation issues. It is not enabled on browsers, and may not necessarily be enabled on any proxies that the HTTP request passes through.

Uvicorn supports pipelining pragmatically. It will queue up any pipelined HTTP requests, and pause reading from the underlying transport. It will not start processing pipelined requests until each response has been dealt with in turn.

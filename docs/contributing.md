# 기여하기

Uvicorn에 관심을 가져주셔서 감사합니다.
프로젝트에 기여할 수 있는 방법은 여러 가지가 있습니다:

- Uvicorn을 여러분의 기술 스택에서 사용하고 [버그 발견하기/이슈 보고하기](https://github.com/encode/uvicorn/issues/new)
- [새 기능 구현 및 버그 수정](https://github.com/encode/uvicorn/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)
- [다른 사용자의 Pull Requests 검토](https://github.com/encode/uvicorn/pulls)
- 문서 작성하기
- 토론 참여하기

## 버그, 이슈 또는 기능 요청 보고하기

Uvicorn이 지원해줘야 할 것을 발견하셨나요?
예상치 못한 동작을 발견하셨나요?
누락된 기능이 필요하신가요?

기여는 일반적으로 이전 토론에서 시작해야 합니다.
[커뮤니티 챗](https://gitter.im/encode/community)
또는 [github 토론 탭](https://github.com/encode/uvicorn/discussions)에서
다른 사람에게 연락할 수 있습니다.

토론 탭에서 새로운 주제를 생성할 때, 버그는 "Potential Issue" 토론으로, 
기능 요청은 "Ideas" 토론으로 제기할 수 있습니다. 
그 후에 해당 토론을 "Issue"로 격상할 필요가 있는지, 
또는 풀 리퀘스트를 고려할지 여부를 결정할 수 있습니다.

버그 신고의 경우 가능한 한 사세하게 설명하고,
다음과 같이 최대한 많은 정보를 제공해주세요:

- 운영체제 플랫폼
- Python 버전
- 설치된 종속성 및 버전 (`python -m pip freeze`)
- 코드 스니펫
- 오류 traceback


모든 예시는 문제를 보여주는 *가장 간단한 사례*로 줄이려고 항상 노력해야 합니다.

잠재적인 문제 범위를 좁히는 데 유용한 몇 가지 팁을 알려드립니다...

- `Multiprocess`와 같은 특정 슈퍼바이저에 문제가 있나요, 아니면 둘 이상의 슈퍼바이저에 문제가 있나요?
- 문제가 asgi에 있나요, wsgi에 있나요, 아니면 둘 다에 있나요?
- Uvicorn을 Gunicorn과 함께 운영하시나요, 아니면 다른 서비스와 함께 운영하시나요, 아니면 단독으로 운영하시나요?

## 개발

Uvicorn 개발을 시작하려면 Github에 있는 
[Uvicorn 리포지토리](https://github.com/encode/uvicorn)를 **fork**하세요


그런 다음 fork를 clone합니다.
`YOUR-USERNAME`을 당신의 GitHub 이름으로 바꾸세요:

```shell
$ git clone https://github.com/YOUR-USERNAME/uvicorn
```

이제 다음을 사용하여 프로젝트 및 해당 종속성을 설치할 수 있습니다:

```shell
$ cd uvicorn
$ scripts/install
```

## 테스트 및 코드 검사하기

사용자 지정 셸 스크립트를 사용하여 테스트, 린팅, 
문서 작성 워크플로우를 자동화합니다.

테스트를 실행하려면 다음을 실행하세요:

```shell
$ scripts/test
```

추가 인자는 `pytest`로 전달됩니다. 자세한 내용은 [pytest 문서](https://docs.pytest.org/en/latest/how-to/usage.html)를 참조하십시오.

예를 들어 단일 테스트 스크립트를 실행하는 경우:

```shell
$ scripts/test tests/test_cli.py
```

코드 자동 서식 지정을 실행하려면

```shell
$ scripts/lint
```

마지막으로 코드 검사를 별도로 실행하려면 (`scripts/test`의 일부로 실행되기도 함), 다음을 실행합니다:

```shell
$ scripts/check
```

## 문서화

문서 페이지는 `docs/` 폴더 아래에 위치해 있습니다.

문서 사이트를 로컬에서 실행하려면(변경 사항 미리 보기에 유용) 다음을 사용합니다:

```shell
$ scripts/docs serve
```

## 빌드 / CI 실패 해결

풀 리퀘스트를 제출하면 테스트 스위트가 자동으로 실행되며, 결과는 GitHub에 표시됩니다.
테스트 suite가 실패하면 'Details' 링크를 클릭하여 테스트 suite가 왜 실패했는지 파악하려고 시도해야 합니다.

<p align="center" style="margin: 0 0 10px">
  <img src="https://raw.githubusercontent.com/encode/uvicorn/master/docs/img/gh-actions-fail.png" alt='Failing PR commit status'>
</p>

다음은 테스트 스위트가 실패할 수 있는 몇 가지 일반적인 경우입니다:

### 실패한 작업 확인

<p align="center" style="margin: 0 0 10px">
  <img src="https://raw.githubusercontent.com/encode/uvicorn/master/docs/img/gh-actions-fail-check.png" alt='Failing GitHub action lint job'>
</p>

이 작업이 실패했다는 것은 코드 서식 지정 문제나 유형 주석 문제가 있다는 뜻입니다.
작업 출력을 확인하여 실패한 이유 또는 셸 실행 내역을 파악할 수 있습니다:

```shell
$ scripts/check
```

코드를 자동으로 형식화하려면 `$ scripts/lint`를 실행하는 것이 좋을 수 있으며, 그 작업이 성공하면 변경 사항을 커밋하세요.

### 문서 작업 실패

이 작업이 실패한다는 것은 문서 구축이 실패했다는 것을 의미합니다. 이는 잘못된 마크다운 또는 `mkdocs.yml` 내의 누락된 구성과 같은 다양한 이유로 발생할 수 있습니다.

### Python 3.X 작업 실패

이 작업이 실패한다는 것은 단위 테스트가 실패했거나 모든 코드 경로가 단위 테스트에 의해 커버되지 않았음을 의미합니다.

테스트가 실패하면 커버리지 보고서 아래에 이 메시지가 표시됩니다:

`=== 1 failed, 354 passed, 1 skipped, 1 xfailed in 37.08s ===`

테스트가 성공하더라도 커버리지가 현재 임계값에 도달하지 않으면, 커버리지 보고서 아래에 이 메시지가 표시됩니다:

`Coverage failure: total of 88 is less than fail-under=95`

## 릴리즈

*이 섹션은 Uvicorn 관리자를 대상으로 합니다.*

새 버전을 릴리즈하기 전에 다음을 포함하는 풀 리퀘스트를 생성하세요:

- **변경 로그 업데이트**:
    - [keepachangelog](https://keepachangelog.com/en/1.0.0/)의 형식을 따릅니다.
    - `master`와 최신 릴리즈 태그를 [비교](https://github.com/encode/uvicorn/compare/)하여 사용자에게 관심이 있을만한 모든 항목을 나열하세요:
        - 변경 로그에 **반드시** 포함되어야 하는 것들: 추가, 변경, 폐지 또는 제거된 기능, 버그 수정.
        - 변경 로그에 **포함되지 않아야** 하는 것들: 문서, 테스트 또는 도구의 변경.
        - 가능하면 항목을 중요도 / 영향도에 따라 내림차순으로 정렬하세요.
        - 간결하고 핵심적인 내용을 유지하세요. 🎯
- **버전 업데이트**: `__init__.py`를 참조하세요.

예시는 [#1006](https://github.com/encode/uvicorn/pull/1107)을 참조하세요.

릴리즈 PR이 병합되면 다음을 포함하여 [새 릴리즈](https://github.com/encode/uvicorn/releases/new)를 생성하세요:

- 태그 버전 예: `0.13.3`.
- 릴리즈 제목 `버전 0.13.3`
- 변경 로그에서 복사한 설명.

생성된 이 릴리즈는 자동으로 PyPI에 업로드됩니다.

PyPI 작업에 문제가 발생하면 `scripts/publish` 스크립트를 사용하여 릴리즈를 게시할 수 있습니다.
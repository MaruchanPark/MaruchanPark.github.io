---
layout: post
title: Learn go with tdd (2)
excerpt: Build an application (HTTP server)
date: 2022-3-30 23:00
tags: [Go, TDD, temporal]
use_math: true
--- 

## 어떤 서버를 구현하나?
- GET /players/{name} should return a number indicating the total number of wins
- POST /players/{name} should record a win for that name, incrementing for every subsequent POST

### TDD를 위한 순서를 어떻게 정하나?
- 무슨 말인지 모르겠다. 예제를 진행하면서 이해해보자.

### 첫 번째 테스트
1. 테스트
#### **`server_test.go`**
```go
package main

import (
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestGETPlayers(t *testing.T) {
	t.Run("returns Pepper's score", func(t *testing.T) {
		request, _ := http.NewRequest(http.MethodGet, "/players/Pepper", nil)
		response := httptest.NewRecorder()

		PlayerServer(response, request)

		got := response.Body.String()
		want := "20"

		if got != want {
			t.Errorf("got '%q' want '%q'", got, want)
		}
	})
}
```
- 함수가 없으므로 컴파일 에러가 발생한다.
2. 컴파일 통과, 에러 메세지 확인
#### **`server.go`**
```go
package main

import (
	"net/http"
)

func PlayerServer(w http.ResponseWriter, r *http.Request) {

}
```
```shell
$ go test
--- FAIL: TestGETPlayers (0.00s)
    --- FAIL: TestGETPlayers/returns_Pepper's_score (0.00s)
        server_test.go:20: got '""' want '"20"'
FAIL
exit status 1
FAIL    server  0.002s
```

- 컴파일 통과, 원하는 에러를 확인했다. 다음으로 가자.
4. 테스트 통과
- 테스트 통과를 위한 최소한의 코드 작성
#### **`server.go`**
```go
func PlayerServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "20")
}
```
- ResponseWriter로 "20"을 전달해서 테스트를 통과시켰다.

5. 리팩토링
- 리팩토링 단계에서는 동작하는 서버를 만든다.
#### **`main.go`**
```go
package main

import (
	"log"
	"net/http"
)

func main() {
	handler := http.HandlerFunc(PlayerServer)
	log.Fatal(http.ListenAndServe(":5000", handler))
}
```
- `Handlerfunc` 함수를 통해서 `PlayerServer`로 `ResponseWriter`, `Request`가 전달된다.
- `ListenAndServe`로 입력 포트에 `HandlerFunc`을 통해 `PlayerServer`가 실행된다.
- 포트가 사용 중이라서 에러가 날 경우에는 log가 남겨진다.

### 두 번째 테스트
- `"20"`외에 다른 경우에도 동작하게 하기 위한 테스트를 작성한다.
1. 테스트, 컴파일 통과, 에러 메세지 확인
#### **`server_test.go`**
```go
t.Run("returns Floyd's score", func(t *testing.T) {
    request, _ := http.NewRequest(http.MethodGet, "/players/Floyd", nil)
    response := httptest.NewRecorder()

    PlayerServer(response, request)

    got := response.Body.String()
    want := "10"

    if got != want {
        t.Errorf("got '%q' want '%q'", got, want)
    }
})
```
- 서브 테스트 추가 후 테스트
```shell
$ go test -v
=== RUN   TestGETPlayers
=== RUN   TestGETPlayers/returns_Pepper's_score
=== RUN   TestGETPlayers/returns_Floyd's_score
    server_test.go:34: got '"20"' want '"10"'
--- FAIL: TestGETPlayers (0.00s)
    --- PASS: TestGETPlayers/returns_Pepper's_score (0.00s)
    --- FAIL: TestGETPlayers/returns_Floyd's_score (0.00s)
FAIL
exit status 1
FAIL    server  0.002s
```
- 에러 메세지가 적절함을 확인, 테스트를 통과하는 코드를 작성한다.
4. 테스트 통과
- `player` 별 케이스 구현
#### **`server.go`**
```go
package main

import (
	"fmt"
	"net/http"
	"strings"
)

func PlayerServer(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	if player == "Pepper" {
		fmt.Fprint(w, "20")
		return
	}

	if player == "Floyd" {
		fmt.Fprint(w, "10")
		return
	}
}
```
- 간신히 테스트만 통과할 수 있는 초 간단한 수정이다.

```
여기까지 하고 정리를 해보자.
httptest로 테스트용 response, request body를 만들고, 함수에 입력했다. api 서버를 만들고 테스트 하는 것 보다 덜 구현된 상태에서 테스트가 가능하다.
player 저장소가 없는 상태에서 꼼수(?) 값으로 테스트 하는 것도, 많은 구현 없이 TDD를 시작할 수 있게 해준다.
```
5. 리팩토링
- `PlayerServer` 기능을 분리
#### **`server.go`**
```go
func PlayerServer(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	fmt.Fprint(w, GetPlayerScore(player))
}

func GetPlayerScore(name string) string {
	if name == "Pepper" {
		return "20"
	}

	if name == "Floyd" {
		return "10"
	}
	return ""
}
```
- 테스트도 수행, 통과를 확인.
- 테스트 코드도 기능을 분리해서 리팩토링 한다.
#### **`server_test.go`**
```go
package main

import (
	"fmt"
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestGETPlayers(t *testing.T) {
	t.Run("returns Pepper's score", func(t *testing.T) {
		request := newGetScoreRequest("Pepper")
		response := httptest.NewRecorder()

		PlayerServer(response, request)

		got := response.Body.String()
		want := "20"

		assertResponseBody(t, got, want)
	})

	t.Run("returns Floyd's score", func(t *testing.T) {
		request := newGetScoreRequest("Floyd")
		response := httptest.NewRecorder()

		PlayerServer(response, request)

		got := response.Body.String()
		want := "10"

		assertResponseBody(t, got, want)
	})
}

func newGetScoreRequest(name string) *http.Request {
	req, _ := http.NewRequest(http.MethodGet, fmt.Sprintf("/players/%s", name), nil)
	return req
}

func assertResponseBody(t testing.TB, got, want string) {
	t.Helper()
	if got != want {
		t.Errorf("got '%q' want '%q'", got, want)
	}
}
```


#### **``**
```go
```

-----

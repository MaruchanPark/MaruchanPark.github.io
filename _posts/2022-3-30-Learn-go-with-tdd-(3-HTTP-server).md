---
layout: post
title: Learn go with tdd (http server)
excerpt: Build an application (HTTP server)
date: 2022-4-6 12:00
tags: [Go, TDD, HTTP]
use_math: true
--- 

HTTP 서버를 구현할 때, TDD를 어떻게 적용할 지 알아보자.  
혼자서 머리를 굴려봤는데, 시작을 어떻게 할지 정하기도 어려워서, 진도를 좀 건너뛰고, 바로 서버 구현에서의 TDD 실습을 해보겠다.  

[교재](https://quii.gitbook.io/learn-go-with-tests/build-an-application/http-server)  
[코드](https://github.com/MaruchanPark/Learn_go_with_tests/tree/main/build_an_application/HTTP_server)


# Chapter 20 : HTTP server
- 구현 할 API
    - GET /players/{name} should return a number indicating the total number of wins
    - POST /players/{name} should record a win for that name, incrementing for every subsequent POST

## [20-1] 서버 테스트 시작하기
- 실제로 동작하는 서버를 만들고 기능을 테스트 할 경우, 기능 구현에 많은 시간이 걸리고 구현하는 과정에 대한 테스트는 건너뛰게 된다.
- 최초의 테스트를 만들어보자.

### [20-1] 테스트
- `net/http/httptest`로 테스트용 `request`, `response`를 만들 수 있다.

```go
//server_test.go
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

### [20-1] 컴파일 통과, 에러 메세지 확인

```go
//server.go
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

### [20-1] 테스트 통과
- 테스트 통과를 위한 최소한의 코드 작성

```go
//server.go
func PlayerServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "20")
}
```
- ResponseWriter로 "20"을 전달해서 테스트를 통과시켰다.

### [20-1] 리팩토링
- 리팩토링 단계에서는 동작하는 서버를 만든다.

```go
//main.go
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

## [20-2] 테스트 케이스 추가
- `"20"`외에 다른 경우에도 동작하게 하기 위한 테스트를 작성한다.

### [20-2] 테스트, 컴파일 통과, 에러 메세지 확인
- `"10"`의 경우에 대한 서브 테스트를 추가해준다.

```go
//server_test.go
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
- 테스트 수행. 컴파일은 통과된다.

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

### [20-2] 테스트 통과
- `player` 별로 다른 출력을 내게 수정해준다.

```go
//server.go
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
httptest로 테스트용 response, request를 만들고, 함수에 입력했다.

서버를 만들고 실제로 response, request로 테스트를 하려고 했다면, 서버를 만드는 과정은 테스트 하지 않은 셈이 된다.

우리는 테스트를 먼저 수행하고, main 프로그램을 작성했다.
근데, main은 아직 테스트 안 했다. 계속 해보자.
```

### [20-2] 리팩토링
- `PlayerServer` 기능을 분리

```go
//server.go
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

```go
//server_test.go
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
- 인터페이스를 사용해서 코드를 다시 작성하자. (왜 사용하는지 모름...)

```go
//server.go
type PlayerStore interface {
	GetPlayerScore(name string) (score int)
}

type PlayerServer struct {
	store PlayerStore
}

func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")
	fmt.Fprint(w, p.store.GetPlayerScore(player))
}
```
- `PlayerServer`는 `PlayerStore`를 갖는다.
- `PlayerServer`에 `ServeHTTP` 함수를 bound 한다. (표현이 맞는지 모르겠다)
- `PlayerServer` 객체를 통해 인터페이스에 명시된 `GetPlayerScore` 함수를 호출 가능하다. (코드를 그대로 읽는 수준의 설명..)
- `GetPlayerScore` 함수가 구체적으로 명시되어 있지 않다는 점이 의아하다. 테스트 코드도 수정한다.

```go
//server_test.go
func TestGETPlayers(t *testing.T) {
	server := &PlayerServer{}

	t.Run("returns Pepper's score", func(t *testing.T) {
		request := newGetScoreRequest("Pepper")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertResponseBody(t, response.Body.String(), "20")
	})

	t.Run("returns Floyd's score", func(t *testing.T) {
		request := newGetScoreRequest("Floyd")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertResponseBody(t, response.Body.String(), "10")
	})
}
```
- `server` 객체의 인스턴스를 만들고, `PlayerServer`를 `ServeHTTP`로 변경해줬다.
- 컴파일 먼저 통과 시키자. `main.go`에서 사용되는 함수를 바꾼다.

```go
//main.go
package main

import (
	"log"
	"net/http"
)

func main() {
	server := &PlayerServer{}
	log.Fatal(http.ListenAndServe(":5000", server))
}
```
- 테스트를 통과하기 위해, 테스트 코드에 `PlayerStore`와 `GetPlayerScore`를 구현한다.(왜 `server.go`가 아니라 테스트 코드에 구현할까?)

```go
//server_test.go
type StubPlayerStore struct {
	scores map[string]int
}

func (s *StubPlayerStore) GetPlayerScore(name string) (score int) {
	score = s.scores[name]
	return score
}

func TestGETPlayers(t *testing.T) {
	store := StubPlayerStore{
		map[string]int{
			"Pepper": 20,
			"Floyd":  10,
		},
	}

	server := &PlayerServer{&store}

	t.Run("returns Pepper's score", func(t *testing.T) {
		request := newGetScoreRequest("Pepper")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertResponseBody(t, response.Body.String(), "20")
	})

	t.Run("returns Floyd's score", func(t *testing.T) {
		request := newGetScoreRequest("Floyd")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertResponseBody(t, response.Body.String(), "10")
	})
}
```
- `server.go`에서 명시해둔 함수와 자료형(?)을 `server_test.go`에서 선언하고, 실행했다.
- 한 곳에 쓰는게 더 나을 것 같은데.. 염두에 두고 계속 진행 해봐야겠다.

## [20-3] Application 실행
- 서버를 실행하고 브라우저로 접속을 시도하면 서버 콘솔에 다음과 같은 에러가 뜬다. (너무 길어서 윗부분만 옮겼다)

```shell
2022/04/04 14:13:50 http: panic serving 119.203.228.160:55050: runtime error: invalid memory address or nil pointer dereference
```
- `main.go`에 `PlayerStore`를 구체적으로 명시해주자.

```go
//main.go
type InMemoryPlayerStore struct{}

func (i *InMemoryPlayerStore) GetPlayerScore(name string) (score int) {
	score = 123
	return
}

func main() {
	server := &PlayerServer{&InMemoryPlayerStore{}}
	log.Fatal(http.ListenAndServe(":5000", server))
}
```
- 수정하고 빌드, 실행 후에 브라우저로 접속하면 `123`을 확인할 수 있다.
![](https://i.imgur.com/TTNYDhg.png)
- 서버가 출력을 내긴 하지만, 목표한 동작은 아니다.
- [20-4, 5, 6]에서는 player가 없는 경우, `POST /player/{name}` 케이스를 구현한다.

## [20-4] 예외처리
- 서버에 없는 player의 score를 요청할 때, 404 status를 내는지 테스트 해보자.

### [20-4] 테스트, 컴파일 통과, 에러 메세지 확인

```go
//server_test.go
t.Run("returns 404 on missing players", func(t *testing.T) {
	request := newGetScoreRequest("Apollo")
	response := httptest.NewRecorder()

	server.ServeHTTP(response, request)

	got := response.Code
	want := http.StatusNotFound

	if got != want {
		t.Errorf("got status %d want %d", got, want)
	}
})
```
```shell
$ go test
--- FAIL: TestGETPlayers (0.00s)
    --- FAIL: TestGETPlayers/returns_404_on_missing_players (0.00s)
        server_test.go:57: got status 200 want 404
FAIL
exit status 1
FAIL    server  0.002s
```
- 통과하는 코드를 작성하자.

### [20-4] 테스트 통과
- 테스트를 통과할 수 있는 최소한의 구현을 한다.

```go
//server.go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	w.WriteHeader(http.StatusNotFound)

	fmt.Fprint(w, p.store.GetPlayerScore(player))
}
```
- 무조건 404를 내는 코드를 구현했다.
- 방금 작성한 테스트를 통과할 수 있는 코드이고, 모든 경우에 404를 내니 잘못된 것이 맞다. 그런데 테스트는 통과한다.
- 이는 player가 있는 경우에는 상태 코드 200을 내는지 확인하는 테스트를 작성하지 않았다는 것을 드러내준다.
- 테스트 코드를 수정해주자.

```go
//server_test.go
func TestGETPlayers(t *testing.T) {
	store := StubPlayerStore{
		map[string]int{
			"Pepper": 20,
			"Floyd":  10,
		},
	}

	server := &PlayerServer{&store}

	t.Run("returns Pepper's score", func(t *testing.T) {
		request := newGetScoreRequest("Pepper")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatus(t, response.Code, http.StatusOK)
		assertResponseBody(t, response.Body.String(), "20")
	})

	t.Run("returns Floyd's score", func(t *testing.T) {
		request := newGetScoreRequest("Floyd")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatus(t, response.Code, http.StatusOK)
		assertResponseBody(t, response.Body.String(), "10")
	})

	t.Run("returns 404 on missing players", func(t *testing.T) {
		request := newGetScoreRequest("Apollo")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatus(t, response.Code, http.StatusNotFound)
	})
}

func assertStatus(t testing.TB, got, want int) {
	t.Helper()
	if got != want {
		t.Errorf("got status %d want %d", got, want)
	}
}
```
```shell
$ go test
--- FAIL: TestGETPlayers (0.00s)
    --- FAIL: TestGETPlayers/returns_Pepper's_score (0.00s)
        server_test.go:35: got status 404 want 200
    --- FAIL: TestGETPlayers/returns_Floyd's_score (0.00s)
        server_test.go:45: got status 404 want 200
FAIL
exit status 1
FAIL    server  0.003s
```
- 2개의 테스트가 실패하는 것을 확인할 수 있다! 예상대로다!!
- 이제 player를 찾지 못하는 경우에만 404를 내도록 코드를 수정하자.

```go
//server.go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	score := p.store.GetPlayerScore(player)

	if score == 0 {
		w.WriteHeader(http.StatusNotFound)
	}

	fmt.Fprint(w, score)
}
```
- 테스트는 통과한다! 예외처리는 구현했고, POST 요청을 구현해보자.

## [20-5] player POST (StatusCode)
### [20-5] 테스트, 컴파일 통과, 에러 메세지 확인
- POST 요청을 구분하는지, 확인하는 테스트 구현.

```go
//server_test.go
func TestStorePlayerWins(t *testing.T) {
	store := StubPlayerStore{
		map[string]int{},
	}
	server := &PlayerServer{&store}

	t.Run("it returns accepted on POST", func(t *testing.T) {
		request, _ := http.NewRequest(http.MethodPost, "/players/Pepper", nil)
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatus(t, response.Code, http.StatusAccepted)
	})
}
```
- GET 요청과 다른 상태 코드를 부여하여 GET과 구분하여 다루고자 한다. 

```shell
$ go test
--- FAIL: TestStorePlayerWins (0.00s)
    --- FAIL: TestStorePlayerWins/it_returns_accepted_on_POST (0.00s)
        server_test.go:71: got status 404 want 202
FAIL
exit status 1
FAIL    server  0.003s
```
- player `Pepper`가 존재하지 않으므로, 에러가 발생한다. 통과하게 하자!

### [20-5] 테스트 통과

```go
//server.go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {

	if r.Method == http.MethodPost {
		w.WriteHeader(http.StatusAccepted)
		return
	}

	player := strings.TrimPrefix(r.URL.Path, "/players/")

	score := p.store.GetPlayerScore(player)

	if score == 0 {
		w.WriteHeader(http.StatusNotFound)
	}

	fmt.Fprint(w, score)
}
```
- POST 일 경우 상태코드를 변경하고 바로 리턴한다. 테스트는 통과한다.

### [20-5] 리팩토링
- 함수가 길다! 분리하자!!

```go
//server.go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {

	switch r.Method {
	case http.MethodPost:
		p.processWin(w)
	case http.MethodGet:
		p.showScore(w, r)
	}
}

func (p *PlayerServer) showScore(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	score := p.store.GetPlayerScore(player)

	if score == 0 {
		w.WriteHeader(http.StatusNotFound)
	}

	fmt.Fprint(w, score)
}

func (p *PlayerServer) processWin(w http.ResponseWriter) {
	w.WriteHeader(http.StatusAccepted)
}
```
- 기능을 분리해서, POST, GET 각각이 무엇을 하는지 읽기 더 쉬운 코드가 되었다.
- 이제 POST 요청에 대해 `PlayerStore`에 저장을 해보자!

## [20-6] player POST (Store)
### [20-6] 테스트
- `POST /players/{name}`는 win player의 name을 저장한다.
- slice에 저장하고 크기를 확인하는 테스트를 하자.

```go
//server_test.go
type StubPlayerStore struct {
	scores   map[string]int
	winCalls []string
}

func (s *StubPlayerStore) GetPlayerScore(name string) (score int) {
	score = s.scores[name]
	return score
}

func (s *StubPlayerStore) RecordWin(name string) {
	s.winCalls = append(s.winCalls, name)
}
```
- 객체에 자료형을 추가해주고, api의 작업을 수행하는 함수를 작성했다.
- 테스트를 작성한다.

```go
//server_test.go
func TestStorePlayerWins(t *testing.T) {
	store := StubPlayerStore{
		map[string]int{},
	}
	server := &PlayerServer{&store}

	t.Run("it returns accepted on POST", func(t *testing.T) {
		request := newPostWinRequest("Pepper")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatus(t, response.Code, http.StatusAccepted)

		if len(store.winCalls) != 1 {
			t.Errorf("got %d calls to RecordWin want %d", len(store.winCalls), 1)
		}
	})
}

func newPostWinRequest(name string) *http.Request {
	req, _ := http.NewRequest(http.MethodPost, fmt.Sprintf("/players/%s", name), nil)
	return req
}
```
- 추가된 자료형에 대해서 객체를 선언할 때 고려해주면 컴파일을 통과할 수 있다.

### [20-6] 컴파일 통과, 에러 메세지 확인
- 최소한의 코드로 컴파일을 통과하자!

```go
//server_test.go
func TestGETPlayers(t *testing.T) {
	store := StubPlayerStore{
		map[string]int{
			"Pepper": 20,
			"Floyd":  10,
		},
		nil,//ㅋㅋㅋ
	}
```
```go
//server_test.go
func TestStorePlayerWins(t *testing.T) {
	store := StubPlayerStore{
		map[string]int{},
		nil,
	}
```
- 두 테스트 함수의 객체 선언 부분을 수정했다.
- 테스트를 통과시켜보자!

### [20-6] 테스트 통과
- `ServeHTTP`의 POST case에서 새로 구현한 함수가 사용할 수 있도록 interface를 고쳐준다.

```go
//server.go
type PlayerStore interface {
	GetPlayerScore(name string) (score int)
	RecordWin(name string)
}
``` 
- main에서 컴파일 문제가 발생한다, 빈 함수를 선언해서 컴파일만 되게 하자.
```go
//main.go
type InMemoryPlayerStore struct{}

func (i *InMemoryPlayerStore) GetPlayerScore(name string) (score int) {
	score = 123
	return
}

func (i *InMemoryPlayerStore) RecordWin(name string) {}
```
```shell
--- FAIL: TestStorePlayerWins (0.00s)
    --- FAIL: TestStorePlayerWins/it_returns_accepted_on_POST (0.00s)
        server_test.go:81: got 0 calls to RecordWin want 1
FAIL
exit status 1
FAIL    server  0.002s
```
- `RecordWin` 함수가 실행되지 않았으므로, 테스트 에러의 결과와 일치한다.
- 함수를 실행하고, 테스트를 통과시키자.

```go
//server.go
func (p *PlayerServer) processWin(w http.ResponseWriter) {
	p.store.RecordWin("Bob")
	w.WriteHeader(http.StatusAccepted)
}
```
```shell
$ go test
PASS
ok      server  0.003s
```
- Pepper를 추가해달라고 요청했는데, Bob이 추가되었다. 갯수만 세기 때문에 테스트는 통과되었다.
- 테스트를 강화하자!

```go
//server_test.go
t.Run("it returns accepted on POST", func(t *testing.T) {
    player := "Pepper"
    request := newPostWinRequest(player)
    response := httptest.NewRecorder()

    server.ServeHTTP(response, request)

    assertStatus(t, response.Code, http.StatusAccepted)

    if len(store.winCalls) != 1 {
        t.Errorf("got %d calls to RecordWin want %d", len(store.winCalls), 1)
    }

    if store.winCalls[0] != player {
        t.Errorf("did not store correct winner got %q want %q", store.winCalls[0], player)
    }
})
```
- `winCalls`에 저장된 값과 입력 player를 비교하는 코드를 추가해주었다. Bob != Pepper이므로 통과하지 않는다.
- `server.go`에서 Bob 대신 player 입력을 추가하도록 수정하자.

```go
//server.go
func (p *PlayerServer) processWin(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")
	p.store.RecordWin(player)
	w.WriteHeader(http.StatusAccepted)
}
```

### [20-6] 리팩토링

```go
//server.go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	switch r.Method {
	case http.MethodPost:
		p.processWin(w, r, player)
	case http.MethodGet:
		p.showScore(w, r, player)
	}
}

func (p *PlayerServer) showScore(w http.ResponseWriter, r *http.Request, player string) {
	score := p.store.GetPlayerScore(player)

	if score == 0 {
		w.WriteHeader(http.StatusNotFound)
	}

	fmt.Fprint(w, score)
}

func (p *PlayerServer) processWin(w http.ResponseWriter, r *http.Request, player string) {
	p.store.RecordWin(player)
	w.WriteHeader(http.StatusAccepted)
}
```
- player 입력을 받도록 하고 반복을 제거하였다.

## [20-7] 테스트 통합
- 현재 테스트 하는 PlayerStore는 main이 아닌, 테스트 코드에 구현되어 있다.
- 프로그램은 `main.go`로 실행 될 것이므로, 실제 사용할 프로그램을 테스트는 하지 않은 셈이다.
- 테스트가 main 코드에도 적용될 수 있게 하자.

### [20-7] 테스트, 컴파일 통과, 에러 메세지 확인
- `main.go`의 InMemoryStore를 사용하는 테스트 코드 작성.

```go
//server_integration_test.go
package main

import (
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestRecordingWinsAndRetrievingThem(t *testing.T) {
	store := InMemoryPlayerStore{}
	server := PlayerServer{&store}
	player := "Pepper"

	server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
	server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
	server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))

	response := httptest.NewRecorder()
	server.ServeHTTP(response, newGetScoreRequest(player))
	assertStatus(t, response.Code, http.StatusOK)

	assertResponseBody(t, response.Body.String(), "3")
}
```
```shell
$ go test
--- FAIL: TestRecordingWinsAndRetrievingThem (0.00s)
    server_integration_test.go:22: got '"123"' want '"3"'
FAIL
exit status 1
FAIL    server  0.003s
```
- main에는 메모리가 구현되어 있지 않고, "123"을 일률적으로 출력한다. 수정해서 테스트를 통과시키자.
- 테스트 통과에 앞서 `InMemoryPlayerStore`를 `main.go`에서 분리시키자.

### [20-7] 테스트 통과
```go
//in_memory_player_store.go
package main

func NewInMemoryPlayerStore() *InMemoryPlayerStore {
	return &InMemoryPlayerStore{map[string]int{}}
}

type InMemoryPlayerStore struct {
	store map[string]int
}

func (i *InMemoryPlayerStore) RecordWin(name string) {
	i.store[name]++
}

func (i *InMemoryPlayerStore) GetPlayerScore(name string) int {
	return i.store[name]
}
```
- `main.go`와 겹치므로 컴파일 에러가 발생할 것이다.

```go
//main.go
package main

import (
	"log"
	"net/http"
)

func main() {
	server := &PlayerServer{&InMemoryPlayerStore{}}
	log.Fatal(http.ListenAndServe(":5000", server))
}
```
- `InMemoryPlayerStore` 선언 부분을 main에서 제거하고, 테스트는 통과된다.
- 빌드하고 서버를 실행해서 결과를 보자.
![](https://i.imgur.com/GwWZ7ny.png)
- 3번의 POST 요청 이후에 3을 출력하는 것을 확인할 수 있다. 정상 동작한다.

### [20-7] 리팩토링
- 뮤텍스를 적용해서 동시동작 할 수 있게 해준다.

```go
//in_memory_player_server.go
package main

import "sync"

func NewInMemoryPlayerStore() *InMemoryPlayerStore {
	return &InMemoryPlayerStore{
		map[string]int{},
		sync.RWMutex{},
	}
}

type InMemoryPlayerStore struct {
	store map[string]int
	lock  sync.RWMutex
}

func (i *InMemoryPlayerStore) RecordWin(name string) {
	i.lock.Lock()
	defer i.lock.Unlock()
	i.store[name]++

}

func (i *InMemoryPlayerStore) GetPlayerScore(name string) int {
	i.lock.RLock()
	defer i.lock.RUnlock()
	return i.store[name]
}
```

## 요약
- Test-driven 서버 개발을 위해서 서버, 메모리를 구현하지 않고 request를 수행하는 함수를 먼저 테스트 했다.
    - `"net/http/httptest"`로 테스트용 request, response를 만들어서 서버 없이도 API 테스트를 수행할 수 있었다.
    - 메모리를 구현하지 않고, 고정된 값을 출력하게 함수를 만들어서 테스트 했다.
- 인터페이스로 구체적인 동작 `GetPlayerScore`, `RecordWin`은 더 상위 개념인 서버 관련 기능(`server.go`)과 분리했다.
    - `server.go`의 인터페이스를 따르는 `Store`는 테스트 코드(`server_test.go`) 내에서 우선 구현해서 테스트 하고,
    - 프로그램에서 실행 될 실제 함수는 `in_memory_player_store.go`에 구현하고, `server_integration_test.go`에서 테스트 했다.
-> 테스트 과정이 두 번인데, 한 번에 수행할 수 있지 않았을까? 하는 생각이 든다.

-----

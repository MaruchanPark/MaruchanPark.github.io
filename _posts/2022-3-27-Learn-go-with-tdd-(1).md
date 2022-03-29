---
layout: post
title: Learn go with tdd (1)
excerpt: Hello, World
date: 2022-3-27 15:00
tags: [Go, TDD]
use_math: true
--- 

Go를 기초부터 배우기 적합한 교재를 찾아서 TDD도 익힐 겸, Go도 익힐겸 겸사겸사 공부하고 정리하려고 한다. TDD 개발은 매우 효율적이라서 남아도는 시간을 어떻게 써야할지 고민을 해야 한다고 한다!! 어서 해봐야겠다!!!  
교재 : https://quii.gitbook.io/learn-go-with-tests/

hello.go
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World")
}
```
- main 함수에서 fmt 패키지의 Println 함수로 Hello, World를 출력한다.

hello.go
```go
package main

import "fmt"

func Hello() string {
	return "Hello, World"
}

func main() {
	fmt.Println(Hello())
}
```
- domain(Hello, World)과 side-effect(Println)를 분리한다.

hello_test.go
```go
package main

import "testing"

func TestHello(t *testing.T) {
	got := Hello()
	want := "Hello, World"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```
- hello.go가 위치한 디렉토리에 테스트 코드 hello_test.go 작성

테스트 수행을 위해 `go test`를 터미널에 입력하면, 모듈 에러가 발생한다. 구버전을 사용할 경우 통과하는 경우도 있다고 한다. `go mod init hello`로 go.mod 파일을 생성 한 후 테스트를 진행 할 수 있다.  
test 코드 작성에는 몇 가지 규칙이 있다.
- 테스트 코드 파일명 형식 : xxx_test.go
- 테스트 함수는 Test로 시작한다. (ex: `func TestHello`)
- 테스트 함수는 t *testing.T 1개 만을 인자로 받는다.
- "testing" 패키지 import  

![image](https://user-images.githubusercontent.com/48475993/160267366-5b7c1ef9-46d7-4bc3-9ca4-299901b8a5ff.png)

위 예제에서는 함수를 먼저 작성하고 테스트 코드를 작성했다면, 이제부터는 테스트를 먼저 작성하고, 함수를 작성한다.  

hello_test.go
```go
package main

import "testing"

func TestHello(t *testing.T) {
	got := Hello("Maru")
	want := "Hello, Maru"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```
- Hello 함수가 인자를 받는 방식으로 테스트 코드를 변경하였다.
- 현재 구현은 인자가 없기 때문에 테스트 시에 컴파일 에러를 예측할 수 있다.  

![image](https://user-images.githubusercontent.com/48475993/160267526-1d48452c-fff1-4b4c-83ef-d7beba5956da.png)

hello.go
```go
func Hello(name string) string {
	return "Hello, World"
}

func main() {
	fmt.Println(Hello("World"))
}
```
- 입력 인자만 추가해주고 테스트를 실행되게 하자.  

```bash
$ go test
--- FAIL: TestHello (0.00s)
    hello_test.go:10: got "Hello, World" want "Hello, Maru"
FAIL
exit status 1
FAIL    hello   0.001s
```

```go
func Hello(name string) string {
	return "Hello, " + name
}
```
- 테스트를 통과하게 코드 수정  

```bash
$ go test
PASS
ok      hello   0.001s
```

```go
```

```go
```

### Go doc
`sudo apt-get install golang-golang-x-tools`를 터미널에 입력, godoc을 설치하고 `godoc -http :8000`을 실행, 브라우저로 localhost:8000/pkg에 접속하면 사용중인 시스템에 설치된 패키지에 대한 설명을 볼 수 있다.  
`$ go install golang.org/x/tools/cmd/godoc@latest` 이렇게도 설치 할 수 있다고 한다.

-----

[^fn-sample_footnote]: Handy! Now click the return link to go back.
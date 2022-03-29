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
코드 : https://github.com/MaruchanPark/Learn_go_with_tests/tree/main/fundamentals/HelloWolrd

#### **`hello.go`**
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World")
}
```
- main 함수에서 fmt 패키지의 Println 함수로 Hello, World를 출력한다.

#### **`hello.go`**
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

#### **`hello_test.go`**
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

#### **`hello_test.go`**
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

#### **`hello.go`**
```go
func Hello(name string) string {
	return "Hello, World"
}

func main() {
	fmt.Println(Hello("World"))
}
```
- 입력 인자만 추가해주고 테스트를 실행되게 하자.  

```shell
$ go test
--- FAIL: TestHello (0.00s)
    hello_test.go:10: got "Hello, World" want "Hello, Maru"
FAIL
exit status 1
FAIL    hello   0.001s
```

#### **`hello.go`**
```go
func Hello(name string) string {
	return "Hello, " + name
}
```
- 테스트를 통과하게 코드 수정  

```shell
$ go test
PASS
ok      hello   0.001s
```

#### **`hello.go`**
```go
const englishHelloPrefix = "Hello, "

func Hello(name string) string {
	return englishHelloPrefix + name
}
```
- 상수를 지정해준다. 매번 Hello 함수를 통해 "Hello, "를 새로 불러오는 것 보다 더 빠르다고 한다. 간단한 예제라서 차이가 크진 않겠지만, 상수 변수에 이름도 지정해줄 수 있고, 성능도 좋아진다니 여러모로 유용하겠다.
- 전역 변수로 지정된 상수를 함수가 사용한다. 좋은 방법일까?  

#### **`hello_test.go`**
```go
package main

import "testing"

func TestHello(t *testing.T) {
	t.Run("Saying hello to people", func(t *testing.T) {
		got := Hello("Maru")
		want := "Hello, Maru"

		if got != want {
			t.Errorf("got %q want %q", got, want)
		}
	})

	t.Run("Saying hello to world when an empty string is supplied", func(t *testing.T) {
		got := Hello("")
		want := "Hello, World"

		if got != want {
			t.Errorf("got %q want %q", got, want)
		}
	})
}
```
- subtests 도입. 함수의 인자가 없는 경우도 테스트에 추가.
- 반복되는 코드를 함수로 만들자.  

#### **`hello_test.go`**
```go
func TestHello(t *testing.T) {

	assertCorrectMessage := func(t testing.TB, got, want string) {
		t.Helper()
		if got != want {
			t.Errorf("got %q want %q", got, want)
		}
	}

	t.Run("Saying hello to people", func(t *testing.T) {
		got := Hello("Maru")
		want := "Hello, Maru"

		assertCorrectMessage(t, got, want)
	})

	t.Run("Saying hello to world when an empty string is supplied", func(t *testing.T) {
		got := Hello("")
		want := "Hello, World"

		assertCorrectMessage(t, got, want)
	})
}
```
- 함수 내에서 함수를 선언하고 사용 할 수 있다.
- t.Helper()는 어떤 subtest에서 실패했는지 에러 라인을 명확히 알려준다.

t.Helper() 코멘트 처리 후 테스트  
```shell
hello_test.go:10: got "Hello, " want "Hello, World"
```

t.Helper() 코멘트 해제 후 테스트  
```shell
hello_test.go:25: got "Hello, " want "Hello, World"
```
- 이제 테스트를 통과하게 함수를 고치자.

#### **`hello.go`**
```go
func Hello(name string) string {
	if name == "" {
		name = "World"
	}
	return englishHelloPrefix + name
}
```

```shell
$ go test
PASS
ok      hello   0.002s
```
- 여기까지 하고 커밋!
- 지금까지 한 TDD 과정은 다음과 같다.
	1. 테스트 작성
	2. 컴파일 통과
	3. 테스트 실행, 에러 메세지가 적절한지 확인
	4. 테스트 통과하게 코드 작성
	5. 리팩토링
- 테스트 케이스를 미리 작성했으므로, 테스트가 간단하고, 리팩토링을 하고 테스트를 다시 하기 때문에 코드 기능이 변하지 않음을 보장해준다.

#### **`hello_test.go`**
```go
t.Run("in spanish", func(t *testing.T) {
	got := Hello("Elodie", "Spanish")
	want := "Hola, Elodie"
	assertCorrectMessage(t, got, want)
})
```
- 함수의 인자로 언어를 반영하는 테스트 추가
- 함수 작성 전에 테스트 케이스 먼저 작성한다. 당연히, 컴파일 에러가 발생한다.

```shell
$ go test
# hello [hello.test]
./hello_test.go:27:15: too many arguments in call to Hello
        have (string, string)
        want (string)
FAIL    hello [build failed]
```

#### **`hello.go`**
```go
func Hello(name string, language string) string {
```
- Hello 함수에 언어 인자를 받게 수정해준다.
- 다시 테스트 해보자

```shell
$ go test
# hello [hello.test]
./hello.go:15:19: not enough arguments in call to Hello
        have (string)
        want (string, string)
./hello_test.go:15:15: not enough arguments in call to Hello
        have (string)
        want (string, string)
./hello_test.go:21:15: not enough arguments in call to Hello
        have (string)
        want (string, string)
FAIL    hello [build failed]
```
- 컴파일 에러가 다시 발생했다.
- 기존에 작성해두었던 테스트 케이스 2개, hello.go의 main 
함수에서 인자가 부족한 호출로 인한 에러, 총 3개가 발생했다.
- TDD가 아니었으면, 앞의 2가지 테스트 케이스는 그냥 넘어갈 수도 있었다.
- 인자로 ""를 추가해주어 컴파일을 가능하게 해보자.

#### **`hello.go`**
```go
func main() {
	fmt.Println(Hello("World", ""))
}
```
#### **`hello_test.go`**
```go
t.Run("Saying hello to people", func(t *testing.T) {
	got := Hello("Maru", "")
	want := "Hello, Maru"
	assertCorrectMessage(t, got, want)
})

t.Run("Saying hello to world when an empty string is supplied", func(t *testing.T) {
	got := Hello("", "")
	want := "Hello, World"
	assertCorrectMessage(t, got, want)
})
```

```shell
$ go test
--- FAIL: TestHello (0.00s)
    --- FAIL: TestHello/in_spanish (0.00s)
        hello_test.go:29: got "Hello, Elodie" want "Hola, Elodie"
FAIL
exit status 1
FAIL    hello   0.001s
```
- Spanish 인데 Hello 인삿말을 하니 잘못된 게 맞다.
- 이제 Spanish에 대해서 제대로 동작하게 코드를 수정하자.

#### **`hello.go`**
```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	if language == "Spanish" {
		return "Hola, " + name
	}

	return englishHelloPrefix + name
}
```

```shell
$ go test
PASS
ok      hello   0.002s
```
- 통과! 명쾌해서 참 좋다.


#### **`hello.go`**
```go
const spanish = "Spanish"
const englishHelloPrefix = "Hello, "
const spanishHelloPrefix = "Hola, "

func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	if language == spanish {
		return spanishHelloPrefix + name
	}

	return englishHelloPrefix + name
}
```
- 테스트 케이스를 통과하는 코드를 작성했으니, 리팩토링을 한다.
- 호출 할 때마다 인삿말, 언어 string을 불러오지 않게, const로 선언해둔다.
- 리팩토링을 하고 테스트를 잊지 않는다. 변한게 없는지 이전 테스트들 모두에 적용되는 코드인지 항상 확인한다.

```shell
$ go test
PASS
ok      hello   0.002s
```

#### **`hello_test.go`**
```go
t.Run("in french", func(t *testing.T) {
	got := Hello("Le monde", "French")
	want := "Bonjour, Le monde"
	assertCorrectMessage(t, got, want)
})
```
- 불어에 대해서도 추가하기 위해 테스트 케이스 먼저 작성.
- Hello, world 구글 번역으로 돌려서 찾아봄...

```shell
$ go test
--- FAIL: TestHello (0.00s)
    --- FAIL: TestHello/in_french (0.00s)
        hello_test.go:35: got "Hello, Le monde" want "Bonjour, Le monde"
FAIL
exit status 1
FAIL    hello   0.001s
```
- 적절한 에러 메시지. 테스트를 통과하게 코드를 수정하자.


#### **`hello.go`**
```go
if language == "French" {
	return "Bonjour, " + name
}
```
- Hello 함수에 French에 해당하는 if문 추가.
- 간단한 코드지만, 리팩토링까지 한 번에 하지 않았다.
- 단계! 별로! 하는 것이다.

```shell
$ go test
PASS
ok      hello   0.002s
```
- 통과 했으니, 이제 리팩토링

#### **`hello.go`**
```go
const spanish = "Spanish"
const french = "French"
const englishHelloPrefix = "Hello, "
const spanishHelloPrefix = "Hola, "
const frenchHelloPrefix = "Bonjour, "

func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	if language == spanish {
		return spanishHelloPrefix + name
	}

	if language == french {
		return frenchHelloPrefix + name
	}

	return englishHelloPrefix + name
}
```
- 리팩토링 적용 이후 테스트

```shell
$ go test
PASS
ok      hello   0.002s
```
- if 문이 많은데, switch 문으로 바꾸자.
- switch 문은 break을 추가한 if 문으로 보면 되겠다.

#### **`hello.go`**
```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	prefix := englishHelloPrefix

	switch language {
	case french:
		prefix = frenchHelloPrefix
	case spanish:
		prefix = spanishHelloPrefix
	}

	return prefix + name
}
```
- 테스트를 수행하면 통과하는 것을 확인할 수 있다.

#### **`hello.go`**
```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	return greetingPrefix(language) + name
}

func greetingPrefix(language string) (prefix string) {
	switch language {
	case french:
		prefix = frenchHelloPrefix
	case spanish:
		prefix = spanishHelloPrefix
	default:
		prefix = englishHelloPrefix
	}
	return
}
```
- Hello 함수가 여러 일을 수행하고 있으므로, 분리하면 더 읽기 쉬운 코드가 된다.
- greetingPrefix 함수에 리턴 형식을 `(prefix string)`으로 지정해주었다. 
- prefix를 만들지 않고 값만 할당했으며, return 뒤에도 prefix를 명시하지 않았음에도 의도대로 동작하는 것을 알 수 있다. (초기값은 타입마다 다르다)
- switch 문에서 default는 case에서 해당되는 경우가 없을 때 실행된다.
- public 함수 (Hello)는 대문자로 시작, private 함수는 (greetingPrefix) 소문자로 시작한다. greetingPrefix는 코드 밖에서 실행 될 일이 없어보이므로 private으로 작성.
- 리팩토링 후에는 테스트를 수행한다.

```shell
$ go test
PASS
ok      hello   0.002s
```

무엇을 테스트 할지 정하고, 테스트를 통과하기 위한 최소한의 수정을 하고, 리팩토링 한다. 각 단계는 분리하여 수행하고, 항상 단계마다 테스트를 수행한다!!



## 기타:
### Go doc
`sudo apt-get install golang-golang-x-tools`를 터미널에 입력, godoc을 설치하고 `godoc -http :8000`을 실행, 브라우저로 localhost:8000/pkg에 접속하면 사용중인 시스템에 설치된 패키지에 대한 설명을 볼 수 있다.  
`$ go install golang.org/x/tools/cmd/godoc@latest` 이렇게도 설치 할 수 있다고 한다.

-----

[^fn-sample_footnote]: Handy! Now click the return link to go back.
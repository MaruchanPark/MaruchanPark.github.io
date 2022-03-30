---
layout: post
title: Learn go with tdd (3)
excerpt: Go Fundamentals-Iteration
date: 2022-3-32 15:00
tags: [Go, TDD]
use_math: true
--- 

## Iteration
Go에는 반복을 위해 for 만을 사용한다. while, do, until 등은 존재하지 않는다. 테스트 케이스 먼저 작성하면서, 반복문을 알아보자.

### character 5개 작성 반복문 구현
1. 테스트
2. 컴파일 통과
3. 에러 메시지 확인
4. 테스트 통과
5. 리팩토링

#### 1. 테스트
#### **`repeat_test.go`**
```go
package iteration

import "testing"

func TestRepeat(t *testing.T) {
	got := Repeat("a")
	want := "aaaaa"

	if got != want {
		t.Errorf("want '%q' but got '%q'", want, got)
	}
}
```
- `Repeat` 함수를 작성하지 않았으므로 컴파일 에러 발생, 해결한다.

#### 2. 컴파일 통과
- 함수 작성

#### **`repeat.go`**
```go
package iteration

func Repeat(character string) string {
	return ""
}
```
- 컴파일은 통과하지만, 테스트는 통과하지 못한다.

#### 3. 에러 메세지 확인
```shell
$ go test
--- FAIL: TestRepeat (0.00s)
    repeat_test.go:10: expected '"aaaaa"' but got '""'
FAIL
exit status 1
FAIL    iteration       0.002s
```
- 에러 메세지에 문제는 없다.

#### 4. 테스트 통과
#### **`repeat.go`**
```go
package iteration

func Repeat(character string) (repeated string) {
	
	for i := 0; i < 5; i++ {
		repeated += character
	}

	return
}
```
```shell
$ go test
PASS
ok      iteration       0.002s
```
#### 5. 리팩토링
- 더 개선할 사항이 보이지 않으므로 생략

### 반복 수 지정 기능 추가
#### 1. 테스트 생성

#### **`repeat_test.go`**
```go
func assertCorrectMessage(t testing.TB, got, want string) {
	t.Helper()
	if got != want {
		t.Errorf("want '%q' but got '%q'", want, got)
	}
}

func TestRepeat(t *testing.T) {
	t.Run("five repetition", func(t *testing.T) {
		got := Repeat("a", 5)
		want := "aaaaa"

		assertCorrectMessage(t, got, want)
	})

	t.Run("seven repetition", func(t *testing.T) {
		got := Repeat("a", 7)
		want := "aaaaaaa"

		assertCorrectMessage(t, got, want)
	})
}
```
- 테스트 케이스를 추가하고, 반복을 줄여 테스트 코드를 작성하였다.
- 함수는 인자를 1개만 받는 상태, 컴파일 에러가 발생한다.

#### 2. 컴파일 통과, 3. 에러 메세지 확인

#### **`repeat.go`**
```go
func Repeat(character string, repetition int) (repeated string) {
```
- 함수의 입력 인자를 추가해서 컴파일만 가능하게 변경

```shell
$ go test
--- FAIL: TestRepeat (0.00s)
    --- FAIL: TestRepeat/seven_repetition (0.00s)
        repeat_test.go:24: want '"aaaaaaa"' but got '"aaaaa"'
FAIL
exit status 1
FAIL    iteration       0.001s
```
- 새로운 테스트는 통과하지 못한다. 입력 반복수 만큼 반복하게 함수를 수정하자.

#### 4. 테스트 통과
#### **`repeat.go`**
```go
func Repeat(character string, repetition int) (repeated string) {

	for i := 0; i < repetition; i++ {
		repeated += character
	}

	return
}
```
- 모든 테스트 케이스를 통과한다.

#### 5. 리팩토링
- 함수의 이름을 Repeat -> RepeatCharacter
    - 입력 인자로 character가 있으니, 무엇을 반복하는지 명확해짐.
    - 설명 주석 생략 가능
- 반복수 인자의 이름을 repetition -> numberOfRepetition으로 변경
- 좀 길기는 하지만, 의미가 확실해서 더 좋을 것 같다.
- 이름을 바꾸고 테스트를 해보면, 당연히 통과한다.

#### **`repeat.go`**
```go
package iteration

func RepeatCharacter(character string, numberOfRepetition int) (repeated string) {

	for i := 0; i < numberOfRepetition; i++ {
		repeated += character
	}

	return
}
```


### Example 추가
#### **`repeat_test.go`**
```go
func ExampleRepeatCharacter(t *testing.T) {
	got := RepeatCharacter("abc", 3)
	fmt.Println(got)
	//Output:abcabcabc
}
```


### Benchmarking
- 아래와 같이 벤치마크 수행을 추가할 수 있다.
- `go test -bench=.`으로 실행할 수 있다.
#### **`repeat_test.go`**
```go
func BenchmarkRepeat(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Repeat("a")
	}
}
```
- 결과  
```shell
$ go test -bench=.
goos: linux
goarch: amd64
pkg: iteration
cpu: AMD Ryzen 7 2700X Eight-Core Processor         
BenchmarkRepeat-16       7207819               157.7 ns/op
PASS
ok      iteration       1.309s
```





#### **`repeat.go`**
```go
```

#### **`repeat_test.go`**
```go
```

```shell
$ go test
```

-----
---
layout: post
title: Learn go with tdd (2)
excerpt: Go Fundamentals (2.Integers, 3.Iteration, 4.Arrays and slices)
date: 2022-3-29 12:00
tags: [Go, TDD]
use_math: true
--- 

## Chapter2: Integers
- TDD 과정을 복기하자.  
	1. 테스트 작성
	2. 컴파일 통과 시키기
	3. 에러 메시지가 적절한지 확인
	4. 테스트를 통과하는 코드 작성
	5. 리팩토링

- 이번 챕터에서 구현할 코드를 TDD 단계를 반복적으로 거치면서 완성해볼 것이다.
- [교재](https://quii.gitbook.io/learn-go-with-tests/go-fundamentals/integers)  
- [코드](https://github.com/MaruchanPark/Learn_go_with_tests/tree/main/fundamentals/2_Integers)  

### 인자 2개를 더하는 함수의 테스트 작성
#### **`adder_test.go`**
```go
package integers

import "testing"

func TestAdder(t *testing.T) {
	sum := Add(2, 2)
	expected := 4

	if sum != expected {
		t.Errorf("expected '%d' but got '%d'", expected, sum)
	}
}
```
- test 수행 시에 모듈 에러 부터 뜬다.
- `go mod init integers`로 go.mod 생성
- Add 함수가 없으므로 컴파일 에러가 발생한다.
- 테스트 코드의 이름으로 함수가 위치할 파일명은 정해졌다. `adder.go`에 함수를 작성하자.

### 테스트를 통과하는 코드 작성
#### **`adder.go`**
```go
package integers

func Add(x, y int) int {
	return x + y
}
```
- 이제 main을 작성하지 않는다!
- 예시가 너무 간단해서 굳이 2, 3, 4 과정을 거치기 보다는 한 번에 4번 단계로 넘어가는 것을 선택했다.
```shell
$ go test
PASS
ok      integers        0.002s
```

### 리팩토링
- Example 작성

#### **`adder.go`**
```go
package integers

// Add takes two integers and returns the sum of them.
func Add(x, y int) int {
	return x + y
}
```
- 함수 상단에 설명을 작성하면 다른 코드에서 함수를 사용할 때, 에디터로 설명을 볼 수 있다고 한다. (지금 내 환경에서는 안 되는 것 같다..)

#### 함수의 Example
- Example을 주석으로, 외부 문서로 작성하는 경우가 있는데, 코드를 수정하고 문서를 수정하지 않으면 일치하지 않는 예시가 남게 된다.
- test 코드 내에 Example 함수를 작성하는 것으로 장점만 누릴 수 있다고 한다.

#### **`adder_test.go`**
```go
package integers

import (
	"fmt"
	"testing"
)

func TestAdder(t *testing.T) {
	sum := Add(2, 2)
	expected := 4

	if sum != expected {
		t.Errorf("expected '%d' but got '%d'", expected, sum)
	}
}

func ExampleAdd() {
	sum := Add(1, 5)
	fmt.Println(sum)
	//Output: 6
}
```
- `//Output: 6` 주석을 지우면 Example 함수는 동작하지 않는다.
- 추가한 Example은 Godoc에서 확인할 수 있다.

```shell
$ godoc -http:8000
using module mode; GOMOD=/home/maru/EXT/2022/go_tdd/fundamentals/2_Integers/go.mod
```

#### 브라우저로 확인을 하니 아래와 같이 나온다. 신기하다.
![image](https://user-images.githubusercontent.com/48475993/160756042-577a233f-7e53-4927-83e4-62a541a343d6.png)

- 이제 코드 수정 후에, 유효하지 않은 예제는 등록되지 않는다. 잘못된 예제를 제공할 위험은 사라졌다!


## Chapter3: Iteration
Go에는 반복을 위해 for 만을 사용한다. while, do, until 등은 존재하지 않는다. 테스트 케이스 먼저 작성하면서, 반복문을 알아보자.

[교재](https://quii.gitbook.io/learn-go-with-tests/go-fundamentals/iteration)  
[코드](https://github.com/MaruchanPark/Learn_go_with_tests/tree/main/fundamentals/3_Iteration) 

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

## Chapter4: Arrays and slices
- array에는 여러 값을 저장할 수 있다.
- array의 값을 합하는 함수를 구현해보자.

[교재](https://quii.gitbook.io/learn-go-with-tests/go-fundamentals/arrays-and-slices)  
[코드](https://github.com/MaruchanPark/Learn_go_with_tests/tree/main/fundamentals/4_Arrays_and_slices) 

### 4-1. sum 함수 구현
##### [4-1] 테스트

```go
//sum_test.go
package main

import "testing"

func TestSum(t *testing.T) {

	numbers := [5]int{1, 2, 3, 4, 5}

	got := Sum(numbers)
	want := 15

	if got != want {
		t.Errorf("got '%d' want '%d' given, %v", got, want, numbers)
	}
}
```
- 에러 메세지에 입력도 출력하게 해주었다. `%v`는 default format이라고 한다. 뭐가 default 인가 싶지만... array에 잘 동작한다니 넘어가도록 하자.

```shell
$ go mod init main
$ go test
# main.test
/tmp/go-build2428740758/b001/_testmain.go:13:2: cannot import "main"
/tmp/go-build2428740758/b001/_testmain.go:21:14: undefined: _test
FAIL    main [build failed]
```
- main 단독으로는 unit-testable 하지 않다고 한다. 무슨 말인지는 잘 모르겠지만, `go.mod` 파일의 package를 `sum`으로 수정해주니 테스트는 진행할 수 있었다.

```go
//go.mod
package sum

go 1.17
```
```shell
$ go test
# sum [sum.test]
./sum_test.go:9:9: undefined: Sum
FAIL    sum [build failed]
```

##### [4-1] 컴파일 통과, 에러 메세지 확인

```go
//sum.go
package main

func Sum(numbers [5]int) int {
	return 0
}
```
```shell
$ go test
--- FAIL: TestSum (0.00s)
    sum_test.go:13: got '0' want '15' given, [1 2 3 4 5]
FAIL
exit status 1
FAIL    sum     0.001s
```
- 예상되는 에러를 확인했다. 통과하는 코드를 작성하자.

##### [4-1] 테스트 통과
```go
//sum.go
package main

func Sum(numbers [5]int) (sum int) {
	for i := 0; i < 5; i++ {
		sum += numbers[i]
	}
	return
}
```

##### [4-1] 리팩토링
```go
//sum.go
package main

func Sum(numbers [5]int) (sum int) {
	for _, number := range numbers {
		sum += number
	}
	return
}
```
- range로 array안의 인덱스, 값 쌍을 순서대로 뽑을 수 있다.
- 인덱스는 `_`(blank identifier)로 받지 않게 처리하였다.

### 4-2. 다양한 크기의 array에 대해 적용되게 구현하기
- array 타입을 `[5]int`로 지정하였다. 다른 타입인 `[6]int`에 대해서는 동작하지 않을 것이다.
- Go에서는 크기가 고정되지 않은 array를 slice라고 한다.
- slice에 대한 테스트 작성으로 구현을 시작하자.

##### [4-2] 테스트
- 함수의 입력 타입이 slice인 다른 케이스를 추가한다.

```go
//sum_test.go
func TestSum(t *testing.T) {

	t.Run("collection of 5 numbers", func(t *testing.T) {
		numbers := [5]int{1, 2, 3, 4, 5}

		got := Sum(numbers)
		want := 15

		if got != want {
			t.Errorf("got '%d' want '%d' given, %v", got, want, numbers)
		}
	})

	t.Run("collection of any size", func(t *testing.T) {
		numbers := []int{1, 2, 3}

		got := Sum(numbers)
		want := 6

		if got != want {
			t.Errorf("got '%d' want '%d' give, %v", got, want, numbers)
		}
	})
}
```
- Sum 함수는 [5]int 타입 입력을 받도록 되어있다. 컴파일 에러가 발생한다.

##### [4-2] 컴파일 통과, 에러 메세지 확인, 테스트 통과

```go
//sum.go
func Sum(numbers []int) (sum int) {
```
- 입력을 slice로 변경
- 여전히 컴파일 되지 않는데, 테스트 코드의 [5]int 입력 때문이니 slice로 변경해준다.

```go
//sum_test.go
func TestSum(t *testing.T) {

	t.Run("collection of 5 numbers", func(t *testing.T) {
		numbers := []int{1, 2, 3, 4, 5}
```
- 테스트가 통과한다.

##### [4-2] 리팩토링
- 2개의 테스트 케이스까지는 필요하지 않아보임.
- `go test -cover`로 coverage를 확인한다.
```shell
$ go test -cover
PASS
coverage: 100.0% of statements
ok      sum     0.001s
```
- 테스트 케이스를 지워보자.
```go
//sum_test.go
func TestSum(t *testing.T) {
	
	t.Run("collection of any size", func(t *testing.T) {
		numbers := []int{1, 2, 3}

		got := Sum(numbers)
		want := 6

		if got != want {
			t.Errorf("got '%d' want '%d' give, %v", got, want, numbers)
		}
	})
}
```
- 다시 coverage를 확인한다.
```shell
$ go test -cover
PASS
coverage: 100.0% of statements
ok      sum     0.001s
```

### [4-3] SumAll 구현
##### [4-3] 테스트
- 기존 테스트 코드에 새로운 함수의 테스트를 구현한다.
```go
//sum_test.go
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if got != want {
		t.Errorf("got %v want %v", got, want)
	}
}
```

##### [4-3] 컴파일 통과, 에러 메세지 확인
- 여러 개의 slice를 받을 수 있는 `SumAll` 함수를 정의하고 컴파일을 통과하자.
```go
//sum.go
func SumAll(numbersToSum ...[]int) (sums []int) {
	return
}
```
```shell
$ go test
# sum [sum.test]
./sum_test.go:24:9: invalid operation: got != want (slice can only be compared to nil)
FAIL    sum [build failed]
```
- 여전히 컴파일 되지 않는다.
- slice를 비교하기 위해서는 iterate 하게 비교하는 함수를 직접 작성할 필요가 있어 보인다.
- 당장은 편의를 위해 `reflect.DeepEqual`을 사용하자.
```go
//sum_test.go
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```
- 다시 테스트 해보자.
```shell
$ go test
--- FAIL: TestSumAll (0.00s)
    sum_test.go:28: got [] want [3 9]
FAIL
exit status 1
FAIL    sum     0.001s
```
- 정상적으로 에러 메세지가 출력되는 것을 확인할 수 있다.
- `DeepEqual`은 "type safe" 하지 않은 함수로, 사용에 주의가 필요하다.
- 예를 들어, 다음과 같은 `string`, `slice`의 비교도 컴파일 된다.
```go
//sum_test.go
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := "bob"

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```
```shell
$ go test
--- FAIL: TestSumAll (0.00s)
    sum_test.go:28: got [] want bob
FAIL
exit status 1
FAIL    sum     0.001s
```
- 비교할 수 없는 타입에 대해서도 컴파일 되는 것을 확인할 수 있다.
- 앞으로 사용에 주의하도록 하고, 테스트를 원래대로 돌리고 하던 것을 계속 하자.
##### [4-3] 테스트 통과
- 입력 slice들(`numbersToSum`)의 합을 담을 slice(`sums`)를 만들고, `Sum` 함수로 결과를 구한다.
```go
//sum.go
func SumAll(numbersToSum ...[]int) (sums []int) {
	lengthOfNumbers := len(numbersToSum)
	sums = make([]int, lengthOfNumbers)

	for i, numbers := range numbersToSum {
		sums[i] = Sum(numbers)
	}

	return
}
```

##### [4-3] 리팩토링
- `append`를 활용해서 slice의 크기를 고려하지 않아도 되도록 만들자.
```go
//sum.go
func SumAll(numbersToSum ...[]int) (sums []int) {

	for _, numbers := range numbersToSum {
		sums = append(sums, Sum(numbers))
	}

	return
}
```
- [4-4]에서는 `SumAll`을 `SumAllTail`(summation 대상에서 첫 번째 성분을 제외)로 수정 해보자.

### [4-4] SumAllTail 구현

##### [4-4] 테스트
- TestSumAllTails 작성
```go
//sum_test.go
func TestSumAllTails(t *testing.T) {

	got := SumAllTails([]int{1, 2}, []int{0, 9})
	want := []int{2, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

##### [4-4] 컴파일 통과, 에러 메세지 확인
- 함수가 구현되어 있지 않으므로, 컴파일 에러가 발생한다.
- 컴파일을 통과하기 위한 최소한의 수정으로 함수명을 `SumAllTails`로 변경하고 에러 메세지를 확인한다.
```go
//sum.go
func SumAllTails(numbersToSum ...[]int) (sums []int) {
```
- 컴파일 통과, 에러 메세지 확인

##### [4-4] 테스트 통과
- python과 같이 slicing이 가능하다.
- 첫 번째 성분만 빼고 `Sum`함수에 입력한다.
```go
//sum.go
func SumAllTails(numbersToSum ...[]int) (sums []int) {

	for _, numbers := range numbersToSum {
		tail := numbers[1:]
		sums = append(sums, Sum(tail))
	}

	return
}
```
- 테스트는 통과한다.

##### [4-4] 리팩토링
- 리팩토링 할 만한 것이 없다! 다음으로 넘어가자.

### [4-5] empty slice 처리
- empty slice에 대해 0으로 출력하게 처리하자.

##### [4-5] 테스트
- 테스트 먼저 구현
```go
//sum_test.go
func TestSumAllTails(t *testing.T) {

	t.Run("make the sums of some slices", func(t *testing.T) {
		got := SumAllTails([]int{1, 2}, []int{0, 9})
		want := []int{2, 9}
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	})

	t.Run("sum over empty slices", func(t *testing.T) {
		got := SumAllTails([]int{}, []int{3, 4, 5})
		want := []int{0, 9}
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	})
}
```
- 테스트 해보자.
```shell
$ go test
--- FAIL: TestSumAllTails (0.00s)
    --- FAIL: TestSumAllTails/sum_over_empty_slices (0.00s)
panic: runtime error: slice bounds out of range [1:0] [recovered]
        panic: runtime error: slice bounds out of range [1:0]
...
```
- 런타임 에러가 발생했다.
- 컴파일 되지 않게 만들 수는 없었을까?

##### [4-5] 컴파일 통과, 에러 메세지 확인
- 컴파일에는 문제가 없으므로 테스트 통과로 넘어가자.

##### [4-5] 테스트 통과
- 빈 slice 입력에 대한 예외처리를 해주자.
```go
//sum.go
func SumAllTails(numbersToSum ...[]int) (sums []int) {

	for _, numbers := range numbersToSum {
		if len(numbers) == 0 {
			sums = append(sums, 0)
		} else {
			tail := numbers[1:]
			sums = append(sums, Sum(tail))
		}
	}

	return
}
```
- 크기가 1인 slice 입력에 대해서는, `[1:]` slicing으로 빈 slice로 `Sum`에 입력된다.
- `Sum`은 빈 slice 입력에 대해 문제가 없는 것을 알 수 있다.

##### [4-5] 리팩토링
- 테스트 코드에 반복이 있으므로 리팩토링을 진행한다.
```go
//sum_test.go
func TestSumAllTails(t *testing.T) {

	checkSums := func(t testing.TB, got, want []int) {
		t.Helper()
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	}

	t.Run("make the sums of some slices", func(t *testing.T) {
		got := SumAllTails([]int{1, 2}, []int{0, 9})
		want := []int{2, 9}

		checkSums(t, got, want)
	})

	t.Run("sum over empty slices", func(t *testing.T) {
		got := SumAllTails([]int{}, []int{3, 4, 5})
		want := []int{0, 9}

		checkSums(t, got, want)
	})
}
```
- `checkSums`의 `got`, `want` 입력 타입을 slice로 지정해줌으로써 `reflect.DeepEqual`의 "type safey" 문제가 해결되었다!!!





-----

[^fn-sample_footnote]: Handy! Now click the return link to go back.
---
layout: post
title: Learn go with tdd (2)
excerpt: Go Fundamentals-Integers
date: 2022-3-27 15:00
tags: [Go, TDD]
use_math: true
--- 

## Chapter: Integers
이번 타입에서 하는 것:

TDD 단계를 복기하자.  
1. 테스트 작성
2. 컴파일 통과 시키기
3. 에러 메시지가 적절한지 확인
4. 테스트를 통과하는 코드 작성
5. 리팩토링

이번 챕터에서 구현할 코드를 TDD 단계를 반복적으로 거치면서 완성해볼 것이다.

[교재](https://quii.gitbook.io/learn-go-with-tests/go-fundamentals/integers)  
[코드](https://github.com/MaruchanPark/Learn_go_with_tests/tree/main/fundamentals/2_Integers)  

### 인자 2개를 더하는 함수의 테스트 작성
#### **`adder_test.go`**
```Go
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
```Go
package integers

func Add(x, y int) int {
	return x + y
}
```
- 이제 main을 작성하지 않는다!
- 예시가 너무 간단해서 굳이 2, 3, 4 과정을 거치기 보다는 한 번에 4번 단계로 넘어가는 것을 선택했다.

```Shell
$ go test
PASS
ok      integers        0.002s
```

### 리팩토링
- Example 작성

#### **`adder.go`**
```Go
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
```Go
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

```Shell
$ godoc -http:8000
using module mode; GOMOD=/home/maru/EXT/2022/go_tdd/fundamentals/2_Integers/go.mod
```

#### 브라우저로 확인을 하니 아래와 같이 나온다. 신기하다.
![image](https://user-images.githubusercontent.com/48475993/160756042-577a233f-7e53-4927-83e4-62a541a343d6.png)

- 이제 코드 수정 결과, 예제가 유효하지 않으면 테스트가 실패하게 된다. 함수 예제를 잊을까봐 염려하지 않아도 된다!

-----
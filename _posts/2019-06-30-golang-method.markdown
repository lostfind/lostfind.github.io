---
layout: post
title:  "Go 언어의 메소드와 함수"
date:   2019-06-30 11:28
author: lostfind
categories: Development
tags: Go Method
cover:  "/assets/instacode.png"
---

## 함수
함수는 단순히 파라미터 리턴값을 갖는다.

```go
func Meow(name string) {
	fmt.Printf("function: my name is %s. MEOW~\n", name)
}
```

## 메소드
메소드는 함수와 거의 동일하지만, 구조체에 붙여서 사용하며 구조체 이외에도
기본타입 이외의 어떠한 타입에든 붙여서 사용할 수 있다.
특정 구조체에 붙여 사용하기 위해서 메소드명 앞의 리시버에 구조체(혹은 타입명)를 명시한다.

```go
type Cat struct {
	name string
	age  int
}

// Cat 구조체의 메소드 Meow
func (c Cat) Meow() {
	fmt.Printf("my name is %s. MEOW~\n", c.name)
}

func main() {
	// neco *Cat
	neco := new(Cat)
	neco.name = "neco"
	neco.age = 0

	// nabi Cat
	nabi := Cat{name: "nabi", age: 1}

	neco.Meow() // my name is neco. MEOW~
	nabi.Meow() // my name is nabi. MEOW~
}
```

### 포인터 리시버
Go 언어는 일반적으로는 값을 전달하기 때문에 메모리상에서 변수가 복제되어 전달되지만
리시버를 포인터 리시버로 정의하면 불필요한 복제를 방지하며, Call by reference와 같이 동작한다.

아래의 코드와 같이 포인터 리시버를 사용한 메소드는 `Cat`구조체의 필드 `age`의 값을 직접 수정한다.

```go
// Cat 구조체의 메소드 AddAge (포인터 리시버)
func (c *Cat) AddAge() {
	c.age++ // 구조체의 필드를 직접 수정
}

func main() {
	// neco *Cat
	neco := new(Cat)
	neco.name = "neco"
	neco.age = 0

	// nabi Cat
	nabi := Cat{name: "nabi", age: 1}

	fmt.Println(neco.age) // 0
	fmt.Println(nabi.age) // 1

	neco.AddAge()
	nabi.AddAge()

	fmt.Println(neco.age) // 1
	fmt.Println(nabi.age) // 2
}
```

그렇다면, 포인터 리시버 메소드의 경우에는 구조체를 포인터형으로 생성해야 하는가?
위 코드의 경우 `neco`는 Go의 내장함수`new()`을 통해 생성했기 때문에 포인터형(*Cat)으로 생성되어 있고
`nabi`는 직접 생성했기 때문에 포인터가 아닌 객체(Cat)로 생성되었다.

하지만, 동일한 `AddAge()`메소드를 호출하면 `neco`, `nabi` 둘 다 직접 값이 수정되어 있다.
이는, 컴파일러가 암묵적으로 변환해 주기 때문에 가능한 것이다. [Go매뉴얼 Calls](https://golang.org/ref/spec#Calls)

명시적으로 포인터형의 메소드를 호출하는 것도 가능하며, 결과는 동일하다.
```go
	fmt.Println(neco.age) // 0
	fmt.Println(nabi.age) // 1

	neco.AddAge()
	(&nabi).AddAge() // nabi.AddAge()와 동일

	fmt.Println(neco.age) // 1
	fmt.Println(nabi.age) // 2
```

## 함수 vs 메소드
함수의 경우에는 프로그램 전체적으로 파라미터에 대해 일정한 처리를 하는 경우에 사용하고,
메소드는 객체별 기능을 구현할 때 사용하는 것이 좋다고 생각한다.

비슷한 기능이라도 구조체별로 다른 동작을 하는 경우에 메소드로 한다면
`hogeSave()`, `fugeSave()`이 아니라 `객체명.Save()`와 같이 이름을 짓기도 편하며
수정이 발생하는 경우에 어디까지 영향이 미치는지 범위를 파악하기도 쉬워진다.

특별한 경우가 아니라면 기본적으로 메소드로 구현하는 것이 나을 듯하다.

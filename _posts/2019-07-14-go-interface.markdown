---
layout: post
title:  "Go 언어의 인터페이스"
date:   2019-07-14 18:28
author: lostfind
categories: Development
tags: Go Interface
cover:  "/assets/instacode.png"
---

Go에서의 인터페이스는 메서드의 집합체로, 인터페이스를 만족하기 위해서는 위해서는 인터페이스가 갖고있는 모든 메소드를 구현 하기만 하면 된다.

단, Java의 `implements`와 같은 표기는 하지 않기 때문에 어떤 인터페이스를 구현 한건지 코드만 봤을땐 파악하기 힘들다는 단점이 있다.

## 예제
`Pet`이라는 구조체가 `Animal`과 `Cat` 두개의 인터페이스를 구현하는 예제이다.

### 인터페이스 정의
```go:lostfind/playground/zoo/animal.go
package zoo

type Animal interface {
	Walk()
}

type Cat interface {
	Cry()
}
```

### 인터페이스 구현
두 인터페이스가 정의 되어있는 `zoo`패키지를 import한 후, 각 메소드를 구현 한다.
```go:main.go
package main

import (
	"fmt"
	"lostfind/playground/zoo"
)

type Pet struct {
	name string
	age  int
}

// Cat의 인터페이스 구현
func (p *Pet) Cry() {
	fmt.Println("Nya")
}

// Animal 인터페이스의 구현
func (p *Pet) Walk() {
	fmt.Println("I walk with four feet.")
}

func main() {
	// 인터페이스 변수
	var animal zoo.Animal
	var cat zoo.Cat

	myPet := &Pet{
		name: "Nyanko",
		age:  10,
	}

	// myPet은 Cat, Animal의 인터페이스를 구현 하였으므로 각 인터페이스 변수에 대입 가능
	cat = myPet
	animal = myPet

	cat.Cry() // Nya
	animal.Walk() // I walk with four feet.
}
```

### Java에서의 표현
단순히 이해를 돕기위한 코드로 Java를 코딩 안한지 꽤 오랜 시간이 지나서 문법이 정확하지 않을 수 있다.

```java
public interface Animal {
  public void Walk();
}

public interface Cat {
  public void Cry();
}

class Pet implements Animal, Cat {
  public void Walk() {
    System.out.println("I walk with four feet.");
  }

  public void Cry() {
    System.out.println("Nya");
  }
}
```
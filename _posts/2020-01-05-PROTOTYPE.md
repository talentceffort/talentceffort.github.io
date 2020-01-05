---
title: '프로토타입에 대한 정리'

categories:
  - Blog
tags:
  - Blog
---

## 객체
```js
var obj = {}
var obj = new Object()
```
위 두가지 방식 모두 Object 타입을 갖는 객체로 Object 타입의 메소드인 `isPrototypeOf`, `toString`, `valueOf` 등을 사용할 수 있다. Object 는 모든 객체의 최상위 타입이다. 다른 객체지향언어의 관점에서 보면 위의 코드는 Object 객체의 인스턴스를 만든 것에 불과해 상속받았다고 표현하기 힘들다. 따라서 자바스크립트에서는 다른 관점으로 바라보아야 하며 Object 생성자의 프로토타입을 상속받은 객체라고 표현하는 것이 정확한 표현이다.

## 프로토타입
프로토타입을 이용하면 객체와 객체를 연결하고 한쪽 방향으로 상속을 받는 형태로 만들 수 있다. 자바스크립트에서 객체와 객체를 연결해서 상속 받는다는 것은 다른 말로 객체와 객체를 연결해 멤버 함수나 멤버 변수를 공유 한다는 것이다.

```js
var a = {
    attr1: 'a1'
}

var b = {
    attr2: 'a2'
}

b.__proto__ = a

b.attr1 // 'a1'

a.attr1 = 'a000' // 상속받은 객체의 내용 변경

b.attr1 // 'a000'

a.attr3 = 'a3' // 상속받은 객체의 내용이 추가
b.attr3 // 'a3'

delete a.attr1 // 상속받은 객체의 내용이 삭제
b.attr1 // undefined
```

a 와 b 객체가 존재할 때 b객체를 통해 a 객체의 `attr1` 에 접근 할 수 없다. b 에서 a 의 `attr1` 에 접근 하기 위해서는 `__proto__` 속성을 이용한다.
`__proto__` 속성은 ECMAScript 의 스펙 `[[Prototype]]`이 노출된 것이다. 크롬 개발자 도구에서 디버깅 할 때 노출되어 있는 걸 본 적 있을 것이다. 하지만 개발 할 때 `__proto__` 에 직접 접근하면 안되고 `Object.getPrototypeOf()` 를 사용해서 현재 `__proto__` 이 참조하는 객체를 알아 볼 수 있다. 예를 들어 현재 b 객체의 `__proto__` 를 알고 싶다면 `Object.getPrototypeOf(b)` 를 사용하면 된다. 객체와 객체의 연결을 통한 단방향 공유 관계를 **프로토타입 체인**이라고 한다.

## 프로토타입 룩업
 프로토타입 체인을 통해 객체의 메서스나 속성을 찾아가는 과정을 프로토타입 룩업이라고 한다.프로토타입 룩업이라는 과정은 클래스의 상속과 비교되는 특징 중 하나이다. 클래스 상속은 객체를 만든 시점에 이미 이 객체가 상속구조를 통해 어떤 멤버들을 보유하고 있는지 결정된 반면 프로토타입 체인을 통한 상속의 경우 실행을 해봐야 객체가 해당 멤버를 가지고 있는지 알 수 있다.그래서 이미 만들어진 객체의 상속된 내용이 변경될 수 있는 것이다. 객체가 만들어 질 때가 아닌 실행할 때의 내용이 중요해서 그렇다.

```js
var a = {
    attr1: 'a'
};

var b = {
    __proto__: a,
    attr2: 'b'
};

var c = {
    __proto__: b,
    attr3: 'c'
};

c.attr1 // 'a'
```
위의 코드에서는 객체를 세 개 만들고 각 객체의 `__proto__` 속성을 이용해 c -> b -> a 로 연결했다. `c.attr1` 로 c 객체에는 없는 `attr1`이라는 속성에 접근하면 자바스크립트 엔진은 아래와 같은 작업을 수행한다.

1. c 객체 내부에 `attr1` 속성을 찾는다. -> 없다.
2. c 객체에 `__proto__` 속성이 존재하는지 확인한다. -> 있다.
3. c 객체의 `__proto__` 속성이 참조하는 객체로 이동한다. -> b 객체로 이동
4. b 객체 내부에 `attr1` 속성을 찾는다. -> 없다.
5. b 객체에 `__proto__` 속성이 존재하는지 확인한다. -> 있다.
6. b 객체의 `__proto__` 속성이 참조하는 객체로 이동한다. -> a 객체로 이동
7. a 객체 내부에 `attr1` 속성을 찾는다. -> 있다.
8. 찾은 속성의 값을 리턴한다.

만약에 어떤 객체에도 존재하지 않는 `attr4` 를 검색한다면 7번 부터 다른 과정을 거치게 된다.

1. a 객체 내부에 `attr4` 속성을 찾는다. -> 없다.
2. a 객체에 `__proto__` 속성이 존재하는지 확인한다. -> 있다.
3. a 객체의 `__proto__` 속성이 참조하는 객체로 이동한다. -> `Object.prototype` 로 이동
4. `Object.prototype` 에서 `attr4` 속성을 찾는다. -> 없다.
5. `Object.prototype` 에서 `__proto__` 속성을 찾는다. -> 없다.
6. `undefiend` 를 리턴한다.

모든 프로토타입 체인의 끝은 항상 `Object.prototype` 이다. 그래서 `Object.prototype` 은 `__proto__` 속성이 없다. `attr4` 라는 속성은 프로토타입의 마지막 단계인 `Object.prototype` 에 존재하지 않고 `Object.prototype` 에는 `__proto__` 속성이 존재하지 않으니 탐색을 종료하고 `undefined` 를 리턴한다

## 생성자
생성자를 이용해 객체를 생성하면 생성된 객체는 생성자의 프로토타입 객체와 프로토타입 체인으로 연결된다.

```js
//constructor
function Parent(name) {
    this.name = name
}

Parent.prototype.getName = function() {
    return this.name;
}

var p = new Parent('myName')
```
`Parent` 라는 생성자가 만들어내는 객체는 p 는 name 이라는 속성을 가지고 있고 getName 이라는 프로토타입 메서드를 사용할 수 있다. p 객체가 `Parent` 의 프로토타입 메서드에 접근할 수 있는 이유는 p 객체의 `__proto__` 속성이 `Parent.prototype` 을 가리키고 있기 때문이다.

```js
var p = new Parent('myName')

// 엔진 내부에서 하는 일
p = {} // 새로운 객체 생성
Parent.call(p, 'myName') // call 을 이용해 Parent함수의 this를 p 로 대신해서 실행
p.__proto__ = Parent.prototype; // 프로토타입을 연결

p.getName(); // 'myName'
```
위 코드는 생성자가 만들어 내는 객체가 어떻게 생성자의 `Prototype` 과 연결되는지 보여주고 있다. 위 과정을 통해서 프로토타입 룩업시 p -> `Parent.prototype` 의 탐색 과정을 만들어 낼 수 있다.

## Object.create()

```js
var a = {
    attr1: 'a'
};

var b = {
    __proto__: a,
    attr2: 'b'
};

```
위 코드는 `__proto__` 를 통해 프로토타입이 연결되는 과정을 설명하려고 작업한 실제로는 사용해선 안될 코드다. `__proto__` 속성에 개발자가 직접 접근한 코드가 문제인데 `Object.create()` 을 이용해 `__proto__` 속성에 직접 접근하지않고 프로토타입 체인을 연결할 수 있다.

```js
var a = {
    attr1: 'a'
};

var b = Object.create(a)

b.attr2 = 'b'
```
`Object.create()` 으로 인해 `__proto__` 가 a를 참조하는 새로운 빈객체를 리턴하게되고 b 에 참조되어 `b.attr1` 처럼 a 객체에도 접근할 수 있다.

```js
function Parent(name) {
    this.name = name
}

Parent.prototype.getName = function() {
    return this.name
};

function Child(name) {
    Parent.call(this, name)

    this.age = 0
}

Child.prototype = Object.create(Parent.prototype) // (1)
Child.prototype.constructor = Child

Child.prototype.getAge = function() {
    return this.age
}

var c = new Child() // (2)
```

(1) 에서 `Object.create()` 을 이용해 Child의 `prototype` 객체를 교체했다. (1) 에서 만들어진 새 객체 즉 `Child.prototype` 는 `__proto__` 속성이 `Parent.prototype` 을 나타내고 (2) 에서 Child의 인스턴스 c의 `__proto__` 가 `Child.prototype` 을 참조하게 된다. 이렇게 해서 c -> Child.prototype -> Parent.prototype 으로 연결되는 프로토타입이 만들어졌고 프로토타입 룩업시 의도한 탐색 경로로 식별자를 찾게 된다. 추가로 Child 생성자에서 Parent 생성자를 빌려서 객체를 확장한 것은 생성자 빌려 쓰기라는 오래된 기법으로 자바스크립트 상속에서 부모의 생성자를 실행하는 유일한 방법이다. 이렇게 해야 부모타입의 생성자의 내용도 상속 받는다.

## ES6
ECMAScript6에서 class 스펙이 추가되었다. 클래스라고는 하지만 새로운 개념이 아니고 상속의 구현 원리는 기존과 동일한 내용으로 장황했던 코드를 간결하는 문법적 설탕을 입혔다고 생각하면 된다. 예제로 만들어봤던 Parent 와 Child 는 ES6의 class 를 이용하면 아래와 같이 사용할 수 있다.

```js
class Parent {
    constructor(name) {
        this.name = name;
    }

    getName() {
        return this.name;
    }
}

class Child extends Parent {
    constructor(name) {
        super(name); // 생성자 빌려쓰기 대신....super 함수를 이용 한다.
        this.age = 0;
    }

    getAge() {
        return this.age;
    }

}
```
코드가 간결해지고 이해하기 쉽게 바뀌었다. 코드는 달라졌다 하더라도 이를 통해 만들어진 객체의 프로토타입 체인 연결 구조는 기존과 동일하고 또한 동일한 방식의 프로토타입 룩업으로 식별자를 찾아간다.

## 정리
이 글에서는 프로토타입 체인이 어떤 모습인지와 프로토타입 룩업을 통해 어떻게 식별자를 찾아가는지 그리고 프로토타입 체인을 어떻게 만드는지에 대해 알아보았으며. 글 대부분을 [이 곳에서](https://meetup.toast.com/posts/104) 참고했다. 사실 참고라기 보다는 거의 복붙에 가까운 수준인데 그래도 그 내용을 이해하기 위해 직접 코드를 실행시키고 이해하면서 가져왔다는 것에 만족하자.


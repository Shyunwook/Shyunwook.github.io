---
title: "[Javascript] iterable 객체"
subtitle: "iterable 객체, for-of, 반복가능한 객체"
categories:
  - Javascript
tags:
  - Javascript
  - 반복문
classes: wide
---

## 들어가며

이전에 제가 작성한 [반복문의 종류에 관한 글](https://shyunwook.github.io/javascript/Javascript-forEach,-for-in,-for-of/)에서 '`for-of`는 **반복 가능한 객체**에 있는 각 요소들에 대해 반복적인 동작을 할 수 있게 해준다' 라고 이야기한 적이 있습니다.

여기서 반복 가능한 객체란 무엇일까요?

단순히 **반복**이라는 단어를 떠올렸을 때는 배열의 요소나 객체의 속성을 순회하는 것을 떠올릴 수 있겠지만 ES6에 들어서며 약간 더 심오한(?) 개념이 추가됩니다😳

javascript에서 말하는 **반복(순회)가능한 객체**란 무엇인지 알아보도록 하겠습니다.

## iteration Protocol

흔히 프로그래밍에서 말하는 iteration이란 반복처리라는 뜻으로, 데이터 내부 요소를 연속적, 반복적으로 꺼내어 사용하는 행위를 의미합니다.
간단하게 아래와 같은 상황을 예로 들 수 있습니다👇🏻

```javascript
const arr = [1, 2, 3, 4, 5];

for (let i in arr) {
  console.log(arr[i]);
} // 1 2 3 4 5
```

ES6에 도입된 iteration 프로토콜은 데이터 내부 요소를 순회하기 위해 미리 약속된 규칙입니다.

말이 어렵죠? 쉽게 말하자면, 자바스크립트에서 데이터를 순회 하기 위해서는 **iteration**이라는 규칙을 만족시킬 필요가 있다는 것입니다!

그리고 iteration 프로토콜은 다시 **itarable** 프로토콜과 **iterator** 프로토콜로 나뉩니다!

결과적으로 반복 가능한 객체가 되기 위해선

- iterable 프로토콜
- iterator 프로토콜

이 두가지 프로토콜을 준수해야 하는 것이죠🤕(우선은 이정도로 이해하고 넘어갑시다!)  
복잡해 보이지만 각각의 규칙만 기억하시면 쉽게 따라오실 수 있습니다!!

## 이터러블 객체

이터러블 프로토콜을 준수하는 객체를 **이터러블 객체**라고 합니다.

> 규칙 : Symbol.iterator 메소드를 소유하고 있는 객체

이터러블 객체는 **Symbol.iterator**라는 특별한 메소드를 가지고 있는 객체입니다. 그리고 Symbol.iterator는 **iterator** 객체를 반환하는 역할을 합니다.

```text
  이터러블 객체는 Symbol.iterator 메소드를 소유하고, Symbol.iterator 메소드는 이터레이터를 반환한다!
```

```javascript
const arr = [1, 2, 3, 4, 5];

console.log(Symbol.iterator in arr); // true
// 반복 가능한 객체인 배열은 Symbol.iterator를 소유하고 있습니다!
```

## 이터레이터

이터레이터 프로토콜을 준수하는 객체를 **이터레이터 객체**라고 합니다.

> 규칙 : next 메소드를 소유하고 있는 객체

이터레이터 객체는 **next** 메소드를 소유해야 합니다. 그리고 next 메소드는 'value'와 'done'이라는 프로퍼티(속성)을 갖는 객체를 반환해야 합니다.

```text
  이터레이터 객체는 next 메소드를 소유하고, next 메소드는 { value : 'xx', done : true } 와 같은 객체를 반환한다!
```

```javascript
const arr = [1, 2, 3];
const iter = arr[Symbol.iterator](); // Synbol.iterator 메소드는 이터레이터를 반환합니다.

// 이터레이터는 next 메소드를 소유하고 있습니다.
console.log("next" in iter); // true

console.log(iter.next()); // {value: 1, done: false}
console.log(iter.next()); // {value: 2, done: false}
console.log(iter.next()); // {value: 3, done: false}
console.log(iter.next()); // {value: undefined, done: false}
```

이터레이터의 next 메소드를 호출하면 내부적으로 데이터의 각 요소를 순회하며, 현재 이터레이터가 가리키고 있는 요소에 대한 정보를 객체 형태로 반환합니다.

- value : 현재 요소의 값
- done : 데이터 순회 완료 여부

## 이터레이션 프로토콜이 왜 필요할까?

이러한 복잡한 컨셉의 프로토콜과 객체는 왜 필요한 것일 까요?
자바스크립에는 데이터를 사용하는 다양한 데이터 소비자가 존재합니다. 예를 들면 `for-of`, `for-in`, `spred연산자` 등등의 것들이죠!
또한 `배열`, `문자열` 등과 같은 데이터 공급자(데이터 소스)들도 존재합니다.

![](/assets/images/post/iteration-protocol-interface.png){: width="600"}{: .align-center}

이렇게 다양한 데이터 소비자와 데이터 공급자들이 각각의 순회방식을 가지고 있다면 어떨까요? 가령 `배열`과 `문자열`의 순회 방식이 다르다면, `for`문 하나를 작성하더라도 각 데이터의 순회 방식을 고려해야 합니다.

이런 불편함을 방지하기 위해 각 데이터 소스가 하나의 순회 방식을 갖도록 약속하고, 데이터 소비자는 하나의 순회 방식에 대해서만 신경쓰면 되도록 한 것이죠!!😎 그리고 이 하나의 순회 방식만 잘 지켜 준다면 조금더 다양한 방식의 순환 기능을 직접 구현할 수도 있을 것입니다.

## 이터러블 객체 구현

이터레이션 프로토콜만 준수 한다면 입맛에 맞는 다양한 이터러블 객체를 만들 수 있습니다. `range`라는 이터러블 객체를 만들어 1 부터 3까지의 정수를 출력하는 예제를 만들어 보겠습니다.🏄‍♂️

```javascript
const range = {
  from: 1,
  to: 3,
  [Symbol.iterator]() {
    // 이터러블 프로토콜!
    return {
      current: this.from,
      last: this.to,
      next() {
        // 이터레이터 프로토콜!
        if (this.current <= this.last) {
          return { value: this.current++, done: false };
        } else {
          return { value: this.current++, done: true };
        }
      }
    };
  }
};

// 일반 객체가 아닌 이터러블 객체는 for-of로 순회할 수 있습니다!!
for (let num of range) {
  console.log(num);
}
```

예제의 실행 결과입니다.

![](/assets/images/post/iterable-ex.png)

방금 우리가 작성한 `range` 객체는 이터레이션 프로토콜을 잘 지키고 있습니다. Symbol.iterator 메소드를 가지고 있고, Symbol.iterator 메소드는 next 메소드를 가지는 객체를 반환할 것이고, next 메소드는 `{value: xx, done: xx}` 형태의 객체를 잘 반환하고 있으니까요!!🤩

예제가 실행될 때 for-of의 내부 동작을 while문으로 나타내면 아래와 같습니다

```javascript
//... range 구현 부분 생략!

const iterator = range[Symbor.iterator](); // 명시적으로 iterator 호출

while (true) {
  const result = iterator.next(); // { value : 1, done: false}
  if (result.done) {
    break;
  }
  console.log(result.value);
}
```

이터러블 객체인 range와 range가 소유한 Symbol.iterator 메소드, Symbol.iterator가 반환하는 이터레이터 객체에 대해 생각해 보면 위와 같이 명시적으로 이터레이터를 호출하는 코드도 이해하실 수 있을 겁니다.

## 이터러블이면서 이터레이터인 객체를 생성하는 함수

예제 코드를 약간🤓 수정하면 **이터러블이면서 이터레이터인 객체**를 생성하는 함수를 만들 수 있습니다. **이터러블이면서 이터레이터** 라는 것이 이해가 안가시죠?? 코드를 확인해 보겠습니다.

```javascript
const range = function() {
  let from = 1,
    to = 3;

  return {
    // ------- 1
    current: from,
    last: to,
    [Symbol.iterator]() {
      return this; // ------- 2
    },
    next() {
      // ------- 3
      if (this.current <= this.last) {
        return { value: this.current++, done: false };
      } else {
        return { value: this.current++, done: true };
      }
    }
  };
};

const iterator = range(); // ------- 4

console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
```

예제의 실행 결과는 아래와 같습니다!

![](/assets/images/post/iterable-ex2.png)

코드를 천천히 뜯어보면,

- 코드의 1번 부분에서 `range` 함수는 하나의 객체를 반환합니다. 해당 객체는 **Symbol.iterator** 메소드를 소유하고 있죠! 그러므로 해당 객체는 **이터러블** 객체라고 할 수 있습니다.
- 코드의 2번 부분에서 **Symbol.iterator** 메소드는 `this`를 반환하고 있는데, 여기서 `this`는 1번 부분에서 반환하려고 했던 객체 자기 자신입니다! 그리고 그 객체는 **next**메소드를 지니고 있군요!(코드 3번 부분) 그러므로 해당 객체는 **이터레이터** 객체라고 할 수 있습니다.

즉 **이터러블** 이면서 **이터레이터**인 객체가 만들어 진 것입니다. 그렇기 때문에 코드의 4번 부분에서 Symbol.iterator를 따로 호출하지 않고도 이터레이터 객체를 사용할 수 있습니다.

## 무한개의 이터레이터와 지연 평가

지금까지는 유한한 횟 수 동안 반복하는 이터러블 객체의 예를 살펴보았습니다. 이를 약간 응용하면 **무한히** 반복하는 이터러블 객체를 생성할 수 있습니다.

```javascript
const number = function() {
  let count = 0;
  return {
    [Symbol.iterator]() {
      return this;
    },
    next() {
      count++;
      return { value: count };
    }
  };
};

for (let num of number()) {
  console.log(num);
  if (num === 10) {
    break;
  }
}
```

`number` 함수는 **이터러블이면서 이터레이터**인 객체를 생성하는 함수로, `number` 함수가 반환하는 이터레이터는 next메소드의 반환 결과로 done 속성을 가지고 있지 않습니다. 즉 데이터를 순회하다가 순회를 멈출 시점을 알 수 없기 때문에 무한히 반복하게 되는 것이죠!

이터러블 객체는 해당 객체가 데이터 소비자에게 할당되어 이터레이터 객체의 next 메소드가 호출되기 전까지, 메모리에 데이터를 가지고 있지 않다는 특징을 가지고 있습니다.

즉, 위의 예에서 `number` 함수가 `for-of` 에 할당되기 전까지, 1부터 10이라는 숫자는 메모리상에 존재하지 않는 다는 것 입니다.

```javascript
// 구조분해 할당의 예
const [one, two, three] = number();
console.log(one, two, three); // 1 2 3
```

[구조분해 할당(destructuring)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)의 경우도 마찬가지입니다!! 이러한 특성을 이용해 데이터가 필요할때 까지 데이터의 생성을 지연하다 필요할 때 데이터를 생성할 수 있습니다.

> 함수의 '평가'란 함수를 수행하는 시점을 의미합니다.

## 나가며

반복문을 다루다보면 이러한 에러 메세지를 발견하게 되는 경우가 종종 있습니다.

![](/assets/images/post/iterable-ex3.png)

```javascript
const testObject = {
  one: 1,
  two: 2
};

for (let num of testObject) {
  console.log(num);
}
```

그때마다 '아 뭐야!'하고 원인을 찾지 않은채 빠르게 다른 방식으로 문제를 피해가는 방식으로 일을 해왔습니다🤕
앞으로도 많은 경우에 이렇게 덮어두고 지나가는 경우가 많겠지만 조금씩 문제를 돌파하는 습관을 키워야겠습니다!

이제서야 저 에러 한 줄을 이해할 준비가 된 것 같네요🤺

긴 글 읽어주셔서 감사합니다! 누군가에게는 도움이 됐길 바랍니다👏🏻

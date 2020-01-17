---
title: "[Javascript] forEach, for in, for of"
subtitle: "Javascript 반복문 비교"
categories:
  - Javascript
tags:
  - Javascript
classes: wide
---

## 들어가며

Javascript에는 객체와 배열을 반복하는 여러가지 방법이 존재합니다. 실제 코딩을 하면서 여러 방식들을 두루두루 사용해 보았지만 어떤 상황에 어떤 방식을 사용하는 것이 적합한가에 대한 개념은 잡혀있지 않았습니다. 고질적인 습관성 코드...😳

그냥 그때 그때 느낌적인 느낌으로 적절한 방법을 사용했다고 할까요??  
아버지은(?) 방에 들어가셨다 라고 쓰지 않는 것 처럼요!

그래서 이번 기회에 각 반복의 방법에 대해 정리해 보려고 합니다.

## forEach

우선 가장 덜 헷갈리는 forEach 부터 살펴보겠습니다.

```javascript
let arr = [1, 2, 3];

arr.forEach((el, i, arr) => {
  console.log(el);
});
// 1 2 3
```

`forEach`의 콜백함수는 다음 세가지 인수와 함께 호출될 수 있습니다.

- 처리할 현재 요소
- 처리할 현재 요소의 인덱스
- `forEach`를 호출한 배열

`forEach`는 엄밀히 말하면 반복'문'이 아니라 메소드입니다. Array의 prototype 객체 안에 선언된 함수이기 때문에 forEach는 배열에 대해서만 사용 가능합니다. 아래와 같이 객체에 사용하려고 한다면 `TypeError`가 발생합니다

```javascript
let obj = { a: 1, b: 2, c: 3 };

obj.forEach((el, i) => {
  console.log(el);
});

//TypeError: arr.forEach is not a function...
```

## for-in VS for-of

`for-in`과 `for-of` 도대체 뭐가 다른 걸까요???😳 결론부터 말하자면

- `for-in`은 **객체**에 있는 각 항목들에 대해 반복적인 동작을 할 수 있게 해줍니다.
- `for-of`는 **반복 가능한 객체**에 있는 각 항목들에 대해 반복적인 동작을 할 수 있게 해줍니다.

뭔가 너무 어렵습니다.... 네 맞습니다... 이 글을 적고있는 저도 너무 어렵습니다😭
많이 어렵지만 하나하나 살펴보도록 하겠습니다.

## for-of

```javascript
let arr = ["a", "b", "c"];

for (let value of arr) {
  console.log(`value : ${value}`);
}
```

![](/assets/images/post/forof1.png)

`for-of`는 각 요소를 순회하며 요소의 값에 접근합니다. 콜백 함수가 필요 없다는 점만 뺀다면 이런 동작 방식은 앞서 살펴 본 `forEach`와 비슷해 보입니다.

> `for-of`는 `forEach`와 다르게 순회 중 `continue`, `break`, `return`을 사용할 수 있다는 장점이 있습니다!!!

아래 코드는 어떤 동작을 하게 될까요?

```javascript
let obj = { a: 1, b: 2, c: 3 };

for (let value of obj) {
  console.log(`value : ${value}`);
}
```

![](/assets/images/post/forof2.png)

`for-of`는 **반복 가능한 객체**에 대해서 사용할 수 있기 때문에 위와 같은 경우에 TypeError가 발생하게 됩니다.

> 반복 가능한 객체에는 Array, Map, Set, String, TypedArray, arguments 객체 등이 있습니다. iterable Object에 대해선 추후 다뤄보도록 하겠습니다.
>
> > 사실 `for-of`는 반복 가능한 객체라면 어떤 것에라도 사용할 수 있다는 것이 장점이기도 합니다!!

### 참고사항

```javascript
let arr = ["a", "b", "c"];

for (const [key, value] of arr.entries()) {
  console.log(`key : ${key}, value : ${arr[key]}`);
}
```

위와 같이 `for-of`도 `Array.prototype.entries()` 메소드를 사용하여 배열의 인덱스에 접근할 수 있습니다.

## for-in

```javascript
let arr = ["a", "b", "c"];

for (let key in arr) {
  console.log(`key : ${key}, value : ${arr[key]}`);
}
```

![](/assets/images/post/forin1.png)

`for-in`의 경우 요소의 key를 통해 각 요소를 순회 합니다.

> Javascript에서는 배열도 객체의 한 종류이고 배열의 `index`가 요소의 `key`입니다!

그리고 `for-in`은 객체에 대해서도 잘 동작합니다!

```javascript
let obj = { a: 1, b: 2, c: 3 };

for (let key in obj) {
  console.log(`key : ${key}, value : ${obj[key]}`);
}
```

![](/assets/images/post/forin2.png)

### for-in 사용의 위험성

많은 개발자들이 `for-in`을 사용함에 있어 주의를 기울여야한다고 말합니다. 그 이유는 무엇일까요?'
다음의 코드를 살펴보겠습니다.

```javascript
let obj = { a: 1, b: 2, c: 3 };

Object.prototype.fnc = function() {
  console.log("fnc in Object");
}; // --- 1

for (let key in obj) {
  console.log(`key : ${key}, value : ${obj[key]}`);
}
```

코드의 1번 부분에서 모든 객체가 상속하고 있는 Object의 prototype에 `fnc`라는 메소드를 하나 추가하였습니다.

![](/assets/images/post/forin3.png)

결과에서 볼 수 있듯이 `for-in`을 사용하게 되면 해당 객체의 속성 뿐만 아니라 객체가 상속하고 있는 속성까지도 접근하기 때문에 의도치 않은 결과가 나올 수 있습니다.

> 이러한 현상을 방지하기 위해선 `hasOwnProperty` 함수를 사용할 수 있습니다.
>
> > 혹은 Object.keys() 를 이용하여 해당 객체의 `key` 값에 대해 순회 할 수 있습니다.

```javascript
let obj = { a: 1, b: 2, c: 3 };

Object.prototype.fnc = function() {
  console.log("fnc in Object");
};

for (let key in obj) {
  if (obj.hasOwnProperty(key)) {
    console.log(`key : ${key}, value : ${obj[key]}`);
  }
}
```

> 자바스크립트의 코딩 스타일과 잠재된 위험성을 점검해주는 JSLint에서도 `for-in`을 사용할 때 이처럼 `hasOwnProperty` 함수로 객체의 속성인지를 점검할 것을 권장하고 있습니다.

이 외에도 `for-in`은 객체를 순회 할 때 속성들의 순서를 보장하지 않는 다는 단점이 있습니다. **따라서 배열을 순회할때 `for-in`을 사용하는 것은 그리 좋은 생각이 아닙니다**

## 나가며

좀 복잡하고 장황한 글이 되지 않았나 싶습니다ㅜㅜ

정리해 보자면,

- forEach
  - 배열에 사용 가능
  - 콜백에서 각 요소의 값과 인덱스를 매개변수로 받을 수 있음
- for-of
  - 반복 가능한 객체에 사용 가능(iteration protocol을 준수하는 객체)
  - 일반 객체 사용X, 각 요소의 값에 접근
  - `continue`, `break`, `return`을 사용할 수 있음
- for-in
  - 배열, 객체에 사용 가능
  - 상속 받은 값에도 접근 => `hasOwnProperty`로 속성 검사 필요!
  - `continue`, `break`, `return`을 사용할 수 있음

글 작성을 위해 혼자 이런 저런 자료를 찾아보면서, 정말 자주 쓰는 표현이었는데도 모르는 것들이 많았다는 것을 다시한번 깨달았습니다😫 역시 아직 갈길이 멀군요!! 부지런히 가보겠습니다.👏🏻👏🏻👏🏻

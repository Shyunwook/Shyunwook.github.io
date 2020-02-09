---
title: "[Javascript] 이벤트 위임, 이벤트 전파 중단"
subtitle: "이벤트 위임, 이벤트 전파 중단, 이벤트 기본 동작 취소"
categories:
  - Javascript
tags:
  - Javascript
  - 이벤트
classes: wide
---

## 들어가며

Javascript 이벤트 두 번째 시간입니다😎

[저번 글](/javascript/Javascript-이벤트란)을 통해 우리는 이벤트란 무엇인가, 이벤트의 전파방식, 이벤트 핸들러에 대해 알아보았습니다. 그럼 이제 아래와 같은 상황을 상상해 봅시다.

1. 100개의 버튼이 존재하는 경우
2. 화면에 동적으로 새로운 버튼이 생성 되는 경우

이벤트 핸들러를 어떻게 등록해야 할까요?🤔 위와 같은 상황에서 효과적으로 이벤트 제어를 할 수 있는 방법이 있습니다. 바로 **이벤트 위임(delegation)** 패턴인데요, 아주 중요한 개념이니 반드시 이해하고 넘어갑니다.

화이팅!!👍🏻

## 이벤트 위임이 왜 필요한거야?😩

이벤트 위임이 무엇인지 알아보기 전, 이벤트 위임이란 것이 왜 필요한지 먼저 예제를 통해 알아보겠습니다.

```html
<ul id="parent-list">
  <li id="list-1">Item 1</li>
  <li id="list-2">Item 2</li>
  <li id="list-3">Item 3</li>
  <li id="list-4">Item 4</li>
  <li id="list-5">Item 5</li>
  <li id="list-6">Item 6</li>
</ul>
```

```javascript
let list = document.querySelectorAll("li");
list.forEach(function(li) {
  li.addEventListener("click", function(e) {
    console.log(e.target.id);
  });
});
```

> 여기서 `e`는 이벤트 객체, `target`은 실제 이벤트가 발생한 요소를 의미합니다.

반복문을 통해 모든 `li` 요소에 이벤트 핸들러를 등록하고, `li`를 클릭했을 때 해당 `li`의 `id` 속성을 화면에 출력하는 예제입니다. 큰 문제 없이 잘 동작하는군요! 그럼 예제를 조금 더 확장해 보겠습니다.

```html
<button id="add">추가 버튼!!</button>
<ul id="parent-list">
  <li id="list-1">Item 1</li>
  <li id="list-2">Item 2</li>
  <li id="list-3">Item 3</li>
  <li id="list-4">Item 4</li>
  <li id="list-5">Item 5</li>
  <li id="list-6">Item 6</li>
</ul>
```

```javascript
let add_btn = document.querySelector("#add");
let parent = document.querySelector("#parent-list");
let list = document.querySelectorAll("li");

// 새로운 li 요소의 추가
add_btn.addEventListener("click", function() {
  let new_li = document.createElement("li");
  new_li.setAttribute("id", "list-7");
  new_li.innerText = "새로운 리스트";

  parent.appendChild(new_li);
});

list.forEach(function(li) {
  li.addEventListener("click", function(e) {
    console.log(e.target.id);
  });
});
```

버튼을 클릭하면 새로운 `li` 요소를 하나 생성하는 이벤트 핸들러가 추가 됐습니다. 예제를 실행해 볼까요?

![](/assets/images/post/delegation.gif){: .center}

결과에서 볼 수 있듯이 기존에 존재하던 요소의 경우 이벤트 핸들러가 잘 동작하지만 새로 생성된(동적으로 생성된 요소) `li`는 클릭해도 아무런 동작을 하지 않습니다. 의도대로 동작하지 않는 이유는 간단합니다.

반복문으로 이벤트 핸들러를 등록할 시점에 새로운 `li`는 아직 존재 하지 않았기 때문입니다!!😳

이런 문제를 해결할 수 있는 것이 바로 **이벤트 위임** 입니다.

> 동적 요소의 이벤트 제어 외에도 기존 이벤트 제어 방식은 아래와 같은 문제가 발생할 수 있습니다.
>
> - 코드의 유지보수 비용 증가
> - 중복된 이벤트 핸들러로 인한 메모리 사용의 비효율성

## 이벤트 위임이란?

이벤트 위임이란, '이벤트 제어의 역할을 각 하위요소가 아닌 상위 요소에게 맡기는 방식'이라고 할 수 있습니다. 앞에서 다뤘던 예제를 **이벤트 위임** 방식으로 개선해 보겠습니다.

```html
<!DOCTYPE html>

<html>
  <body>
    <button id="add">추가 버튼!!</button>
    <ul id="parent-list">
      <p id="p태그">paragraph</p>
      <li id="list-1">Item 1</li>
      <li id="list-2">Item 2</li>
      <li id="list-3">Item 3</li>
      <li id="list-4">Item 4</li>
      <li id="list-5">Item 5</li>
      <li id="list-6">Item 6</li>
    </ul>
  </body>
</html>
```

```javascript
let parent = document.querySelector("#parent-list");

///... 생략

// 수정 이전 내용
// list.forEach(function(li) {
//   li.addEventListener("click", function(e) {
//     console.log(e.target.id);
//   });
// });

// 수정된 내용
parent.addEventListener("click", function(e) {
  console.log(e.target.id);
});
```

이벤트 제어역할을 하위요소(`li`)가 아닌 상위 요소(`ul`)에게 맡기는 방식으로 수정하였습니다.

![](/assets/images/post/delegation2.gif){: .center}

> `target`은 우리가 실제 클릭 이벤트를 발생시킨 요소이기 때문에 `li`의 id가 출력됐습니다.

동적으로 추가된 요소에서도 이벤트 핸들러가 잘 동작합니다!!🤭 이처럼 우리는 이벤트 핸들러를 각 하위 요소마다 등록하는 것이 아니라 상위 요소에 등록함으로써 동적인 요소의 이벤트 제어를 할 수 있게 되었습니다.

### 이게 어떻게 가능한거지?

**이벤트의 전파**에 대해 기억나시나요? 이벤트 위임은 바로 이벤트 전파의 특성을 이용한 패턴입니다. 위 예제에서 우리는 상위 요소인 `ul` 태그에 이벤트 핸들러를 등록하였습니다. 그리고 하위 요소인 `li`를 클릭했죠. 이벤트 전파 흐름에 따라 내부적으로는 이런 식으로 이벤트가 전파 됨을 예상할 수 있습니다.

- 상위 요소 ... -> **ul** -> li : 캡쳐링 단계
- li : 타겟 단계
- li -> **ul** -> ... 상위 요소 : 버블링 단계

즉, 하위 요소를 클릭하더라도 이벤트의 전파에 의해 상위 요소로 이벤트가 전파 되기 때문에 **상위 요소에서 하위 요소의 이벤트 발생을 감지하고 제어** 할 수 있는 것입니다.

### 생각해 볼 점

마지막 예제에서 `ul`의 하위에 `li`, `p` 두가지 요소가 존재하고 있습니다. 하위 요소 중 특정 요소에서 발생한 이벤트만 제어 하고 싶을때는 어떻게 해야할까요?

> 힌트 : `target`을 이용해서 구현할 수 있지 않을까요🤭

<br />
<br />
<br />

---

👇🏻정답!!👇🏻

```javascript
parent.addEventListener("click", function(e) {
  if (e.target.nodeName == "LI") console.log(e.target.id);
});
```

이벤트가 발생한 `target`이 내가 원하는 요소인지 검사하는 코드를 추가하면 끝!!!😋

![](/assets/images/post/delegation3.gif){: .center}

## 이벤트 전파 중단 & 이벤트 기본 동작의 취소

이벤트의 마지막 내용으로 이벤트 전파 중단과 이벤트 기본 동작의 취소에 대해 알아보겠습니다.

### 이벤트 전파 중단

---

아래의 예를 살펴 봅시다!

```html
<div class="parent" style="background: black">
  <button class="child">클릭!</button>
</div>
```

```javascript
let parent = document.querySelector(".parent");
let child = document.querySelector(".child");

parent.addEventListener("click", function(event) {
  console.log("parent click!");
});

child.addEventListener("click", function(event) {
  console.log("child click!");
});
```

이벤트 전파에 관한 내용을 떠올리며 결과를 예상해 보면 버튼을 클릭 했을 때 이벤트 버블링에 의해 `button`과 `div`의 이벤트 핸들러가 모두 동작할 것입니다.

![](/assets/images/post/stop.gif){: .center}

예상한대로 `div`를 클릭 했을 때는 `div`의 이벤트 핸들러만 동작하고, `button`을 클릭했을 때는 `div`와 `button`의 이벤트 핸들러가 모두 동작했습니다. 만약 버튼을 클릭했을 때, 버튼의 이벤트 핸들러만 동작하게 하고 싶다면 어떻게 하면 될까요?

`stopPropagation` 메소드를 사용하여 간단하게 구현할 수 있습니다. 해당 메소드는 더이상 이벤트의 전파가 진행되지 않게 해줍니다.

버튼의 이벤트 핸들러에서 `stopPropagation` 메소드를 호출하면 이벤트 버블링 단계가 진행되는 것을 막아 원하는 결과를 얻을 수 있습니다.

```javascript
child.addEventListener("click", function(event) {
  event.stopPropagation();
  console.log("child click!");
});
```

![](/assets/images/post/stop2.gif){: .center}

### 이벤트 기본 동작의 취소

---

웹브라우저의 요소들은 기본적인 동작 방식을 가지고 있습니다. 특정 이벤트가 발생했을 때 동작 방식이 미리 정의되어 있는 것이라고 이해하시면 됩니다.

- `a` 링크를 클릭하면 해당 페이지가 열림
- `form` 안에서 `submit` 이벤트가 발생하면 데이터를 전송
- `checkbox` 를 클릭하면 체크가 됨

이렇게 미리 정의된 이벤트 기본 동작은 `preventDefault` 메소드를 사용하여 취소할 수 있는데, 사용법은 아주 간단합니다.

```html
<form>
  <input type="checkbox" />
  <label for="checkbox">체크박스</label>
</form>
```

```javascript
document.querySelector("input[type=checkbox]").addEventListener("click", function(e) {
  console.log(e.cancelable);
  e.preventDefault();
});
```

![](/assets/images/post/prevent.gif){: .center}

> 인라인 이벤트 핸들러 방식과 이벤트 핸들러 프로퍼티 방식의 경우는 `return false`로 같은 동작을 할 수 있습니다.

## 나가며

오늘 내용은 어떠셨나요? 최대한 쉽게 적어보려고 했는데 어떠셨을지 모르겠네요🥺

오늘 다룬 내용들은 실제 개발 도중 마주치는 여러 상황들에서 유용하게 사용되는 개념인 만큼 꼭 이해하고 넘어가시면 좋을 것 같습니다.

이상한 점이나 궁굼한 점이 있다면 언제든 댓글 남겨주세요~!👍🏻

## 참고

![test](https://naver.com)

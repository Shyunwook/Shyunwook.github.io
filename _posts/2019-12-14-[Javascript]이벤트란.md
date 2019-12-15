---
title: "[Javascript] 이벤트란?"
subtitle: "이벤트란, 이벤트 핸들러, 이벤트 전파방식"
categories:
  - Javascript
tags:
  - Javascript
  - 이벤트
classes: wide
---

## 들어가며

개인적으로 개발을 하면서 많은 코드들을 무심코 작성하는 경우가 많은 것 같습니다. 물론 결과물만 뚝딱뚝딱 나온다면 큰 문제는 없지만 문득 '이해 없이 작성한 코드가 정말 내 코드일까?' 라는 생각이 들어 습관성 코드(?)에 대한 내용들을 정리해 보려고 합니다.

오늘은 그 첫번째로 '이벤트'에 대한 내용을 다뤄보겠습니다.

매 번 잘 읽히는 글을 쓰자는 생각으로 내용을 정리하고 있습니다만... 모든 글 쓰는 분들 존경합니다😭 부족한 글이지만 재밋게 봐주세요!👍🏻

## 이벤트란?

브라우저 환경에서 이벤트란 DOM 요소와 관련되어 발생하는 어떤 사건들을 의미합니다.  
예를 들어 다음과 같은 상황들이 있을 수 있겠죠.

- 버튼을 클릭 했을 때
- 브라우저 창의 크기를 조절 했을 때
- 웹페이지가 로드 됐을 때

브라우저는 이 외에도 다양한 [이벤트](<(https://developer.mozilla.org/en-US/docs/Web/Events)>)를 감지 할 수 있습니다.

## 이벤트 핸들러(리스너)

웹 어플리케이션은 '특정 요소에서 특정 이벤트가 발생 했을 경우 브라우저로 부터 통지 받겠다'라는 것과 해당 이벤트 발생 시 '수행 할 행동'을 등록 할 수 있습니다. 이때 '수행할 행동'을 `이벤트 핸들러`라고 하고 이벤트 핸들러를 명시하는 일련의 과정을 `이벤트 핸들러 등록` 이라고 합니다. 예제 코드를 살펴 보겠습니다🤩

```html
<!DOCTYPE html>

<html>
  <body>
    <button class="btn">Click</button>

    <script>
      document.querySelector(".btn").addEventListener("click", function() {
        console.log("Hi!"); // Hi!
      });
    </script>
  </body>
</html>
```

사용자가 버튼을 클릭 했을 때 콘솔창에 'Hi'를 출력하는 간단한 예제입니다. `addEventListener` 메소드를 사용하여 `btn` class를 가진 요소의 `click` 이벤트에 대해 `console.log('Hi!')`라는 동작을 하는 이벤트 핸들러(함수)를 등록해보았습니다. 이러한 이벤트 핸들러는 이벤트 발생 전에는 실행되지 않다가, 특정 이벤트가 발생했을 때 실행되는 것이 특징입니다.

## 이벤트 핸들러 등록 방식

이벤트 핸들러를 등록하는 방식은 크게 3가지가 있습니다.

### 1.인라인 이벤트 핸들러 방식

```html
<!DOCTYPE html>

<html>
  <body>
    <button class="btn" onclick="handler()">Click</button>

    <script>
      function handler() {
        console.log("핸들러 실행");
      }
    </script>
  </body>
</html>
```

첫번째 방법은 HTML 요소의 이벤트 핸들러 속성(attribute)에 직접 이벤트 핸들러를 등록하는 방식입니다.
다른 방식에 비해 이벤트 핸들러와 요소의 관계가 직관적으로 보일 수 있으나 HTML코드와 Javascript 코드는 분리하는 것을 권장하므로 좋은 방법은 아닙니다.

### 2.이벤트 핸들러 프로퍼티 방식

두번째 방법은 DOM 요소의 이벤트 핸들러 프로퍼티를 이용하여 이벤트 핸들러를 등록하는 방식입니다.

```html
<!DOCTYPE html>

<html>
  <body>
    <button class="btn">Click</button>

    <script>
      let targetBtn = document.querySelector(".btn");
      targetBtn.onclick = handler1;
      targetBtn.onclick = handler2;

      function handler1() {
        console.log("핸들러111 실행");
      }

      function handler2() {
        console.log("핸들러222 실행");
      }
    </script>
  </body>
</html>
```

이와 같은 방식은 HTML과 Javascript 코드를 분리 할 수 있다는 장점이 있지만 하나의 이벤트 핸들러만 등록 할 수 있다는 단점이 있습니다. 위 코드의 경우 버튼 클릭 시 아래와 같은 결과가 나오게 됩니다.

![](/assets/images/post/property_binding.png)

> ### HTML 요소와 DOM 요소?
>
> HTML은 우리가 html 문법을 사용하여 작성한 html 파일 자체를 의미합니다.  
> DOM(Document Object Model)은 우리가 작성한 html 파일이 Javascript와 같은 프로그래밍 언어로 접근 할 수 있도록 브라우저에 의해 파싱된 것 정도로 알고 넘어갑시다! DOM에 대해서는 이후에 다시 다루도록 하겠습니다.

### 3.addEventListener 메소드 방식

`addEventListener` 메소드를 이용하여 특정 DOM 요소에 이벤트 핸들러를 등록하는 방식이며 이벤트 등록 방식 중 가장 권장되는 방법입니다. `addEventListner`는 아래 세가지 매개변수를 가집니다.

- type : 이벤트의 타입 ex) click, load, scroll...
- listner : type에서 지정한 이벤트 발생시 실행할 함수, (= 이벤트 핸들러)
- useCapture(option) : capture의 사용여부(Boolean)

  - **선택사항**으로 필수 매개변수가 아닙니다
  - 기본값은 `false`로 이 경우 이벤트 버블링 단계에서 리스너를 트리거 합니다.
  - `true`로 설정할 경우 이벤트 캡쳐 단계에서 리스너를 트리거 합니다.

> addEventListener의 세번째 매개변수(option)은 실제 'passive', 'once', 'cature'와 같은 다양한 옵션을 지원합니다. addEventListener에 대한 내용도 따로 다뤄보도록 하겠습니다.  
> 자세한 내용을 원하시는 분은 [링크](https://developer.mozilla.org/ko/docs/Web/API/EventTarget/addEventListener)를 참고해주세요🧐

코드를 살펴보겠습니다.

```html
<!DOCTYPE html>

<html>
  <body>
    <button class="btn">Click</button>

    <script>
      document.querySelector(".btn").addEventListener("click", handler1);
      document.querySelector(".btn").addEventListener("click", handler2);

      function handler1() {
        console.log("핸들러111 실행");
      }

      function handler2() {
        console.log("핸들러222 실행");
      }
    </script>
  </body>
</html>
```

`btn` class를 가지는 button 요소에 `handler1`와 `handler2` 함수를 이벤트 핸들러로 등록했습니다. 버튼을 클릭하면 아래와 같은 결과를 확인 할 수 있습니다.

![](/assets/images/post/addEventListener.png)

[이벤트 핸들러 프로퍼티 방식]({{site_url}}/javascript/이벤트-버블링,-이벤트-캡쳐,-이벤트-위임/#2이벤트-핸들러-프로퍼티-방식)과의 차이점이 느껴지시나요?? 네 맞습니다! `addEventListener`를 이용할 경우에는 하나의 요소에 동일 이벤트 타입 리스너를 여러개 등록 할 수 있다는 장점이 있습니다.

## 이벤트의 흐름

그렇다면 브라우저에서 특정 이벤트가 발생 했을때 내부적으로 어떤일이 일어날까요? addEventListener의 마지막 매개변수 `useCapture`를 이해하기 위해서라도 이벤트 흐름에 대해 알아볼 필요가 있습니다.

### 이벤트 전파

이벤트가 발생했을 때 이벤트는 해당 요소에만 적용되는 것이 아닙니다. 특정 요소에 이벤트가 발생했을 때 HTML의 계층 구조에 따라 이벤트가 연쇄적으로 전파되는데 이를 **이벤트 전파**라고 합니다. 이벤트의 전파는 전파 방향에 따라 두가지로 나눌 수 있습니다.

- 버블링: 중첩된 요소에서 이벤트가 발생한 요소부터 해당 요소를 감싼 부모 요소까지 이벤트가 전파 되는 것
- 캡쳐링: 버블링과 반대로 부모 요소부터 이벤트 발생 요소까지 이벤트가 전파 되는 것

여기서 중요한 점은 이벤트가 발생했을때 해당 이벤트는 **캡쳐링 -> 타겟 -> 버블링** 의 단계를 거쳐 전파된다는 것입니다.

![](/assets/images/post/eventflow.svg){: width="500"}{: .align-center}

### 이벤트 버블링

이벤트 버블링이 어떤 방식으로 동작하는지 예제를 통해 알아보겠습니다.

```html
<!DOCTYPE html>

<html>
  <body>
    <div>
      <button class="btn">Click</button>
    </div>

    <script>
      document.querySelector(".btn").addEventListener("click", function() {
        console.log("button!");
      });
      document.querySelector("div").addEventListener("click", function() {
        console.log("div!");
      });
      document.querySelector("body").addEventListener("click", function() {
        console.log("body!");
      });
    </script>
  </body>
</html>
```

버튼을 클릭하게 되면 이벤트 전파 순서에 따라 먼저 캡쳐링 단계가 진행 됩니다.

- body -> div -> button

그 후 타겟인 button을 거쳐 버블링 단계가 진행 됩니다.

- button -> div -> body

`addEventListener`에 세번째 인자를 전달하지 않았으므로 세가지 이벤트 핸들러의 경우 모두 버블링 단계에서 트리거 되고 순서대로 각 요소에 등록된 이벤트 핸들러가 실행됨을 확인할 수 있습니다.

![](/assets/images/post/bubbling.png)

### 이벤트 캡쳐링

이번에는 캡쳐링의 경우를 살펴 보겠습니다.

```html
<!DOCTYPE html>

<html>
  <body>
    <div>
      <button class="btn">Click</button>
    </div>

    <script>
      document.querySelector(".btn").addEventListener(
        "click",
        function() {
          console.log("button!");
        },
        true
      );
      document.querySelector("div").addEventListener(
        "click",
        function() {
          console.log("div!");
        },
        true
      );
      document.querySelector("body").addEventListener(
        "click",
        function() {
          console.log("body!");
        },
        true
      );
    </script>
  </body>
</html>
```

`addEventListener`의 세번째 매개변수를 `true`로 넘기고 캡쳐링 단계에서 이벤트 핸들러를 트리거 하도록 설정하면 다음과 같은 결과를 얻을 수 있습니다.

![](/assets/images/post/capturing.png)

### 버블링과 캡쳐링 동시 사용

아직 완벽히 이해하지 못한 분들을 위해 두 가지 경우를 함께 사용한 예를 한번 더 살펴보겠습니다.

```html
<!DOCTYPE html>

<html>
  <body>
    <div>
      <button class="btn">Click</button>
    </div>

    <script>
      document.querySelector(".btn").addEventListener(
        "click",
        function() {
          console.log("button!");
        },
        true
      );
      document.querySelector("div").addEventListener("click", function() {
        console.log("div!");
      });
      document.querySelector("body").addEventListener(
        "click",
        function() {
          console.log("body!");
        },
        true
      );
    </script>
  </body>
</html>
```

이 코드의 경우 어떻게 동작할까요??  
`body`와 `button`은 캡쳐링 설정을, `div`는 버블링 설정을 해주었습니다. 위에서 말씀드린 이벤트의 전파 순서를 잘 생각해 보시면 답을 예상하실 수 있을 겁니다.

스크롤을 잠시 멈추시고 답을 생각해보세요!!

👇🏻 정답은 아래에 있습니다 👇🏻

<br />
<br />
<br />

---

이벤트는 캡쳐링 단계부터 진행되므로 `body`와 `button`의 이벤트 핸들러가 먼저 실행 될겁니다. 그 후 버블링 단계가 진행되면서 `div`의 이벤트 핸들러가 실행되며 결과는 다음과 같습니다.

![](/assets/images/post/bubble_capture.png)

예상한 답이 맞나요? 틀리셨더라도 포기하지 말고 다시 한 번 천천히 읽어보시면 이해하실 수 있을 겁니다.👏🏻

## 나가며

오늘은 이벤트란 무엇인가, 이벤트의 흐름, 이벤트 전파 방식에 대해 알아보았습니다. 저도 지금까지 습관적으로 이벤트 처리에 대한 코드를 작성해왔는데, 글 작성하기 위해 관련 자료들을 찾아보면서 이벤트에 대해 이론적으로 정리해 볼 수 있는 시간이었던 것 같습니다.

아직 이벤트에 대해 알아 볼 중요한 내용들이 남았지만, 호흡이 너무 길어지는 것 같아 끊어서 연재할까 합니다😂

다음 글에서는 **이벤트 위임(Event Delegation)**, **이벤트 기본 동작의 취소**, **이벤트 전파의 중단** 에 대해 다뤄보겠습니다.

긴 글 읽어주셔서 감사합니다👏🏻

---
title: "[Javascript] 자바스크립트 이벤트 루프"
subtitle: "이벤트 루프, 마이크로 태스크, 매크로 태스크"
categories:
  - Javascript
tags:
  - Javascript
  - 이벤트 루프
classes: wide
---

## 들어가며

자바스크립트를 사용하다보면 가끔 고개를 갸우뚱하게 되는 순간이 있습니다.🧐  
예를 들어 아래와 같은 상황이죠👇🏻

```javascript
function one() {
  console.log("one");
  for (let i = 0; i < 1000000000; i++) {
    // 오래 걸리는 작업
  }
}

function two() {
  console.log("two");
}

setTimeout(() => {
  console.log("setTimeout");
}, 0);

one();
two();
```

코드의 실행 결과는 어떻게 될까요?  
정답은 `one`, `two`, `setTimeout`이 순서대로 출력됩니다😲  
코드의 흐름 상, `setTimeout`, `one`, `two` 함수가 차례로 호출되고, `setTimeout`의 `delay`가 0초로 설정 되었기 때문에 delay 없이 `setTimeout`, `one`, `two` 순으로 출력 되어야 하는게 아닐까요??  
하지만 현실은 그렇지 않답니다😭

오늘은 이런 현상이 왜 일어나는지에 대한 글을 적어보려고 합니다👏🏻

## 자바스크립트의 동작방식

![](/assets/images/post/eventloop1.png){: width="600"}{: .align-center}

### Heap Memory & Call Stack

자바스크립트는 **Call Stack**(호출 스택)이라 불리는 공간에 stack의 형태로 쌓이는 작업들을 하나 하나 처리해 가는 방식으로 실행됩니다.(a.k.a 단일 스레드 기반의 프로그래밍 언어)

예를 들어 아래와 같은 코드가 실행된다고 가정했을 때, 실행 순서대로 코드를 따라가 보면

```javascript
function foo() {
  bar();
  console.log("foo");
}

function bar() {
  console.log("bar");
}

foo();
```

Call Stack에는,  
| bar |  
| foo |  
와 같은 형태로 함수가 쌓이게 됩니다. 그리고 후입선출인 stack의 특성에 따라 상단에 위치한(나중에 들어온) 함수부터 실행되면서 `bar`와 `foo`가 순차적으로 출력됩니다.
이때 메모리로 사용되는 공간이 **Memory Heap**입니다.

### Web API

**Web API**는 브라우저에서 제공하는 API 입니다.  
비동기 호출을 위해 사용하는 함수들이 정의 되어있죠(ex. XHttpRequest, setTimeout, DOM 등등)  
호출 스택에서 Web API 함수가 실행되면, 자바스크립트 엔진은 Web API에 해당 함수를 요청하며 콜백을 넘겨줍니다. 그리고 호출 스택에서 실행한 Web API 함수를 제거합니다. 넘겨진 콜백 함수는 특정 조건이 만족되기를 기다렸다가 **CallBack Queue**로 들어가게 됩니다.

여기서 특정 조건이라 함은 클릭 이벤트가 발생 했을 경우, setTimeout에 지정한 시간이 지났을 경우 등을 의미합니다.

### CallBack Queue & Event Loop

대망의 **Callback Queue**와 **Event Loop**입니다. 여기까지 알게 된다면 처음에 나왔던 예제의 상황을 이해할 수 있습니다🎉

콜백 큐는 이름에 나와있듯이 Queue 형태의 공간입니다. 그리고 이곳엔 비동기로 실행된 함수들이 쌓이게 됩니다. 위에서 들었던 예와 같이 클릭 이벤트에 대한 이벤트 핸들러나 setTimeout의 콜백 함수 등이 이곳으로 들어오게 되죠.

여기서 잠깐!!✋🏻

자바스크립트는 호출 스택에 쌓인 작업들을 하나씩 처리해 가는 방식으로 실행된다고 했죠??
그렇다면 콜백 큐에 쌓인 함수들은 대체 실행 되는 걸까요?!!😫 이게 **핵심**입니다!🚀

콜백 큐에 쌓인 함수들이 실행 될 수 있도록 만들어 주는 것이 바로 **이벤트 루프**입니다.

이벤트 루프는 호출 스택과 콜백 큐를 **감시**합니다

- **호출 스택**에 실행 중인 작업이 있는지
- **콜백 큐**에 대기 중인 작업이 있는지

이 두가지를 끊임없이 살펴보다가, 호출 스택에 **실행 중**인 작업이 없어지면 콜백 큐에 **대기 중**인 작업 하나를 호출 스택으로 슬며시 옮겨놓습니다.
그 이후는 말 안해도 알겠죠??🤖

다시 예제를 살펴 보겠습니다.

### 예제에 적용해보자!

```javascript
function one() {
  console.log("one");
  for (let i = 0; i < 1000000000; i++) {
    // 오래 걸리는 작업
  }
}

function two() {
  console.log("two");
}

setTimeout(() => {
  console.log("setTimeout");
}, 0);

one();
two();
```

지금 까지 알아본 내용을 바탕으로, 코드를 위에서 부터 차근차근 다시 읽어보겠습니다.

1. 호출 스택에서 setTimeout **실행** -> setTimeout 함수 종료
   1-1. Web API 영역에서 타이머가 실행되고 지정한 대기시간 동안 기다립니다  
   1-2. 0초의 대기시간이 지난 후 콜백 함수가 콜백 큐로 **이동**
2. 호출 스택에서 one 함수 **실행**
3. 호출 스택에서 two 함수 **실행**
4. 더 이상 호출 스택에서 실행 중인 함수가 없으므로 콜백 큐에 있는 setTimeout의 콜백 함수가 **이벤트 루프**에 의해 호출 스택으로 **이동**
5. 호출 스택에서 콜백 함수 **실행**

여기서 핵심은 setTimout의 콜백 함수가 호출 스택으로 바로 들어가는 것이 아니라 **Web API 와 콜백 큐를 거쳐 이벤트 루프에 의해 호출 스택**으로 들어간다는 점입니다😲

이런 구조 덕분에 자바스크립트는 단일 스레드 언어임에도 여러 작업을 동시에 할 수 있는 것 처럼 동작합니다💡

> setTimout의 delay가 0초 일때 실제 콜백함수는 0초만 대기하는 것이 아니라고 합니다!  
> 브라우저 마다 타이머의 최소 단위(Tick)를 가지고 있기 때문에 delay가 0초라고 해도 최소 단위 만큼은 대기 해야합니다.  
> ex) 크롬 브라우저의 경우 최소단위 4ms  
> setTimeout(callback, 0) = setTimeout(callback, 4)

> setTimout의 delay는 실행되기 전까지의 대시간이 아니라, 콜백 큐에 들어가기 전까지의 대기시간이라고 할 수 있겠네요!

아래는 이벤트 루프의 동작을 애니메이션으로 쉽게 볼 수 있는 링크입니다. 참고하세요🙃  
[JavaScript Visualized: Event Loop](https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif)  
[Loupe - Philip Roberts](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)

## 매크로 태스트와 마이크로 태스크

여기서 한 단계만 더 나가봅시다💪🏻

아래와 같은 코드는 어떻게 실행 될까요?

```javascript
function one() {
  console.log("one");
  for (let i = 0; i < 1000000000; i++) {
    // 오래 걸리는 작업
  }
}

function two() {
  console.log("two");
}

setTimeout(() => {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("promise1");
  })
  .then(() => {
    console.log("promise2");
  });

one();
two();
```

비동기로 동작하는 setTimeout과 promise의 콜백은 순서대로 콜백 큐로 들어갈 것이기 때문에

```text
one -> two -> setTimeout -> promise1 -> promise2
```

라고 예상해 볼 수 있습니다. 그러나 실제 정답은

```text
one -> two -> promise1 -> promise2 -> setTimeout
```

이런 순서 입니다🤭(크롬 브라우저 기준 -> 브라우저마다 다른 결과를 보입니다)

무슨일이 일어난 걸까요😳

우리가 앞서 알아본 콜백 큐에는 사실 여러 종류의 큐가 있고 이들 사이에는 우선 순위가 존재합니다.

- Task Queue(MacroTask Queue)
- MicroTask Queue
- Animation Frames

```text
* 우선 순위
1. MicoTask Queue
2. Animation Frames
3. Task Queue(MacroTask Queue)
```

이벤트 루프는 호출 스택이 비었을 때 위의 우선 순위에 따라 우선적으로 호출 스택으로 이동해야하는 작업을 선별합니다.

이미 눈치 채신 분이 있겠죠??😲  
네 맞습니다. 여러분이 짐작하셨듯이 `setTimout`의 경우 **MacroTask**로 Task Queue에 쌓이게 되고, `promise then`의 콜백 함수는 **MicroTask**로 MicroTask Queue에 쌓이게 된답니다.

예제에서 `setTimout` -> `promise`의 순으로 실행되었지만 Queue 사이의 우선 순위에 따라 `then`의 콜백 함수가 먼저 실행 되고 `setTimeout`의 콜백 함수가 나중에 실행 되는 것입니다!

> 각 우선순위에 따라 MicroTask 수행 후 화면 랜더링, MacroTask 의 순으로 수행 된다고 합니다!

- MicroTask Queue 사용
  - Mutationobserver, promise callback, queueMicrotask를 이용한 함수
- Animation Frames 사용
  - requestAnimationFrame
- MacroTask Queue 사용
  - setTimeout
  - 이벤트 핸들러
  - `<script src="...">` 의 로딩과 실행

## 나가며

오늘은 이벤트 루프를 기반으로 한 자바스크립트의 동작 방식에 대해 알아보았습니다🚀

처음에는 Vue의 `nextTick`에 대한 글을 써보고 싶었는데, `nextTick`을 이해하기 위해선 MacroTask, MicroTask, EventLoop 등의 용어에 대해 꼭 알아야겠더라구요😳

자바스크립트를 이해하기 위해 꼭 필요한 내용 중 하나이므로 반드시 이해하고 넘어갑시다👏🏻  
(참고에 적은 링크들은 제가 이번 개념을 이해하는데 많은 도움이 된 아주 아주 좋은 글들입니다!! 시간 내서 읽어보시길 강추드립니닷!)

참고 하단에 부록 파트도 읽어보세요,,,ㅎ  
꼬리에 꼬리를 무는 호기심으로 async/await에 대한 Event Loop 의 동작을 테스트해보다가 이해하는데 시간이 꽤 걸렸네요😅
혹시 틀린 내용이 있다면 부족한 저에게 지식을 나눠주십쇼...👍🏻

## 참고

[https://ko.javascript.info/event-loop](https://ko.javascript.info/event-loop)  
[https://jakearchibald.com](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)  
[https://meetup.toast.com/posts/89](https://meetup.toast.com/posts/89)  
[https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)  
[https://velog.io/@thms200](https://velog.io/@thms200/Event-Loop-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84)  
[https://engineering.huiseoul.com](https://engineering.huiseoul.com/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84%EC%99%80-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%9D%98-%EB%B6%80%EC%83%81-async-await%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%BD%94%EB%94%A9-%ED%8C%81-%EB%8B%A4%EC%84%AF-%EA%B0%80%EC%A7%80-df65ffb4e7e)
[https://hudi.kr](https://hudi.kr/%EB%B9%84%EB%8F%99%EA%B8%B0%EC%A0%81-javascript-%EC%8B%B1%EA%B8%80%EC%8A%A4%EB%A0%88%EB%93%9C-%EA%B8%B0%EB%B0%98-js%EC%9D%98-%EB%B9%84%EB%8F%99%EA%B8%B0-%EC%B2%98%EB%A6%AC-%EB%B0%A9%EB%B2%95/)

## 부록

아래 예제는 어떻게 동작 할 지 생각해 봅시다🤩

```javascript
function one() {
  console.log("one");
}

function two() {
  console.log("two");
}

function prom() {
  return new Promise((resolve, reject) => {
    console.log("inner-promise");
    resolve();
  });
}

async function ascynchronous() {
  console.log("pre-promise");
  await prom();
  console.log("post-promise");
}

one();

setTimeout(() => {
  console.log("timeout");
}, 0);

ascynchronous();

two();
```

정답은 바로!! 👇🏻

```text
one
pre-promise
inner-promise
two
post-promise
timeout
```

동작 방식을 자세히 뜯어 봅시다. 조금 복잡하지만 한 단계씩 따라가 보세요👍🏻

1. one 함수 실행, `one` 출력
2. setTimeout 콜백 함수는 Web API에서 타이머를 통해 0초 대기 후 MacroTask Queue로 들어갑니다
3. ascynchronous 함수 실행  
   3-1. `pre-promise` 출력  
   3-2. prom 함수 실행  
   3-3. Promise의 Executor(Promise 생성자에 전달되는 콜백)는 생성과 동시에 내용이 실행됩니다  
   3-4. `inner-promise` 출력  
   3-5. resolve 함수 호출하며 fullfilled 상태의 promise 객체 리턴  
   3-6. prom 함수가 promise 객체를 리턴하며 `await` 키워드를 만나게 되면 해당 `async` 함수 **전체**가 MicroTask Queue로 들어갑니다.
4. two 함수 실행, `two` 출력
5. 호출 스택 비어있으므로 MicroTask Queue에 있던 ascynchronous 함수 Event loop에 의해 호출 스택으로 이동  
   5-1. **await 우측의 함수(prom 함수)가 fullfilled 상태인 경우에만 호출 스택으로 이동합니다.**  
   5-2. 이 경우 바로 resolve 함수를 호출하여 fullfilled 상태이므로 await 이후의 코드를 실행합니다.  
   5-3. `post-promise` 출력
6. 호출 스택과 MicroTask Queue가 모두 비었으므로 MacroTask Queue에 있던 setTimeout의 콜백 함수가 호출 스택으로 이동
   6-1. `timeout` 출력

---

이제 진짜 진짜 마지막입니다. 정답을 맞춰 보세요!  
정답은 아래 공개👇🏻

```javascript
function one() {
  console.log("one");
}

function two() {
  console.log("two");
}

function prom() {
  return new Promise((resolve, reject) => {
    console.log("inner-promise");
    // --- 2
    setTimeout(() => {
      resolve(10);
    }, 0);
  });
}

async function ascynchronous() {
  console.log("pre-promise");
  let result = await prom();
  console.log(result);
}

one();

// --- 1
setTimeout(() => {
  console.log("timeout");
}, 0);

ascynchronous();
two();
```

과연 정답은.....???

```javascript
// 미리보기 방지....🗿
// javascript
// event loop
// 어렵다 정말
// 그래도 거의 다왔다
// 사실 이게 끝이 아닐거야....
```

💡답지 공개💡

```text
one
pre-promise
inner-promise
two
timeout
10
```

이게 무슨 일이죠??  
MacroTask의 결과인 `timeout`이 MicroTask인 promise의 결과 보다 먼저 출력됐네요???

이전 예제의 순서를 잘 따라오셨다면 눈치채셨을 수 있습니다😎

이번 예제의 경우는 async 함수 내부에 `resolve`가 `setTimeout`로 감싸져있죠??

그렇다면 코드의 흐름 상 1번의 setTimeout이 2번(async 내부의 setTimeout) 보다 먼저 MacroTask Queue에 들어갔을 것이고, 호출 스택이 비어 MicroTask Queue에 있는 ascynchronous 함수를 호출 스택으로 올리려는 시점에서 promise 객체의 상태는 아직 **pending** 일 것입니다

MicroTask Queue에 있는 async 함수는 promise 객체가 resolve 된 것을 확인 한 후에 호출 스택으로 갈 수 있다고 했으므로 MacroTask Queue에 있는 setTimeout의 콜백 함수들이 먼저 실행 됩니다.

결과적으로 `timeout`이 먼저 출력되고,  
그 뒤에 있던 2번 setTimeout 콜백 함수가 실행되면서 promise 객체가 fullfilled 상태로 변합니다. 그리고 ascynchronous의 마지막 `console.log`를 실행하면서 전체 코드가 종료 됩니다.

아래는 이런 저의 테스트 코드를 이해하는데 도움이 된 글입니다🧐

[https://medium.com/@lydiahallie](https://medium.com/@lydiahallie/javascript-visualized-promises-async-await-a3f1aad8a943)  
[https://soobakba.tistory.com/48](https://soobakba.tistory.com/48)
[https://ko.javascript.info/promise-basics](https://ko.javascript.info/promise-basics)

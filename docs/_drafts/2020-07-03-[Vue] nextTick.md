---
title: "[Vue] nextTick"
subtitle: "Vue nextTick"
categories:
  - Vue
tags:
  - Vue
classes: wide
---

## 들어가며

Vue로 개발을 하다보면 종종 데이터 변경과 화면 갱신을 통해 무언가를 해야하는 상황에 놓이게 됩니다.
예를 들면 아래와 같은 상황이 있을 수 있죠

```javascript
<div id="app">
  <div v-if="isLoading">Loading...</div>
  <div v-if="done">DONE</div>
  <button v-on:click="loading">COUNT</button>
</div>;

var app = new Vue({
  el: "#app",
  data: {
    isLoading: false,
    done: false,
    count: 0,
  },
  methods: {
    loading() {
      this.isLoading = true;
      this.$nextTick(() => {
        for (var i = 1; i < 10000000; ++i) {
          // 무언가 오래걸리는 작업....
          this.count = i;
        }
        this.done = true;
        this.isLoading = false;
      });
    },
  },
});
```

위 코드는 어떤 식으로 동작할까요? 코드의 의도는 다음과 같습니다.

1. 버튼을 클릭하면 `isLoading`을 `true`로 바꾼다.
2. `isLoading`가 `true`로 바뀜에 따라 화면에는 'Loding...'이라는 글씨가 표시된다.
3. `for`문 내부의 작업이 완료 된 후 `isLoading`과 `done`을 각각 `false`와 `true`로 변경한다.
4. 'DONE'이 화면에 출력되며 종료

이제 실행해보겠습니다.

이미지 -> nexttick1.gif

뭔가 이상하죠?! 화면에 출력되어야 할 'Loading...'이라는 글씨가 보이지 않습니다. 이런 결과가 나온 이유는 'nextTick' 메소드에 대한 이해가 부족했기 때문입니다.

## nextTick?

예제에서 사용한 `nextTick`이란 무엇일까요?

nextTick의 동작을 이해하려면 아래 두가지에 대해 알아야합니다.

- 이벤트 루프와 microtask
- Vue의 랜더링 프로세스(feat.virtualDOM)

// -> 테스트코드에서 DOM 감시자로 테스트 해본 결과 내가 의심 한 것 처럼 예제 코드에서는 nextTick 밖에서 한번, 안에서 한번 DOM 업데이트가 일어난다. 그런데 왜 동시에 일어나냐 이건 모르겟다 내 예상은 밖에서 먼저 일어나고 for문 돌고 안에서 일어나고 이런식인데 실제로는 for문 다 돌고 한꺼번에 두개가 업데이트 쳐진다...?

-> 생각해본 결과 맞는 거 같다

1. this.Loading 변경 -> Virtual Dom 반영을 위해 watcher가 큐에 들어감.
2. DOM에 반영 -> micro task
   - virtual dom 의 작업이 마이크로 테스트라고 본 것 같은데 맞는지 확인 요망
3. Mutation은 WebAPI 기 떄문에 WebAPI -> macro task queue로 들어감
   - 찾아보니 Mutation은 microtask라는 것 같다
   - 그렇다면 이렇게 된 이유는 loading을 변경하고 nexttick이 호출되고(호출스택으로) 콜백함수가 micro에 먼저 들어가고 뒤따라서 mutation이 micro로 들어간다 이런 식인듯?
4. DOM update 종료 됐으므로 nextTick이 call stack으로 -> nextTick은 microtask
   - this.loading 변경되어도 다음 실행 코드인 nextTick이 실행되기 전 그 사이에 이벤트 루프라 큐에있는 작업 가져와야하는 거 아니냐는 생각이 있었지만 스택에 쌓일 작업이 계속 있으면 스택부터 쌓는 것 같다!
   - eventtest.html 참고s
5. for문 실행
6. new Date 콘솔 로그
7. this.Loading 변경 -> WebAPI -> macro task queue push
8. microtask, call stack 전부 비게됨
9. macrotask에 들어있는 mutation 이벤트 루프에 의해 콜스택으로 이동
10. 3번, 7번 순서로 콘솔 로그

-> 다시 총 정리해보면 이런 것인듯
일단 dom patching , mutation , nextTick 다 microstack

1. this.Loading 변경
2. microstack에 dom patching 들어감
   - https://forum.vuejs.org/t/does-nexttick-work-weirdly/42918/6 여기 답변에 돔패칭 마이크로테스크라고함
3. nextTick 실행
4. microstack에 nextTick 콜밸 들어감
5. microstack에 있던 dom patching 실행
6. mutaion 이 microtask에 들어감
7. nextTick의 콜밸함수 microtask에서 나와서 실행
8. this.Loading 변경 -> microstack에 dom patching 들어감
9. microtask 큐에 있던 mutation 나와서 콘솔 찍음
10. nextTick 콜백 안에서 발생한 dom patching 실행
11. mutation -> microtask에 들어감
12. microstack에 있는 마지막 mutation 나와서 실행됨

nextTick -> 화면 랜더링을 보장하는 것 아님, dom 변경을 보장! dom 변경 후 repaint 일어나기 전 nextTick의 콜백 내용이 실행된
후에야 화면 repaint가 일어남.

결국 생각해보면 dom변경을 보장하는게 뭔가 nextTick의 특별한 기능인게 아니라!!!
뷰 자체의 dom patching이 microtask 이기 때문에 nextTick은 그냥 콜백 함수를 마이크로 테스크로 만들어서 무조건 앞서 들어간 돔패칭 뒤에 콜백이 실행되도록 해주는 놈!
마이크로 테스크보다 화면 랜더링이 후순위이기때문에 당연히 넥스틱의 콜백이 실행 된 후에 repaint가 일어나는 것뿐...
그럼 queueMicrotask 이 함수를 써도 당연히 똑같은 결과를 볼 수 있겠지
-> vue.html에 실험해보니 정답!ㅋㅋㅋㅋㅋㅋ
그럼 nextTick은 그냥 queueMicrotask를 랩핑한것 뿐일까??? -> 소스 찾아보니까 그건 아닌 것 같다...!
오 -> https://meetup.toast.com/posts/89
여기 보다가 안건데
MutationObserver는 DOM의 변화를 감지할 수 있게 해 주는 클래스이며, es6-promise와 같은 폴리필에서 마이크로 태스크를 구현하기 위해 사용되기도 한다.
요렇다네!! 그리고 실제로 vue의 next tick 소스에 MutationOberver를 사용했다!!

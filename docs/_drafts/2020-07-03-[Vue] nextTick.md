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
3. Mutation은 WebAPI 기 떄문에 WebAPI -> macro task queue로 들어감
4. DOM update 종료 됐으므로 nextTick이 call stack으로 -> nextTick은 microtask
5. for문 실행
6. new Date 콘솔 로그
7. this.Loading 변경 -> WebAPI -> macro task queue push
8. microtask, call stack 전부 비게됨
9. macrotask에 들어있는 mutation 이벤트 루프에 의해 콜스택으로 이동
10. 3번, 7번 순서로 콘솔 로그

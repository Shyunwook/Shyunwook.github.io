---
title: "2020 KAKAO BLIND RECRUITMENT - 블록이동하기"
subtitle: "2020 KAKAO BLIND RECRUITMENT - 블록이동하기"
categories:
  - 코딩테스트
tags:
  - BFS
  - 코딩테스트
  - 카카오
classes: wide
---

## 들어가며

이번에 도전한 문제는 2020년 카카오 블라인드 채용에 등장했던 문제입니다.
이 문제는 깊이 우선 탐색(DFS), 너비 우선 탐색(BFS)에 대한 지식이 있다면 쉽게 **시작해**볼 수 있는 문제입니다.

해당 알고리즘에 대한 공부가 부족한 분들은 DFS와 BFS에 대해 추가적으로 공부를 하신 후 시도하시는 것을 추천드립니다.

저도 처음에는 손도 못댔다가 알고리즘 공부 후 푸는데 성공했거든요!🤓
(알고리즘에 대한 공부가 너무 부족하다는 생각에 문제를 풀어보고 있지만 이런 문제들을 만날때 마다 좌절하게 되는 것 같습니다😭)

제가 이 문제를 풀기 위해 공부했던 다른 문제들과 참고글을 공유드릴게요!

#### 관련 문제

[Programmers - 타겟 넘버](https://programmers.co.kr/learn/courses/30/lessons/43165)  
[Programmers - 네트워크](https://programmers.co.kr/learn/courses/30/lessons/43162)  
[Programmers - 단어 변환](https://programmers.co.kr/learn/courses/30/lessons/43163)  
[Programmers - 여행 경로](https://programmers.co.kr/learn/courses/30/lessons/43164)

#### 알고리즘 개념 참고 자료(Javascript)

[설영환님의 블로그 - 그래프/깊이, 너비 우선 탐색](https://velog.io/@maintainker/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EA%B7%B8%EB%9E%98%ED%94%84-%EA%B9%8A%EC%9D%B4%EB%84%88%EB%B9%84-%EC%9A%B0%EC%84%A0%ED%83%90%EC%83%89)

## 문제

#### 2020 KAKAO BLIND RECRUITMENT - 블록 이동하기

[https://programmers.co.kr/learn/courses/30/lessons/60063](https://programmers.co.kr/learn/courses/30/lessons/60063)

## 접근 방식

문제 자체의 접근 방식은 간단합니다.  
우선 시작점에서 도착점 까지의 최단 거리를 구해야 하기 때문에 BFS를 사용하여 해결 할 수 있습니다.

로봇의 이동 방식에 대한 특이한 제한 조건과 차지하는 칸이 두 칸이라는 점 등을 제외한다면 일반적인 BFS 문제라고 볼 수 있습니다.
제한 사항에 주의하여 다음 이동 경로를 구하는 로직을 잘 설계해주어야 합니다😫

#### 로봇의 위치

문제를 해결함에 있어 까다로운 점이 로봇이 두 칸을 차지한다는 점인데요, 저같은 경우는 로봇의 위치를 다음과 같이 표현하였습니다.

`[꼬리의 x좌표, 꼬리의 y좌표, 머리의 방향]`

머리의 방향은 양의 Y 방향, 음의 Y 방향, 양의 X 방향, 음의 X 방향 네 가지가 있습니다.

![](/assets/images/post/block-ex.png){: width="500"}{: .align-center}

예를들어 위와 같이 시작 상황에서는 꼬리가 (0,0)의 좌표에 위치하고 머리가 오른쪽 방향이기 때문에
로봇의 위치를 [0,0,Y] 라고 표현 할 수 있는 것이죠!
(반대로 머리가 왼쪽을 향해 있다면 [0,0,-Y] 겠죠?! 🧐)

## 구현

BFS로 경로를 탐색하기 위해 다음의 준비가 필요합니다.

- 다음 탐색지를 저장할 **큐**(배열을 사용해 구현하였습니다)
- 방문 여부를 저장하기 위한 공간([Map](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Map)을 사용해 구현하였습니다.)
- 로봇이 상하좌우 직선으로 이동할 수 있는 경로(위치)를 구하는 로직
- 로봇이 회전하여 이동할 수 있는 경로(위치)를 구하는 로직

BFS로 최단 거리를 구하는 로직의 큰 흐름을 살펴보면 아래와 같습니다.

1. 로봇의 현재 위치를 큐에 넣는다. `[꼬리의 x 좌표, 꼬리의 y 좌표, 머리의 방향, 이동 횟수(시간)]`
2. 로봇의 현재 위치를 방문 여부에 세팅한다.
3. 경로 탐색 시작
   1. 큐의 제일 앞에 있는 경로 꺼내 **현재 경로**로 세팅한다.
   2. **현재 경로**에서 이동할 수 있는 위치, 회전할 수 있는 위치를 구해 `next` 배열에 넣는다.
   3. `next` 배열에 들어있는 다음 경로 중 도착지가 있다면 현재 카운트에 1을 더한 값을 반환하며 탐색을 종료한다.(1을 더하는 이유는 조건을 만족하는 경로가 **다음** 위치이기 때문에 한 번 더 이동해야 해당 위치에 도착하는 것이기 때문입니다👍🏻)
   4. 3 과정에서 다음 경로 중 도착지가 없었다면 큐에 `next` 배열에 있는 다음 경로들을 집어넣고 방문 여부를 체크한다.
   5. 로봇은 항상 목적지에 도착할 수 있기 때문에 탐색 중 목적지가 나올 떄 가지 위 단계를 반복하여 수행한다.

위 과정에서 머리가 아팠던 부분은 로봇이 다음에 이동, 회전할 수 있는 위치를 구하는 부분이었는데요 저는 이 두 부분을 각각 함수로 따로 빼서 구현하였습니다.

```javascript
  // 상하좌우 이동 가능한 위치 구하는 함수
  function straightRoute(x, y, d) {
    let next = [];

    if (d === Y) {
      // 머리 오른쪽 방향
      if (board[x + 1] && board[x + 1][y] === 0 && board[x + 1][y + 1] === 0) {
        // 아래로 한 칸
        next.push([x + 1, y, d]);
      }
      if (board[x][y + 2] === 0) {
        // 오른쪽으로 한 칸
        next.push([x, y + 1, d]);
      }
      if (board[x - 1] && board[x - 1][y] === 0 && board[x - 1][y + 1] === 0) {
        // 위로 한 칸
        next.push([x - 1, y, d]);
      }
      if (board[x][y - 1] === 0) {
        // 왼쪽으로 한 칸
        next.push([x, y - 1, d]);
      }
    } else if (d === -Y) {
      //머리 왼쪽 방향
      if (board[x + 1] && board[x + 1][y] === 0 && board[x + 1][y - 1] === 0) {
        // 아래로 한 칸
        next.push([x + 1, y, d]);
      }
      if (board[x][y + 1] === 0) {
        // 오른쪽으로 한 칸
        next.push([x, y + 1, d]);
      }
      if (board[x - 1] && board[x - 1][y] === 0 && board[x - 1][y - 1] === 0) {
        // 위로 한 칸
        next.push([x - 1, y, d]);
      }
      if (board[x][y - 2] === 0) {
        // 왼쪽으로 한 칸
        next.push([x, y - 1, d]);
      }
    } else if (d === X) {
      // 머리 아래 방향
      if (board[x + 2] && board[x + 2][y] === 0) {
        // 아래로 한 칸
        next.push([x + 1, y, d]);
      }
      if (board[x][y + 1] === 0 && board[x + 1][y + 1] === 0) {
        // 오른쪽으로 한 칸
        next.push([x, y + 1, d]);
      }
      if (board[x - 1] && board[x - 1][y] === 0) {
        // 위로 한 칸
        next.push([x - 1, y, d]);
      }
      if (board[x][y - 1] === 0 && board[x + 1][y - 1] === 0) {
        // 왼쪽으로 한 칸
        next.push([x, y - 1, d]);
      }
    } else if (d === -X) {
      // 머리 위
      if (board[x + 1] && board[x + 1][y] === 0) {
        // 아래로 한 칸
        next.push([x + 1, y, d]);
      }
      if (board[x][y + 1] === 0 && board[x - 1][y + 1] === 0) {
        // 오른쪽으로 한 칸
        next.push([x, y + 1, d]);
      }
      if (board[x - 2] && board[x - 2][y] === 0) {
        // 위로 한 칸
        next.push([x - 1, y, d]);
      }
      if (board[x][y - 1] === 0 && board[x - 1][y - 1] === 0) {
        // 왼쪽으로 한 칸
        next.push([x, y - 1, d]);
      }
    }

    return next;
  }

  // 회전할 수 있는 위치 구하는 함수
  function rotateRoute(x, y, d) {
    let next = [];
    if (d === Y) {
      if (board[x - 1] && board[x - 1][y] === 0 && board[x - 1][y + 1] === 0) {
        //위쪽 방향
        next.push([x - 1, y + 1, X]); // 머리축, 시계 방향
        next.push([x, y, -X]); // 꼬리축, 반시계 방향
      }
      if (board[x + 1] && board[x + 1][y] === 0 && board[x + 1][y + 1] === 0) {
        // 아래쪽 방향
        next.push([x + 1, y + 1, -X]); // 머리축, 반시계 방향
        next.push([x, y, X]); // 꼬리축, 시계 방향
      }
    } else if (d === -Y) {
      if (board[x - 1] && board[x - 1][y] === 0 && board[x - 1][y - 1] === 0) {
        // 위쪽 방향
        next.push([x - 1, y - 1, X]); // 머리축, 반시계 방향
        next.push([x, y, -X]); // 꼬리축, 시계 방향
      }
      if (board[x + 1] && board[x + 1][y] === 0 && board[x + 1][y - 1] === 0) {
        // 아래쪽 방향
        next.push([x + 1, y - 1, -X]); // 머리축, 시계 방향
        next.push([x, y, X]); // 꼬리축, 반시계 방향
      }
    } else if (d === X) {
      if (board[x][y + 1] === 0 && board[x + 1][y + 1] === 0) {
        // 오른쪽 방향
        next.push([x + 1, y + 1, -Y]); // 머리축, 시계 방향
        next.push([x, y, Y]); // 꼬리축, 반시계 방향
      }
      if (board[x][y - 1] === 0 && board[x + 1][y - 1] === 0) {
        // 왼쪽 방향
        next.push([x + 1, y - 1, Y]); // 머리축, 반시계 방향
        next.push([x, y, -Y]); // 꼬리축, 반시계 방향
      }
    } else if (d === -X) {
      if (board[x][y + 1] === 0 && board[x - 1][y + 1] === 0) {
        // 오른쪽 방향
        next.push([x - 1, y + 1, -Y]); // 머리축, 반시계 방향
        next.push([x, y, Y]); // 꼬리축, 시계 방향
      }
      if (board[x][y - 1] === 0 && board[x - 1][y - 1] === 0) {
        // 왼쪽 방향
        next.push([x - 1, y - 1, Y]); // 머리축, 시계 방향
        next.push([x, y, -Y]); // 꼬리축, 반시계 방향
      }
    }

    return next;
  }
}
```

너무 복잡해 보이지만 조건이 많아서 그렇지 금방 이해하실 수 있을겁니다🤩  
위 로직만 잘 짜신다면 그 외의 나머지 부분은 다른 BFS 문제와 크게 다를 것이 없습니다

🔥코드의 전문은 제일 하단에 넣어드릴게요🔥

## 나가며

처음에는 이런 문제를 어떻게 풀어🤯라며 울상을 지었던 문제였는데 이렇게 혼자 풀 수 있게 되서 너무 기쁘네요!
이런게 코딩의 매력이 아닐까 싶습니다. 제가 프로그래밍을 좋아하는 이유중에 하나기도 하구요👏🏻

알고리즘을 공부하는게 많이 어렵긴 하지만 이렇게 하나하나 뿌셔가는 맛이 있으니 중간에 포기하지말고 꾸준히 해야겠습니다👍🏻

끝까지 읽어주신 여러분 모두 화이팅입니다!

## 전체 코드

```javascript
const X = 1;
const Y = 2;

function solution(board) {
  const ARRIVAL_X = board.length - 1;
  const ARRIVAL_Y = board[0].length - 1;

  let queue = [];
  let visited = new Map();
  let start = [0, 0, Y, 0]; // [꼬리의 x 좌표, 꼬리의 y 좌표, 머리의 방향, 이동 횟수(시간)]

  visited.set(start.slice(0, 3).join(), 0);
  queue.push(start);

  while (queue.length > 0) {
    let current = queue.shift();
    let location = current.slice(0, 3);
    let count = current[3];

    let next = [
      ...straightRoute(location[0], location[1], location[2]),
      ...rotateRoute(location[0], location[1], location[2]),
    ];

    for (let i = 0; i < next.length; i++) {
      // 로봇의 일부분이 도착지에 들어갔나 검사하는 부분
      if (
        next[i].join() === [ARRIVAL_X, ARRIVAL_Y, -X].join() ||
        next[i].join() === [ARRIVAL_X - 1, ARRIVAL_Y, X].join() ||
        next[i].join() === [ARRIVAL_X, ARRIVAL_Y, -Y].join() ||
        next[i].join() === [ARRIVAL_X, ARRIVAL_Y - 1, Y].join()
      ) {
        console.log(count + 1);
        return count + 1;
      }

      if (!visited.has(next[i].join())) {
        visited.set(next[i].join(), count + 1);

        next[i].push(count + 1);
        queue.push(next[i]);
      }
    }
  }

  function straightRoute(x, y, d) {
    let next = [];

    if (d === Y) {
      // 머리 오른쪽 방향
      if (board[x + 1] && board[x + 1][y] === 0 && board[x + 1][y + 1] === 0) {
        // 아래로 한 칸
        next.push([x + 1, y, d]);
      }
      if (board[x][y + 2] === 0) {
        // 오른쪽으로 한 칸
        next.push([x, y + 1, d]);
      }
      if (board[x - 1] && board[x - 1][y] === 0 && board[x - 1][y + 1] === 0) {
        // 위로 한 칸
        next.push([x - 1, y, d]);
      }
      if (board[x][y - 1] === 0) {
        // 왼쪽으로 한 칸
        next.push([x, y - 1, d]);
      }
    } else if (d === -Y) {
      //머리 왼쪽 방향
      if (board[x + 1] && board[x + 1][y] === 0 && board[x + 1][y - 1] === 0) {
        // 아래로 한 칸
        next.push([x + 1, y, d]);
      }
      if (board[x][y + 1] === 0) {
        // 오른쪽으로 한 칸
        next.push([x, y + 1, d]);
      }
      if (board[x - 1] && board[x - 1][y] === 0 && board[x - 1][y - 1] === 0) {
        // 위로 한 칸
        next.push([x - 1, y, d]);
      }
      if (board[x][y - 2] === 0) {
        // 왼쪽으로 한 칸
        next.push([x, y - 1, d]);
      }
    } else if (d === X) {
      // 머리 아래 방향
      if (board[x + 2] && board[x + 2][y] === 0) {
        // 아래로 한 칸
        next.push([x + 1, y, d]);
      }
      if (board[x][y + 1] === 0 && board[x + 1][y + 1] === 0) {
        // 오른쪽으로 한 칸
        next.push([x, y + 1, d]);
      }
      if (board[x - 1] && board[x - 1][y] === 0) {
        // 위로 한 칸
        next.push([x - 1, y, d]);
      }
      if (board[x][y - 1] === 0 && board[x + 1][y - 1] === 0) {
        // 왼쪽으로 한 칸
        next.push([x, y - 1, d]);
      }
    } else if (d === -X) {
      // 머리 위
      if (board[x + 1] && board[x + 1][y] === 0) {
        // 아래로 한 칸
        next.push([x + 1, y, d]);
      }
      if (board[x][y + 1] === 0 && board[x - 1][y + 1] === 0) {
        // 오른쪽으로 한 칸
        next.push([x, y + 1, d]);
      }
      if (board[x - 2] && board[x - 2][y] === 0) {
        // 위로 한 칸
        next.push([x - 1, y, d]);
      }
      if (board[x][y - 1] === 0 && board[x - 1][y - 1] === 0) {
        // 왼쪽으로 한 칸
        next.push([x, y - 1, d]);
      }
    }

    return next;
  }
  function rotateRoute(x, y, d) {
    let next = [];
    if (d === Y) {
      if (board[x - 1] && board[x - 1][y] === 0 && board[x - 1][y + 1] === 0) {
        //위쪽 방향
        next.push([x - 1, y + 1, X]); // 머리축, 시계 방향
        next.push([x, y, -X]); // 꼬리축, 반시계 방향
      }
      if (board[x + 1] && board[x + 1][y] === 0 && board[x + 1][y + 1] === 0) {
        // 아래쪽 방향
        next.push([x + 1, y + 1, -X]); // 머리축, 반시계 방향
        next.push([x, y, X]); // 꼬리축, 시계 방향
      }
    } else if (d === -Y) {
      if (board[x - 1] && board[x - 1][y] === 0 && board[x - 1][y - 1] === 0) {
        // 위쪽 방향
        next.push([x - 1, y - 1, X]); // 머리축, 반시계 방향
        next.push([x, y, -X]); // 꼬리축, 시계 방향
      }
      if (board[x + 1] && board[x + 1][y] === 0 && board[x + 1][y - 1] === 0) {
        // 아래쪽 방향
        next.push([x + 1, y - 1, -X]); // 머리축, 시계 방향
        next.push([x, y, X]); // 꼬리축, 반시계 방향
      }
    } else if (d === X) {
      if (board[x][y + 1] === 0 && board[x + 1][y + 1] === 0) {
        // 오른쪽 방향
        next.push([x + 1, y + 1, -Y]); // 머리축, 시계 방향
        next.push([x, y, Y]); // 꼬리축, 반시계 방향
      }
      if (board[x][y - 1] === 0 && board[x + 1][y - 1] === 0) {
        // 왼쪽 방향
        next.push([x + 1, y - 1, Y]); // 머리축, 반시계 방향
        next.push([x, y, -Y]); // 꼬리축, 반시계 방향
      }
    } else if (d === -X) {
      if (board[x][y + 1] === 0 && board[x - 1][y + 1] === 0) {
        // 오른쪽 방향
        next.push([x - 1, y + 1, -Y]); // 머리축, 반시계 방향
        next.push([x, y, Y]); // 꼬리축, 시계 방향
      }
      if (board[x][y - 1] === 0 && board[x - 1][y - 1] === 0) {
        // 왼쪽 방향
        next.push([x - 1, y - 1, Y]); // 머리축, 시계 방향
        next.push([x, y, -Y]); // 꼬리축, 반시계 방향
      }
    }

    return next;
  }
}
```

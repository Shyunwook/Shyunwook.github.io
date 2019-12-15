---
title: "X of a kind in a Deck of Cards (Leetcode No.914)"
subtitle: "leetcode 문제 풀기"
categories:
  - LeetCode
tags:
  - LeetCode
  - Easy
classes: wide
---

## 들어가며

오늘은 Easy 난이도의 문제를 풀어보았습니다!! 사실 말이 'Easy' 난이도지 저에게는 굉장히 어렵게 느껴졌답니다😂 하루 빨리 이런 문제가 쉽게 느껴지는 날이 오길 바라며... 문제 풀이를 해보려 합니다💪

## 문제 설명

정수가 적힌 카드의 덱이 있습니다.  
이 덱을 아래 조건에 해당하는 하나 혹은 그 이상의 그룹으로 나눌 수 있을 때 `true`를 반환합니다.

- 각 그룹은 동일한 정수가 적힌 카드들로 이루어져 있습니다.
- 각 그룹을 이루는 카드의 갯 수(X)는 모든 그룹이 동일해야 합니다.(단 X >= 2)

### Example

- input : [1,2,3,4]
- output : false
- Explanation : [1], [2], [3], [4] 로 각 하나의 카드가 포함된 그룹으로 나눌 수 있으나 2개 이상의 카드로 이루어져 있지 않음

---

- input : [1,2,3,4,4,3,2,1]
- output : true
- Explanation : [1,1],[2,2],[3,3],[4,4]

---

- input : [1,1,2,2,2,2]
- output : true
- Explanation : [1,1],[2,2],[2,2]

---

- input : [1,1,1,2,2,2,3,3]
- output : false
- Explanation : 각 그룹을 동일한 갯 수로 묶을 수 없습니다.

## 접근 방법

1. 각 카드(숫자)의 갯 수 세기
1. 각 카드의 수를 동일하게 나눌 수 있는 X(공약수)의 존재 유무 파악
1. `X >= 2` 이면 `true` 아니면 `false`, 즉 최대 공약수가 2 이상 일 때!

## 코드(javascript)

```javascript
//최대 공약수
let gcd = (a, b) => (b === 0 ? a : gcd(b, a % b));

var hasGroupsSizeX = function(deck) {
  //덱의 갯 수가 1 이하일 때
  if (deck.length < 2) return false;

  //각 카드의 갯 수를 구하는 로직
  // Input : [1,1,2,2,3,3]
  // count 형태 : { 1 : 2, 2 : 2, 3 : 3}
  let count = deck.reduce((acc, val) => {
    acc[val] = (acc[val] || 0) + 1;
    return acc;
  }, {});

  let x;
  let val = Object.values(count);

  // 각 카드 수와 x의 최대 공약수를 구합니다.
  for (let i in val) {
    if (x === undefined) {
      x = val[i];
      continue;
    }

    x = gcd(x, val[i]);
    if (x < 2) return false;
  }

  if (x >= 2) return true;
};
```

## 코드 설명

- 각 카드 숫자의 갯 수를 `reduce`를 이용하여 `Object`의 형태로 만들어줍니다.
- 위 단계에서 구한 `Object`의 `value`에 대해 반복문을 돌며 현재 X와 다음 숫자의 최대 공약수를 구합니다.
  - n개의 숫자에서 최대 공약수를 구하는 과정이라고 보시면 됩니다.
- 결과적으로 전체 각 카드 숫자 갯 수의 최대 공약수, X를 구하게 되고 X가 2이상일 때 `true`를 `return`합니다.

## 나가며

서론에 말씀드렸듯이 제게는 굉장히 어려운 문제였습니다😂😂😂 접근 방법까지는 무사히 떠올릴 수 있었지만 최대 공약수를 구하는 로직을 처음 경험해 봤거든요...🤩 이 알고리즘을 `유클리드 호제법`, `유클리드 알고리즘`이라고 부르더라구요!! 그리고 위 코드에선 `reduce`를 이용하여 객체를 깔끔하게 생성했지만 실제로는 직접 `for`문을 돌리는 방식으로 코딩했었습니다! 아직 `reduce`함수를 완전히 활용하지 못한다는 증거겠죠ㅎㅎ 아직 갈길이 먼 것 같습니다. 많이 부족한 글이지만 재밋는 시간이 되셨길 바랍니다~👋🏻

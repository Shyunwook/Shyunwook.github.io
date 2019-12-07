---
title: "Can Place Flowers"
categories:
  - LeetCode
tags:
  - LeetCode
---

# javascript

```javascript
var canPlaceFlowers = function(flowerbed, n) {
  let location = [];

  flowerbed.forEach((f, i) => {
    if (f === 1) {
      location.push(i);
    }
  });

  if (flowerbed[0] === 0) location.unshift(-2);
  if (flowerbed[flowerbed.length - 1] === 0) location.push(flowerbed.length + 1);

  let len = location.length;

  for (let i = 0; i < len; i++) {
    let diff = location[i + 1] - location[i] - 1;

    if (diff >= 3) {
      n -= Math.floor((diff - 1) / 2);
    }

    if (n <= 0) {
      return true;
    }
  }

  return false;
};
```

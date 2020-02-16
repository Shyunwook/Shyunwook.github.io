---
title: "[Javascript] 실행 컨텍스트, 호이스팅, 클로저 2편"
subtitle: "Javascript 실행 컨텍스트, 호이스팅, 클로저"
categories:
  - Javascript
tags:
  - Javascript
classes: wide
---

## 들어가며

안녕하세요~🚀  
지난 [실행컨텍스트 1편](https://shyunwook.github.io/javascript/Javascript-%EC%8B%A4%ED%96%89%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8,%ED%98%B8%EC%9D%B4%EC%8A%A4%ED%8C%85,%ED%81%B4%EB%A1%9C%EC%A0%80/)에서 우리는 **실행컨텍스트**란 무엇인지, **ES3 기반의 실행컨텍스트**는 어떤 모습인지 살펴봤습니다.
(1편을 읽고 2편을 읽는 것을 추천드립니다!!)

오늘은 예고한대로 ES5 스펙에서 실행컨텍스트를 어떤 식으로 정의하고있는지 살펴보려고 합니다. 그리고 자바스크립트를 이해함에 있어서 빠져서는 안될 **호이스팅**과 **클로저**를 **실행컨텍스트** 관점에서 이해해 보도록 하겠습니다.

쉽지 않은 내용이지만 최대한 쉬운 척하며 써 내려가보겠습니다😳 생각의 흐름을 놓치지 않고 쭉 읽으시다보면 충분히 이해하실 수 있을겁니다!👏🏻

## 실행 컨텍스트의 구조와 생성과정(ES5)

다음은 ES5 기반의 실행 컨텍스의 구조와 생성과정을 정리한 내용입니다.

### 실행 컨텍스트의 구조

먼저 실행 컨텍스트의 구조부터 하나하나 알아보도록 하겠습니다.

```text
ExecutionContext{
  LexicalEnvironment : [Lexical Environment]
  VariableEnvironment : [Lexical Environment]
  ThisBinding: [Object]
}
```

ES5에서 실행컨텍스트는 `LexicalEnvironment`, `VariableEnvironment`, `ThisBinding` 3가지의 프로퍼티를 가집니다.

> `[]`는 해당 프로퍼티의 속성을 의미합니다. 즉 `LexicalEnvironment`와 `VariableEnvironment`는 `Lexical Environment` 속성의 값을 가지는 프로퍼티입니다.

### Lexical Environment의 구조

`Lexical Environment` 타입은 아래와 같은 형태로 구성되어 있습니다.

```text
Lexical Environment = {
  Environment Record,
  Outer Environment Reference
}
```

- Environment Record(환경 레코드) : ES3 스펙에서의 변수 객체를 떠올리시면 됩니다. 유효범위 내의 함수와 변수를 기록하는 공간입니다.
- Outer Environment Reference(외부 환경 참조) : 유효 범위가 중첩되어 있을때, 상위 렉시컬 환경을 참조합니다. ES3의 Scope Chain과 같은 역할을 하여 중첩된 유효범위 간의 탐색을 가능케 합니다.

### 환경 레코드의 종류

환경 레코드는 두 가지 타입으로 나뉘며 저장하는 값의 유형에 따라 쓰임이 달라집니다.

- Declarative Environment Record(선언적 환경 레코드)
- Object Environment Record(객체 환경 레코드)

---

**선언적 환경 레코드** : 변수, 함수, catch문의 식별자 선언과 할당문의 실행 결과가 key, value의 형태로 저장되는 공간입니다.

```javascript
   "선언적 환경 레코드" : {
     x: 'xxx',
     y: 'yyy',
     foo: function object
   }
```

**객체 환경 레코드** : 실행 문맥 외부의 객체를 **참조**하여 해당 객체 내의 프로퍼티를 읽거나 쓰는 방식으로 동작합니다.

```javascript
   "객체 환경 레코드" : {
     bindObject: Object
   }
```

> 객체 환경 레코드는 **Global Environment(전역 환경)** 와 **with** 구문을 통해 생성되는 렉시컬 환경에서 사용됩니다.

개인적으로 **객체 환경 레코드**가 어떤 역할을 하는지 이해하는게 참 어려웠습니다🤕 저와 같은 어려움을 겪으실 분들을 위해 조금 더 살펴보겠습니다.

아래 코드를 실행하면 어떤 결과가 출력될까요?

```javascript
console.log(Math.max(1, 10)); // --- 1

console.log(window.Math.max(1, 10)); // --- 2
```

두 경우 모두 10을 출력합니다😳

사실 `Math`는 window 객체에 내장된 함수로 **window.Math**와 같은 형태로 호출하는게 맞습니다. 하지만 보통 우리는 첫 번째 방식과 같이 `Math` 함수를 사용합니다.

이런 호출방식이 가능한 이유는 **전역 실행 컨텍스트**의 **전역 환경**에 window 객체가 바인딩 되어있기 떄문입니다.

```javascript

"전역 실행 컨텍스트" : {
  "LexicalEnvironment" : "전역 환경"
  "VariableEnvironment" : "전역 환경"
  "ThisBinding" : window
}

"전역 환경" : {
  ObjectEnvironmentRecord : {
    bindObject : window
  },
  OuterEnvironmentReference : null
}

```

최초 `<script>` 내부의 코드가 실행 될 때, 전역 실행 컨텍스트는 위와 같은 모습으로 생성됩니다. 위에서 처럼 `bindObject`에 `window` 객체가 바인딩 되어있기 때문에 전역 환경의 변수와 함수를 `window` 안에서 검색하여 사용할 수 있는 것입니다.

객체 환경 레코드에 window 객체가 바인딩 되기 때문에 아래 코드도 오류없이 실행 가능합니다🤪

```javascript
var a = "this is a";
console.log(window.a); // this is a
```

> "전역 공간에서 사용되는 변수의 기본적인 탐색 대상을 정해준다" 라고 이해하면 될 것 같습니다

---

### ThisBinding

this는 함수의 호출 방식에 따라 결정됩니다. 전역 실행 컨텍스트의 경우 `window`가 기본적으로 바인딩됩니다.

---

지금 까지 알아본 실행컨텍스트의 구조를 나태난 그림입니다. 요기 까지의 내용이 이해 안가신다면 잠시 쉬다가 한 번 더 위의 내용을 읽고 아래로 넘어가시는게 읽기 수월할 겁니다👍🏻
![](/assets/images/post/structure.png)

---

### 실행 컨텍스트의 생성 과정

실행 컨텍스트의 생성과정을 크게 전역 코드와 함수코드로 구분하여 알아보도록 하겠습니다.

### 전역 코드에서의 실행 컨텍스트

```javascript

"전역 실행 컨텍스트" : {
  "ThisBinding" : window,
  "LexicalEnvironment" : "전역 환경",
  "VariableEnvironment" : "전역 환경"
}

"전역 환경" : {
  ObjectEnvironmentRecord : {
    bindObject : window
  },
  OuterEnvironmentReference : null
}

```

전역 코드에서의 실행 컨텍스트 생성은 간단합니다. 자바스크립트 엔진이 `<script>`를 만나는 순간, 전역 실행 컨텍스트가 생성되고 초기화 됩니다.

### 함수 코드에서의 실행 컨텍스트

함수의 실행 컨텍스트는 함수가 실행되는 시점에 생성됩니다. 아래 코드를 살펴보겠습니다.

```javascript
var gl = "global variable!";

function foo(txt) {
  var x = "xxx";
  console.log(txt);

  function bar() {
    console.log(gl);
  }

  bar();
}

foo("foooo!");
```

`foo`함수가 실행되면 먼저 함수 실행 컨텍스트가 생성되고 `this` 바인딩이 먼저 이루어집니다. foo 함수는 일반 함수 호출방식으로 호출됐기 때문에 `thisBinding`이 `null`이고 `thiBinding`이 `null`일 경우 자동으로 전역 객체가 바인딩 됩니다.

> `use strict` 모드에서 this는 null 입니다.

```javascript
  "foo 실행 컨텍스트" : {
    ThisBinding : window,
    LexicalEnvironment : "foo 렉시컬 환경",
    VariableEnvironment : "foo 렉시컬 환경"
  }

  "foo 렉시컬 환경" : {
    "DeclarativeEnvironmentRecord" : uninitialized
    "OuterEnvironmentReference" : uninitialized
  }
```

그 후 유효 범위 내의 매개변수, 함수 선언, argument, 변수 선언의 식별자들이 순서대로 환경 레코드에 바인딩되고, **외부 환경 참조**에 상위 렉시컬 환경이 바인딩 됩니다.

```javascript
  "foo 실행 컨텍스트" : {
    ThisBinding : window,
    LexicalEnvironment : "foo 렉시컬 환경"
    VariableEnvironment : "foo 렉시컬 환경"
  }

  "foo 렉시컬 환경" : {
    "DeclarativeEnvironmentRecord" : {
      txt : "foooo!",
      bar : bar function object,
      arguments : [0 : "foooo!"],
      x : undefined
    }
    "OuterEnvironmentReference" : "전역 환경"
  }
```

> x가 할당문을 만나기 전에 `undefined`인것에 주목해주세요! 이것이 호이스팅의 실체입니다.

`bar`함수까지 실행된 후의 실행 컨텍스트 전체를 나타내보면 다음과 같습니다.

```javascript

"전역 실행 컨텍스트" : {
  "ThisBinding" : window,
  "LexicalEnvironment" : "전역 환경",
  "VariableEnvironment" : "전역 환경"
}

"전역 환경" : {
  ObjectEnvironmentRecord : {
    bindObject : window
  },
  OuterEnvironmentReference : null
}

"foo 실행 컨텍스트" : {
    ThisBinding : window,
    LexicalEnvironment : "foo 렉시컬 환경",
    VariableEnvironment : "foo 렉시컬 환경"
  }

"foo 렉시컬 환경" : {
  "DeclarativeEnvironmentRecord" : {
    txt : "foooo!",
    bar : bar function object,
    arguments : [0 : "foooo!", length: 1],
    x : "xxx"
  }
  "OuterEnvironmentReference" : "전역 환경"
}

"bar 실행 컨텍스트" : {
    ThisBinding : window,
    LexicalEnvironment : "bar 렉시컬 환경",
    VariableEnvironment : "bar 렉시컬 환경"
  }

"bar 렉시컬 환경" : {
  "DeclarativeEnvironmentRecord" : {
    arguments : null,
  }
  "OuterEnvironmentReference" : "foo 렉시컬 환경"
}
```

위 예제에서 `bar` 함수 내부에서 `gl`을 `console.log`로 출력하고 있는데, 이 경우에 탐색 순서는 아래와 같습니다.

- bar 환경 레코드 -> 외부 환경 참조를 통해 foo 렉시컬 환경으로 이동 -> foo 환경 레코드 -> 외부 환경 참조를 통해 전역 환경으로 이동 -> 전역 환경에서 `gl` 발견!

이와 같이 실행 컨텍스트 내부에 **외부 환경 참조**가 상위 렉시컬 환경을 바인딩하고 있기 때문에 하위 스코프 부터 최상위인 전역 스코프까지 식별자 탐색이 가능합니다.

### Variable Environment

Variable Environment는 기본적으로 Lexical Environment와 같은 값을 가지고 있습니다. 단 Lexical Environment는 코드가 실행됨에 따라 변할 수 있지만 Variable Environment는 변하지 않는다는 차이점이 있습니다.

이러한 Variable Environment가 사용되는 대표적인 상황은 with구문을 사용할 때 입니다.

```javascript
var foo = "origin";

with ({ foo: "new" }) {
  function bar() {
    console.log(foo);
  }
  bar();
}
```

예제 코드가 실행되면 최초 아래와 같은 전역 실행 컨텍스트가 생성됩니다.

```javascript

"전역 실행 컨텍스트" : {
  "ThisBinding" : window,
  "LexicalEnvironment" : "전역 환경",
  "VariableEnvironment" : "전역 환경"
}

"전역 환경" : {
  ObjectEnvironmentRecord : {
    bindObject : window
  },
  OuterEnvironmentReference : null
}

```

코드가 실행되다가 with 구문을 만나게 되면 새로운 렉시컬 환경이 생성되고 with 구문의 인자로 넘어온 객체가 새로운 렉시컬 환경에 바인딩 됩니다. 그리고 새롭게 생성된 렉시컬 환경이 전역 실행 컨텍스트의 **LexicalEnvironment**에 할당됩니다.

```javascript

"전역 실행 컨텍스트" : {
  "ThisBinding" : window,
  "LexicalEnvironment" : "new Environment",
  "VariableEnvironment" : "전역 환경"
}

"전역 환경" : {
  ObjectEnvironmentRecord : {
    bindObject : window
  },
  OuterEnvironmentReference : null
}

"new Environment" : {
  ObjectEnvironmentRecord : {
    bindObject : { foo : "new" }
  },
  OuterEnvironmentReference : "전역 환경"
}

```

식별자는 **LexicalEnvironment**를 통해 찾기 때문에 with 구문 이후부터는 with 구문으로 인해 확장된 유효 범위 내에서 식별자를 탐색하게 됩니다.
따라서 with 구문 이후의 아래 코드는 **new**를 출력합니다.

```javascript
console.log(foo); // new
```

이후 with 구문이 끝나게 되면 LexicalEnvironment를 원래대로 복구하기 위해 **VariableEnvironment**의 값을 LexicalEnvironment에 할당합니다.

```javascript
LexicalEnvironment = VariableEnvironment;
```

여기까지가 ES5에서 얘기하고 있는 실행 컨텍스트의 내용이었습니다. 너무 너무 어렵습니다😭 그래도 여기까지 오신 여러분은 큰 산을 넘으신 겁니다👏🏻👏🏻👏🏻 호이스팅과 클로저라는 작은 산 두 개가 더 있긴합니다만.... 조금만 더 힘내서 끝까지 읽어보자구요!

## Hoisting(호이스팅)

호이스팅은 보통, **변수 혹은 함수의 선언이 끌어올려져 유효범위의 최상단으로 이동하는 현상**으로 정의 됩니다.
아래와 같은 상황이 호이스팅의 예라고 할 수 있습니다.

```javascript
console.log(x);
foo();

var x = "this is x...!";
function foo{
  console.log('foo executed!!');
}
```

코드의 실행 순서로 따지자면 아직 선언되지 않은 `x`를 `console.log`로 출력하는데 코드는 실제 에러를 뱉지않고 `undefined`를 출력하며 정상 종료 됩니다.

이 상황을 실행컨텍스트의 관점에서 살펴보겠습니다.

```javascript

"실행 컨텍스트" : {
  "ThisBinding" : window,
  "LexicalEnvironment" : "Local 실행 컨텍스트",
  "VariableEnvironment" : "Local 실행 컨텍스트"

}

"Local 실행 컨텍스트" : {
  EnvironmentRecord : {
    foo : foo function object,
    x : undefined
  },
  OuterEnvironmentReference : "어떤 렉시컬 환경"
}

```

예제의 유효범위에서 실행컨텍스트가 생성됐을 시점에 이미 `foo` 함수와 `x`에 이미 값이 할당 된 것이 보이시나요? 이게 바로 호이스팅의 실체입니다.

실행 컨텍스트가 생성될 때,  
유효 범위 내의 **함수 식별자**의 값으로 해당 **함수 객체**가 할당되고(함수 호이스팅)  
**변수 식별자**에는 **undefined**가 할당되기(변수 호이스팅) 때문에

식별자가 선언되기 이전에 식별자에 접근해도 실행단계에서 오류가 발생하지 않는 것입니다🚀

## Closure(클로저)

MDN에서는 클로저를 아래와 같이 정의하고 있습니다.

> “A closure is the combination of a function and the lexical environment within which that function was declared.”
> 클로저는 함수와 그 함수가 선언됐을 때의 렉시컬 환경(Lexical environment)과의 조합이다.

클로저를 이해하기 위해선 함수 객체의 프로퍼티인 `[[Scope]]`에 대해 알아야 합니다. `[[Scope]]` 프로퍼티는 함수를 실행할 때, 해당 **함수가 어떤 환경에서 실행이 될 것인가**에 대한 정보를 담고 있고, 이 정보는 앞서 알아본 **Lexical Environment**입니다.

예제를 살펴 보겠습니다.

```javascript
function sum(x, y) {
  var result = x + y;

  function printResult() {
    console.log(`result : ${result}`);
  }

  return printResult;
}

var print = sum(10, 20); // --- 1

print();
```

`sum` 함수의 실행 컨텍스트는 아래와 같이 나타낼 수 있습니다.

```javascript
"sum 실행 컨텍스트" : {
  "Lexical Environment" : {
    "EnvironmentRecord" : {
      x : 10,
      y : 10,
      printResult : printResult function object,
      result : undefined
    },
    "외부 환경 참조" : window
  },
  "thisBinding" : window
}
```

`sum`의 실행 컨텍스트가 생성될 때, **함수 호이스팅**이 발생하고, 함수 식별자에 할당 할 함수 객체가 생성됩니다. 바로 이 함수 객체가 생성 될 때 `[[Scope]]` 프로퍼티가 결정되는데 그 값은 현재 생성되는 함수가 존재하는 실행 컨텍스트의 **렉시컬 환경**입니다.

```text
function object : {
  [[Scope]] : sum's 렉시컬 환경
}
```

예제 코드의 실행 중 1번 위치에서 sum함수의 실행이 종료되면 sum 함수의 실행 컨텍스트도 소멸하게 됩니다.

실행 컨텍스트가 사라지게 되면 컨텍스트 내의 렉시컬 환경도 사라져야하지만, `printResult` 함수의 `[[Scope]]` 프로퍼티에서 해당 렉시컬 환경를 참조하고 있기 때문에, 렉시컬 환경은 사라지지않고 메모리 어딘가에 **유지** 됩니다.

그리고 이 상태에서 `printResult`함수를 실행하게 되면 함수 실행 컨텍스트가 생성되고 실행 컨텍스트의 **외부 환경 참조**에 `[[Scope]]` 프로퍼티가 참조하고 있는 렉시컬 환경이 바인딩 됩니다.

```javascript
"printResult 실행 컨텍스트" : {
  "Lexical Environment" : {
      ...
    },
    "외부 환경 참조" : "sum's 렉시컬 환경"
  },
  "thisBinding" : window
}
```

결과적으로 `printResult`가 실행되면서, 환경 레코드내에서 찾을 수 없는 식별자를 외부 환경 참조에 바인딩 된 렉시컬 환경에서 탐색하는 것이 가능해집니다. 이런 함수를 **클로저**라고 합니다.

> 클로저는 반환된 내부함수가 자신이 선언됐을 때의 환경(Lexical environment)인 스코프를 기억하여 자신이 선언됐을 때의 환경(스코프) 밖에서 호출되어도 그 환경(스코프)에 접근할 수 있는 함수
> [(Closure - poiemaweb)](https://poiemaweb.com/js-closure)

## 나가며

그동안 클로저와 호이스팅을 어떠어떠한 현상이다, 이렇게 하면 쓸 수 있다 정도로 알고 넘어갔었는데, 실행 컨텍스트를 공부하며 두 가지 개념을 조금 더 깊이 이해할 수 있었습니다👍🏻

개인적으로 내용이 어렵기도 하고 , 어려운 만틈 정리가 쉽지도 않았는데 2편에 걸쳐 글을 완성하고 나니 굉장히 뿌듯하네요!!

기회가 되면 최신 스펙인 ES6에서의 실행 컨텍스트도 정리해서 올려보고 싶은 욕심이 생겼습니다🤪

여기까지 잘 읽어주신 누군가에게 박수를!!👏🏻

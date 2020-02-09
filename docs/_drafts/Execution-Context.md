Execution Context

**ES3 Ver**
실행문맥

- 변수 객체 (Variable Object)
- 스코프 체인 (Scope Chain)
- thisValue

전역 컨택스트

- 변수 객체 -> Global Object
- 스코프 체인 -> Global Object
- thisValue -> Global Object

함수 컨텍스트(foo)

- 변수 객체 -> Activation Object (foo function Object) [[scope]] -> 현재 스코프 체인
- 스코프 체인 -> 0 : foo's Activation Object, 1 : Global Object
- thisValue -> Global Object

**ES5 Ver**
실행문맥
ExecutionContext{
LexicalEnvironment : [LexicalEnvironment],
VariableEnvironment : [LexicalEnvironment],

}

LexicalEnvironment : {
EnvironmentRecord : { 실제 변수, 함수 등의 값들이 객체의 형태로 저장},
outer : Object -> reference,
this
}

LexicalEnvironment -> Lexical과 동일

EnvironmentRecord

- Declarative Environment Record -> 유효 범위 내의 식별자 바인딩 즉, 실제 변수, 함수를 기록
- Objective Environment Record -> 실제 값은 따로 저장 되어있고 그 저장되어있는 것을 내가 참조하여 사용하겠다라는 의미로 바인딩해주는 것!

호이스팅과 클로저를 설명할 수 있다. 위 실행 환경으로

---

##실행 컨텍스트
실행 컨텍스트는 간단하게 코드가 실행되기 위해 필요한 환경, 코드의 실행 환경이라고 할 수 있습니다. 자바스크립트 코드는 자바스크립트 엔진에 의해 실행되는데, 엔진이 코드를 실행하기 위해 필요한 여러 정보를 담은 것이 실행 컨텍스트 입니다.

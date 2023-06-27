# 타입스크립트 알아보기

## 아이템 1. 타입스크립트와 자바스크립트의 관계 이해하기

> 타입스크립트의 모토: 확장 가능한 자바스크립트

- 모든 자바스크립트는 타입스크립트지만, 모든 타입스크립트는 자바스크립트가 아니다.

- 타입 스크립트는 타입 구문 없이도 오류를 잡을 수 있지만, 타입 구문을 추가하면 더 많은 오류를 찾아낼 수 있다.

  - 코드의 의도가 무엇인지 타입 구문을 통해 타입 스크립트에게 알려줄 수 있다: 코드의 동작과 의도가 다른 부분을 찾을 수 있으므로 의도를 분명하게 하는 것이 좋다.

</br>

## 아이템 2. 타입스크립트 설정 이해하기

- `tsc -init`으로 tsconfig.json 설정 파일을 생성할 수 있다.

- 어디서 소스 파일을 찾고 어떤 종류의 출력을 생성할지 제어하는 내용이 대부분이지만, 언어 자체의 핵심 요소를 제어하는 설정도 존재한다.

### noImplicitAny

- no 암시적 any: 변수가 미리 정의된 타입을 가져야 하는지의 여부를 제어한다.

```ts
function add(a + b) {
  return a + b;
}
```

- 타입을 명시하지 않았다면, 타입스크립트는 각 함수와 파라미터의 타입을 `any`로 추론한다: _암시적 any_

- 만약, `noImplictAny`를 설정한다면 오류를 발생시킨다: `any` 또는 분명한 타입을 명시하면 해결할 수 있다.

- 타입스크립트는 타입 정보를 가질 때 가장 효과적이므로 `noImplicitAny`를 설정하는 것이 좋다.

- 자바스크립트 파일을 타입스크립트로 전환할 때는 설정하지 않는 것이 좋다.

### strictNullChecks

- 엄격한 null 체크: null과 undefined가 모든 타입에서 허용되는지 확인한다.

```ts
// strictNullChecks 설정시 오류 발생: number형 타입에는 null을 할당할 수 없다.
const x: number = null;

// null을 허용하고 싶다면 의도를 명시적으로 드러내야 한다.
const x: number | null = null;
```

- `undefined는 객체가 아닙니다.` 같은 런타임 오류를 방지한다.
- 해당 설정을 사용하려면 `noImplicitAny`를 먼저 설정해야 한다.

> 위 두 설정을 비롯헤 언어에 영향을 미치는 설정들을 모두 체크하고 싶다면 `strict: true`로 설정한다.

```ts
/* Type Checking */
    "strict": true /* Enable all strict type-checking options. */,
    "noImplicitAny": true,                            /* Enable error reporting for expressions and declarations with an implied `any` type.. */
    "strictNullChecks": true,                         /* When type checking, take into account `null` and `undefined`. */
    "strictFunctionTypes": true,                      /* When assigning functions, check to ensure parameters and the return values are subtype-compatible. */
    "strictBindCallApply": true,                      /* Check that the arguments for `bind`, `call`, and `apply` methods match the original function. */
    "strictPropertyInitialization": true,             /* Check for class properties that are declared but not set in the constructor. */
    "noImplicitThis": true,                           /* Enable error reporting when `this` is given the type `any`. */
    "useUnknownInCatchVariables": true,               /* Type catch clause variables as 'unknown' instead of 'any'. */
    "alwaysStrict": true,                             /* Ensure 'use strict' is always emitted. */
    "noUnusedLocals": true,                           /* Enable error reporting when a local variables aren't read. */
    "noUnusedParameters": true,                       /* Raise an error when a function parameter isn't read */
    "exactOptionalPropertyTypes": true,               /* Interpret optional property types as written, rather than adding 'undefined'. */
    "noImplicitReturns": true,                        /* Enable error reporting for codepaths that do not explicitly return in a function. */
    "noFallthroughCasesInSwitch": true,               /* Enable error reporting for fallthrough cases in switch statements. */
    "noUncheckedIndexedAccess": true,                 /* Include 'undefined' in index signature results */
    "noImplicitOverride": true,                       /* Ensure overriding members in derived classes are marked with an override modifier. */
    "noPropertyAccessFromIndexSignature": true,       /* Enforces using indexed accessors for keys declared using an indexed type */
    "allowUnusedLabels": true,                        /* Disable error reporting for unused labels. */
    "allowUnreachableCode": true,                     /* Disable error reporting for unreachable code. */
```

</br>

## 아이템 3. 코드 생성과 타입이 관계없음을 이해하기

- 타입스크립트 컴파일러는 브라우저에서의 동작을 위해 최신 자바스크립트/타입스크립트 파일을 구버전으로 transpile(translate + compile)하거나 타입 오류를 체크한다.

- 런타임과 타입은 독립적으로 실행되므로 함수를 오버로드 할 수 없다.

  - 런타임시, 타입에는 접근할 수 없지만, 값에는 접근할 수 있다.

  - 런타임 타입과 선언된 타입이 맞지 않을 수 있다.

</br>

## 아이템 4. 구조적 타이핑에 익숙해지기

- 자바스크립트는 덕 타이핑(duck typing) 기반이다.

  > 덕 타이핑
  >
  > - 객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주하는 방식.
  >
  > - '만약 어떤 새가 오리처럼 걷고, 헤엄치고, 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다.'

- 매개변수 값이 요구사항을 만족한다면 타입을 고려하지 않는 동작을 그대로 수행하면서 예상치 못한 결과가 나올 수 있다.

### _[구조적 타이핑(structural typing)](https://www.typescriptlang.org/ko/docs/handbook/type-compatibility.html)_

- 상속 관계가 명시되어 있지 않아도 타입 시스템이 객체의 프로퍼티를 체크한다: 개발자가 상속 관계를 명시해야하는 수고를 덜어준다.

  ```ts
  interface Named {
    name: string;
  }

  class Person {
    name: string;
  }

  let p: Named;

  p = new Person();
  ```

</br>

## 아이템 5. any 타입 지양하기

- 타입 안정성이 없다.

  ```ts
  let age: number;

  age = "12" as any;

  // '121'
  age += 1;
  ```

- 함수의 contract를 무시한다.

- 타입스크립트의 언어 서비스(객체 참조 자동완성 기능 및 도움말)를 적용할 수 없다.

- 버그 및 타입 설계를 감추고 타입 시스템의 신뢰도를 떨어트린다: 명확한 설계를 할 수 없다.

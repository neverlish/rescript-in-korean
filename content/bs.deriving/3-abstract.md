---
title: "[@bs.deriving abstract]"
---

이 어노테이션은 대개 권장되지 않으며 가급적 평범한 레코드를 사용하는 것이 바람직하다.

특정 시나리오에서는 유용할 수 있겠지만, 그다지 좋은 모양새는 아닐거다.

@bs.deriving(abstract)는 레코드에 붙일 수 있다. 그리고 이 어노테이션이 붙으면 더이상 레코드 타입이 아니게 된다. 그 대신

레코드 타입에 bs.deriving(abstract)를 써서 타입을 생성(creation)으로 확장한다(?)

그리고 필드와 메서드에 getter,setter를 생성한다.

대개는 뤼스크립트 레코드를 똑같은 형태의 JS 객체로 컴파일할 것이다. 그치만 deriving(abstract)가 유용한 특별한 사례가 하나 있다. 만약 옵션 필트를 가진 레코드를 JS 객체로 변환할 때, JS에 undefined가 나오지 않게 하는 상황이다. (예: {name: "Carl", age: undefined} vs. {name: "Carl"})

이 특별한 시나리오에 대해서는 [옵셔널 레이블](https://rescript-lang.org/docs/manual/latest/generate-converters-accessors#optional-labels) 섹션을 참고하시라.

### 사용 예시

```reason
[@bs.deriving abstract]
type person = {
  name: string,
  age: int,
  job: string,
};

[@bs.val] external john: person = "john";
```

노트: `person`타입은 레코드가 아니다! 이것은 레코드 문법과 타입체킹을 하므로 레코드처럼 보인다. deriving(abstract) 데코레이터는 이것을 "추상 타입"으로 바꾼다. (즉 실제 값의 모양새를 알 수 없다)

### Creation

JS쪽에 이미 있는 person 객체를 위해 바인딩을 할 필요는 없다. 그러한 person JS 객체를 뤼스크립트쪽에서 만들 수 있다. (?)

왜냐면 deriving(abstract)는 위 person 레코드를 추상 타입으로 바꾸기 때문에, 흔히 하던 대로 레코드 타입을 바로 만들 수 가 없다.  다시 말해 `{name: "Joe", age: 20, job: "teacher"}`이게 안된다.

대신 레코드 이름과 동일하게 생긴 생성 함수(creation function)를 쓸 수 있다. 이 함수는 deriving(abstract)가 암묵적으로 만들어준 것이다.

```reason
let joe = person(~name="Joe", ~age=20, ~job="teacher")
```

위 예시는 JS 런타입 오버헤드가 없음에 주목해라.

**필드 이름 바꾸기**

JS 객체를 바인딩 하다보면 ReScript에서 허용하지 않는 이름의 필드명을 사용하고 있는 경우가 있다. 예를 들면, {type: "foo"} (리스크립트의 예약어죠) 라거나 {"aria-checked": true}. 이 때는 가능한 필드명을 사용 뒤에 bs.as로 우회할 수 있다.

```reason
[@bs.deriving abstract]
type data = {
  [@bs.as "type"] type_: string,
  [@bs.as "aria-label"] ariaLabel: string,
};

let d = data(~type_="message", ~ariaLabel="hello");
```

**옵셔널 레이블**

객체를 생성할 때 일부 필드를 생략할 수도 있다.

```reason
[@bs.deriving abstract]
type person = {
  [@bs.optional] name: string,
  age: int,
  job: string,
};

let joe = person(~age=20, ~job="teacher", ());
```

옵셔널 값은 정의되지 않아서, 결과물인 JS 객체에도 나타나지 않는다. 위 예시에서 `name`이 생략된 것을 확인할 수 있다.

**Note** 옵셔널 필드를 윈한다면 bs.optional 를 붙여야 한다. 단순히 name을 option(string)으로 한다고 되는게 아니다.

Note: 생성함수가 옵셔널 필드를 포함하고 있으므로, unlabled () 를 끝에 써서 [함수 applying을 의미함을](https://rescript-lang.org/docs/manual/latest/function#optional-labeled-arguments) 알려줘야 한다.

### 접근자

이번에도 역시 deriving(abstract)가 실제 레코드의 모양을 감추기 때문에, joe.age 이런 식으로 직접 필드에 접근할 수 없다. 이에 대한 해결책으로 getter와 setter가 있다.

**읽기**

deriving(abstract) 레코드 타입의 필드마다 하나의 게터 함수가 생긴다. 위 예시에서는 nameGet, ageGet, jobGet 함수가 생긴다. `person`값을 받아서 각각 string, int, string을 리턴한다.

```reason
let twenty = ageGet(joe);
```

혹은 파이프 오퍼레이터를 써서 더 보기좋게 만들 수도 있다.

```reason
let twenty = joe->ageGet;
```

니가 만약 더 짧은 이름의 게터함수를 선호한다면 light 라고 지정할 수 있다.

```reason
[@bs.deriving {abstract: light}]
type person = {
  name: string,
  age: int,
};

let joe = person(~name="Joe", ~age=20);
let joeName = name(joe);
```

게터함수는 오프젝트 필드 이름과 동일한 이름으로 생성될 것이다.

**쓰기**

deriving(abstract) 값은 기본적으로 불변이다. 이 값을 바꾸려면 우선 레코드의 필드에 mutable로 표시를 해주어야 한다. 일반 레코드에서 mutable을 사용하는 것과 똑같다.

```reason
[@bs.deriving abstract]
type person = {
  name: string,
  mutable age: int,
  job: string,
};
```

그러면 ageSet이라는 이름의 setter가 생성될 것이다. 아래와 같이 쓸 수 있따.

```reason
let joe = person(~name="Joe", ~age=20, ~job="teacher");
ageSet(joe, 21);
```

물론 파이프 first를 쓸 수도 있다.

```reason
joe->ageSet(21);
```

### 메서드

임의의 메서드를 타입에 붙일 수 있따. (사실 deriving(abstract) 레코드 뿐만이 아니라 *아무* 타입이라도 붙일 수 있다.) [Object Method](https://rescript-lang.org/docs/manual/latest/bind-to-js-function#object-method) 페이지의 "JS 함수 바인드" 섹션을 보시라.

### Tips & Tricks

deriving(abstract)을 레버리지하면 좀 더 세밀한 접근제어가 가능하다.

**Mutablility**

구현파일(.res)에 필드를 뮤터블로 지정하더라도 인터페이스 파일에서는 그러한 mutability 속성을 숨길 수 있다.

```reason
/* test.re */
[@bs.deriving abstract]
type cord = {
  [@bs.optional] mutable x: int,
  y: int,
};
```

```reason
/* test.rei */
[@bs.deriving abstract]
type cord = {
  [@bs.optional] x: int,
  y: int,
};
```

짠! 이제 너가 작성하는 파일에서는 마음대로 뮤터블하게 사용할 수 있지만, 다른 곳에서는 그렇지 못하게 막을 수 있다!

**Hide the Creation Function**

레코드를 `private`로 만들어서 생성 함수를 막을 수 있다.

```reason
[@bs.deriving abstract]
type cord = private {
  [@bs.optional] x: int,
  y: int,
}
```

접근자는 여전히 있지만, 더 이상 이러한 데이터를 만들어낼 수 없다. JS 객체에 대한 바인딩은 필요해서 만들지만, 그런 객체가 직접 생성되는 것은 원치 않을 때 사용하면 좋다.

**Use submodules to prevent naming collisions and binding shadowing**

> 서브모듈을 써서 이름 충돌과 바인딩 쉐도잉을 막기

때로는 여러 추상타입들이 같은 이름의 필드를 가질 때가 있다. 기본적으로 리스크립트는 모든 게터/세터와 생성 함수를 같은 타입이 선언된 곳과 같은 스코프에 생성하는데, 그러다보니 쉐도잉 문제가 발생하곤 한다.

**예를 들어**

```reason
[@bs.deriving abstract]
type person = {name: string};

[@bs.deriving abstract]
type cat = {
  name: string,
  isLazy: bool,
};

let person = person(~name="Alice");

/* 에러: 값은 person 타입인데 함수는 cat 타입을 요구한다 */
person->nameGet();
```

이 문제를 피하려면 모듈을 써서 각각의 타입과 관련된 함수들을 그루핑해야 한다. 쓸 때는 지역적으로 open을 하면 된다.

```reason
module Person = {
  [@bs.deriving abstract]
  type t = {name: string}
};

module Cat = {
  [@bs.deriving abstract]
  type t = {
    name: string,
    isLazy: bool,
  };
};

let person = Person.t(~name="Alice");
let cat = Cat.t(~name="Snowball", ~isLazy=true);

/* nameGet 함수들을 구분해서 부를 수 있다. */
let shoutPersonName = {
  open Person;
  person->nameGet->Js.String.toUpperCase;
};

/* Cat의 nameGet을 부르기 위해 `open Cat`을 했다. */
let whisperCatName = {
  open Cat;
  cat->nameGet->Js.String.toLowerCase;
};
```
# 7장 코드를 작성하고 실행하기

## Item 53 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

- 시간이 흐르며 TC39는 부족했던 점들을 대부분 내장 기능으로 추가 했다. 그러나 자바스크립트에 새로 추가된 기능은 타입스크립트 초기 버전에서 독립적으로 개발했던 기능과 호환성 문제를 발생 시켰다.
- 타입스크립트 초기 버전의 형태를 유지 하기 위해 자바스크립트의 신규 기능을 그대로 채택하고 타입스크립트 초기버전과 호환성을 포기 했다.
- 이 과정에서 타입 공간과 값 공간의 경계를 혼란스럽게 만드는 기능이 있다. 여기서는 피해야 하는 기능을 몇가지 살펴 보자.

### 열거형(enum)

```tsx
enum Flavor {
  VANILLA = 0,
  CHOCOLATE = 1,
  STRAWBERRY = 2,
}
let flavor = Flavor.CHOCOLATE; // 타입이 Flavor

Flavor; // 자동완성 추천: VANILLA, CHOCOLATE, STRAWBERRY
Flavor[0]; // 값이 "VANILLA"
```

- 단순히 값을 나열하는 것보다 실수가 적고 명확하기 때문에 일반적으로 열거형을 사용한느게 좋다 그러나 타입스크립트에서는 몇가지 문제가 있다.
  - 숫자 열거형에 0, 1, 2 외의 다른 숫자가 할당되면 매우 위험하다.
  - 상수 열거형은 보통 열거형과 달리 런타임에 완전히 제거된다, 문자열 열거형과 숫자 열거형과 전혀 다른 동작이다.
  - preserveConstEnums플래그를 설정한 상태의 상수 열거형은 보통의 열거형처럼 런타임 코드에 상수 열거형 정보를 유지한다.
- 타입스크립트의 일반적인 타입들이 할당 가능성을 체크하기 위해서 구조적 타이핑을 사용하는 반면, 문자열 열거형은 명목적 타이핑을 사용한다.

```tsx
enum Flavor {
  VANILLA = "vanilla",
  CHOCOLATE = "chocolate",
  STRAWBERRY = "strawberry",
}
let flavor = Flavor.CHOCOLATE; // 타입이 Flavor
flavor = "strawberry";
// --------- '"strawberry'" 형식은 'Flavor' 형식에 할당될 수 없습니다.

function scoop(flavor: Flavor) {
  /*...*/
}
```

- Flavor는 런타임 시점에는 문자열이기 때문에, 자바스크립트에서 다음 처럼 호출할 수 있다.

```tsx
scoop('vanilla1); // 자바스크립트에서 정상
```

- 그러나 타입스크립트에서는 열거형을 임포트하고 문자열 대신 사용해야 한다.

```tsx
scoop("vanilla");
// ------------- "’vanilla"'형식은 'Flavor' 형식의 매개변수에 할당될 수 없습니다.

import { Flavor } from "ice-cream";
scoop(Flavor.VANILLA); // 정상
```

- JS와 TS 동작이 다르기 때문에 문자열 열거형은 사용하지 않는것이 좋습니다. 열거형대신 리터럴 타입의 유니온을 사용하면 된다.

### 매개변수 속성

- 일반적으로 클래스를 초기화할 떄 속성을 할당하기 위해 생성자의 매개변수를 사용한다.
- 그러나 매개변수 속성과 관련된 몇가지 문제점이 존재한다
  - 일반적으로 타입스크립트 컴파일은 타입 제거가 이루어지므로 코드가 줄어들지만, 매개변서 속성은 코드가 늘어나는 문법이다.
  - 매개변수 속성이 런타임에는 실제로 사용되지만, 타입스크립트 관점에서는 사용되지 않는 것처럼 보인다.
  - 매개변수 속성과 일반 속성을 섞어서 사용하면 클래스의 설계가 혼란스러워진다.
- 일반적으로 타입스크립트 코드에서 모든 타입 정보를 제거하면 자바스크립트가 되지만 열거형, 매개변수 속성, 트리플 슬래시 임포트, 데코레이터는 타입 정보를 제거한다고 자바스크립트가 되지 않는다.
- 타입스크립트의 역할을 명확하게 하려면, 열거형, 매개변수 속성, 트리플 슬래시 임포트, 데코레이터는 사용하지 않는게 좋다

## Item54 객체를 순회하는 노하우

```tsx
const obj = {
	one: 'uno',
	two: 1 dos',
	three: 'tres',
}；
for (const k in obj) {
	const v = obj[k];
				// ---------obj 에 인덱스 시그니처가 없기 때문에
				// 엘리먼트는 암시적으로 'any' 타입입니다.
}

// const obj: {
// one: string;
// two: string;
// three: string;
// }
for (const k in obj) { // const k: string
// ...
}
```

- k 타입은 string인 반면 obj 객체에는 ‘one’, ‘two’, ‘three’ 세 개의 키만 존재한다. k와 obj 객체의 키 타입이 서로 다르게 추론되어 오류가 발생한 것이다.
- k의 타입을 더욱 구체적으로 명시해 주면 오류는 사라진다.

```tsx
let k: keyof typeof obj; // "one" | "two" | "three" 타입
for (k in obj) {
  const v = obj[k]; // 정상
}
```

- 첫 번째 문장의 질문을 좀 더 구체적으로 바꿔 보자

```tsx
interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  for (const k in abc) {
    // const k: string
    const v = abc[k];
    // --------- 'ABC 타입에 인덱스 시그니처가 없기 때문에
    //엘리먼트는 암시적으로'any'가 됩니다.
  }
}
```

- 첫 번째 예제와 동일한 오류이다. 그러므로 같은 선언으로 오류를 제거할 수있다.
- 제대로된 오류인 이유를 예로 들어보자

```tsx
const x = {a: 'a', b: *b', c: 2, d: new DateO};
foo(x); // 정상
```

- foo 함수는 a, b, c, 속성 외에 d를 가지는 x 객체로 호출이 가능하다. foo 함수는 ABC 타입에 ‘할당 가능한’ 어떠한 값이든 매개변수로 허용하기 때문이다.

```tsx
function foo(abc: ABC) {
  let k: keyof ABC;
  for (k in abc) {
    // let k: "a" | "b" | "c"
    const v = abc[k]; // string | number 타입
  }
}
```

- k가 ‘a’ | ‘b’ | ‘c’ 타입으로 한정되어 문제가 된 것처럼, v도 string | number 타입으로 한정되어 범위가 너무 좁아 문제가 된다. d 속성은 Date 타입뿐만 아니라 어떠한 타입이든 될 수 있기 때문에 v가 string | number 타입으로 추론된 것은 잘못이며 런타임의 동작을 예상하기 어렵다
- 이런 문제를 해결하려면 Object.entries를 사용하면 된다.

```tsx
function foo(abc: ABC) {
	for (const [k, v] of Object.entries(abc)) {
		k // string 타입
		v // any 타입
	}
}

> Object.prototype.z = 3; // 제발 이렇게 하지 맙시다!
> const obj = {x: 1, y: 2};
> for (const k in obj) { console.log(k); }
x
y
z
```

## Item 55 DOM 계층 구조 이해하기

- DOM 계층은 웹브라우저에서 자바스크립트를 실행할 때 어디에서나 존재한다. 그리고 많은 부분에서 엘리먼트의 DOM과 관련된 메서드를 사용하고 엘리먼트의 속성을 사용하게 된다.
- 타입스크립트에서 DOM 엘리먼트의 계층 구조를 파악하기 용이하다. Element와 EventTarget에 달려 있는 Node의 구체적인 타입을 안다면 타입 오류를 디버깅할 수 있고, 언제 타입 단언을 사용해야 할지 알 수 있다.
- 계층 구조별로 타입을 좀 더 자세히 알아보자
  - 첫 번째, EventTarget은 DOM 타입 중 가장 추상화된 타입이다. 이벤트 리스너를 추가하거나 제거하고, 이벤트를 보내는 것밖에 할 수없다.
  - 두 번쨰, Node 타입을 알아보자.
  - 세 번째, Element와 HTMLElement를 알아보자 SVG 태그의 전체 계층 구조를 포함하면 HTML이 아닌 엘리먼트가 존재하는데, 바로 Element의 또 다른 종류인 SVGElement이다.
  - 마지막으로 HTMLxxxElement 형태의 특정 엘리먼트들은 자신만의 고유한 속성을 가지고있다.
- Node, Element, HTMLElement, EventTarget간의 차이점, 그리고 Event와 Mouse Event의 차이점을 알아야 한다.
- DOM 엘리먼트와 이벤트에는 충분히 구체적인 타입 정보를 사용하거나, 타입스크립트가 추론할 수 있도록 문백 정보를 활용하자

## Item 56 정보를 감추는 목적으로 private 사용하지 않기

- 타입스크립트에는 public, protected, private 접근 제어자를 사용해서 공개 규칙을 강제할 수 있는 것으로 오해할 수있다.

```tsx
class Diary {
  private secret = "cheated on my English test";
}
const diary = new Diary();
diary.secret;
// 'secret' 속성은 private이며
// 'Diary' 클래스 내에서만 접근할 수 있습니다.
```

- 그러나 public, protected, private 같은 접근 제어자는 타입스크립트 키워드이기 때문에 컴파일 후에는 제거된다. 이 타입스크립트 코드를 컴파일 하게 되면 다음 예제의 자바스크립 코드로 변환된다.

```tsx
class Diary {
	constructor() {
		this.secret = 'cheated on my English test1;
	}
}
const diary = new Diary();
diary.secret;
```

- 단언문을 사용하면 타입스크립트 상태에서도 private 속성에 접근할 수 있다.
- 즉, 정보를 감추기 위해 private을 사용하면 안된다.
- 확실히 데이터를 감추고 싶다면 클로저를 사용해야 한다.

## Item 57 소스맵을 사용하여 타입스크립트 디버깅하기

- 타입스크립트 코드를 실행한다는 것은, 엄밀히 말하자면 타입스크립트 컴파일러가 생성한 자바스크립트 코드를 실행한다는 것이다.
- 우리가 디버깅이 필요한 시점에 비로소 타입스크리븥가 직접 실행되는 것이 아니라는 사실을 깨닫게 될 거다. 디버거는 런터임에 동작하며, 현재 동작하는 코드가 어떤 과정을 거쳐서 만들어진 것인지 알지 못한다.
- 원본 코드가 아닌 변환된 자바스크립트 코드를 디버깅하지 말자. 소스맵을 사용해서 런타임에 타입스크립트 코드를 디버깅하자.
- 소스맵이 최종적으로 변환된 코드에 완전히 매핑되었는지 확인하자
- 소스맵에 원본 코드가 그대로 포함되도록 설정되어 있을 수도 있다. 공개되지 않도록 설정을 확인하자.

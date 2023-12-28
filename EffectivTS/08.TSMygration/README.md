# 8장 타입스크립트로 마이그레이션

<aside>
💡 큰 프로젝트를 타입스크립트로 마이그레이션하는 작업은 어렵지만, 프로젝트 품질을 크게 개선할 수 있는 가능성을 열어준다. 한꺼번에 많은 코드를 타입스크립트로 전환할 수 없기 때문에, 대규모 프로젝트를 마이그레이션할 때는 점진적으로 전환해야한다. 8장은 대부분 예제가 순수 자바스크립트여서 타입스크립트 컴파일러에서는 오류가 발생할 수 있다. 타입 체크 설정을 해제하여 타입 체크가 동작하지 않도록 하기 바란다.

</aside>

## Item 58 모던 자바스크립트로 작성하기

- 타입스크립트 컴파일러를 자바스크립트 트랜스파일러로 사용할 수 있다. 타입스크립트는 자바스크립트 상위 집합이기 때문에, 최신 버전의 자바스크립트 코드를 옛날 버전의 자바스크립트 코드로 변환할수 있다.
- 따라서 마이그레이션을 어디서부터 시작해야 할지 몰라 막막하다면 옛날 버전의 자바스크립트 코드를 최신 버전의 자바스크립트로 바꾸는 작업부터 시작해 보기 바란다.

### EcmaScript 모듈 사용하기

- ES2015부터는 임포트와 익스포트를 사용하는 ECMAScript모듈이 표준이 되었다. 만약 마이그레이션 대상인 자바스크립트 코드가 단일 파일이거나 비표준 모듈 시스템을 사용 중이라면 ES모듈로 전환하는게 좋다
- ES모듈 시스템은 타입스크립트에서 잘 동작하며, 모듈 단위로 전환할 수 있게 해주기 때문에 점진적 마이그레이션이 원활해진다.

```tsx
// CommonJS
// a.js
const b = require('•/b,);
console.log(b.name);
// b.js

const name = 'Module B';
module.exports = {name};

// ECMAScript mod나le
// a.ts
import * as b from './b';
console.log(b.name);

// b.ts
export const name = 'Module B‘;
```

### 프로토타입 대신 클래스 사용하기

- 개발자가 사용하기 애매한 프로토타입 모델보다는 견고하게 설계된 클래스 기반 모델을 더 선호했다.
- ES2015에 class 키워드를 사용하는 클래스 기반 모델이 도입되었다

```tsx
// 프로토타입
function Person(first, last) {
	this.first = first;
	this.last = last;
}

Person.prototype.getName = function() {
	return this.first + 1 ' + this.last;
}

const marie = new Person('Marie', ’Curie');
const personName = marie.getName();

// 클래스 타입
class Person {
	first: string;
	last: string;

	constructor(first: string, last: string) {
		this.first = first;
		this.last = last;
	}

	getName() {
		return this.first + ' ' + this.last;
	}
}

const marie = new Person('Marie', 'Curie');
const personName = marie.getName();
```

### var 대신 let const 사용하기

```tsx
function foo() {
  bar();
  function bar() {
    console.log("hello");
  }
}
```

- foo 함수를 호출하면 bar 함수의 정의가 호이스팅 되어 가장 먼저 수행되기 떄문에 bar 함수가 문제없이 호출되고 hello가 출력됩니다. 호이스팅은 실행 순서를 예상하기 어렵게 만들고 직관적이지 않다.
- 대신 함수 표현식 (const bar = () => { ... })을 사용하여 호이스팅 문제를 피하는 것이 좋다

### for(;;) 대신 for-of 또는 배열 메서드 사용하기

```tsx
for (const el of array) {
  // ...
}
```

- for-of 루프는 코드가 짧고 인덱스 변수를 사용하지 않기 때문에 실수를 줄일 수 있다. 인덱스 변수가 필요한 경우엔 forEach메서드를 사용하면 된다.

```tsx
array.forEach((el, i) => {
	// ...
})；
```

### 함수 표현식보다 화살표 함수 사용하기

- 화살표 함수를 사용하면 상위 스코프의 this를 유지할 수 있다.

```tsx
class Foo {
	method() {
		console.log(this);
		[1, 2].forEach(i => {
			console.log(this);
		})；
	}
}
const f = new Foo();
f .methodO;
// 항상 Foo, Foo, Foo을 출력합니다.
```

### 단축 객체 표현과 구조 분해 할당 사용하기

```tsx
const x = 1, y = 2, z = 3;
const pt = {
	x: x,
	y：
	z: z
}；

// 구조 분해 할당
const x = 1, y = 2, z = 3;
const pt = { x, y, z };
```

- 객체의 속성 중 함수를 축약해서 표현하는 방법

```tsx
const obj = {
	onClickLong: function(e) {
		// ...
	},
	onClickCompact(e) {
		// ...
	}
}；
```

- 단축 객체 표현의 반대는 객체 구조 분해입니다.

```tsx
const props = obj.props;
const a = props.a;
const b = props.b;

// 줄이기
const { props } = obj;
const { a, b } = props;

// 더 줄이기
const {
  props: { a, b },
} = obj;
```

- 참고로 a와 b는 변수로 선언되었지만 props는 변수 선언이 아니라는 것을 주의하자

### 함수 매개변수 기본값 사용하기

- 자바스크립트에서 함수의 모든 매개변수는 선택적이며, 매개변수를 지정하지 않으면 undefined로 간주된다.

```tsx
unction log2(a, b) {
	console.log(a, b);
}
log2(); // undefined undefined
```

- 옛날 자바스크립트에서는 매개변수의 기본값을 지정하고 싶을 떄, 다음 코드처럼 구현하곤 했다

```tsx
function parseNum(str, base) {
  base = base || 10;
  return parselnt(str, base);
}
```

- 모던 자바스크립트에서는 매개변수에 기본값을 직접 지정할 수 있다.

```tsx
function parseNum(str, base=10) {
	return parselntfstr, base);
}
```

- 매개변수에 기본값을 지정하면 base가 선택적 매개변수라는 것을 명확히 나타내는 효과도 줄 수 있다. 그리고 기본값을 기반으로 타입 추론이 가능하기 때문에, 타입스크립트로 마이그레이션할 떄 매개변수에 타입 구문을 쓰지 않아도 된다.

### 저수준 프로미스나 콜백 대신 async/await 사용하기

- async와 await를 사용하면 코드가 간결해져서 버그나 실수를 방지할 수 있고, 비동기 코드에 타입 정보가 전달되어 타입 추론을 가능하게 한다.

```tsx
// Promise
function getJSON(url: string) {
  return fetch(url).then((response) => response.json());
}
function getJSONCallback(url: string, cb: (result: unknown) => void) {
  // ...
}

// async/await
async function getJSON(url: string) {
  const response = await fetch(url);
  return response.json();
}
```

### 연관 배열에 객체 대신 Map과 Set 사용하기

### 타입스크립트에 use strict 넣지 않기

- 타입스크립트에서 수행되는 안전성 검사가 엄격모드보다 훨씬 더 엄격한 체크를 하기 때문에, 타입스크립트 코드에서 use strict는 무의미핟,
- 타입스크립트 코드에 ‘use strict’를 쓰지 않고, 대신 alwaysStrict 설정을 사용해야 한다.

## Item 59 타입스크립트 도입 전엔 @ts-check와 JSDoc으로 시험해 보기

- @ts-check 지사자를 사용하여 타입 체커가 파일을 분석하고, 발견된 오류를 보고하도록 지시한다. 그러나 @ts-check 지시자는 매우 느슨한 수준으로 타입 체크를 수행하는데 심지어 noImplicitAny 설정을 해제한 것보다 헐거운 체크를 수행한다는 점을 주의해야 한다.

```tsx
/ @ts-check
const person = {first: 'Grace', last: 'Hopper'};
2 * person.first
// 〜 산술 연산 오른쪽은’any', 'number', 'bigint'
// 또는 열거형 형식이어야 합니다.
```

- 전역 선언과 서드파티 라이브러리의 타입 선언을 추가하는 방법을 익힌다.
- JSDoc 주석을 잘 활용하면 자바스크립트 상태에서도 타입 단언과 타입 추론을 할 수 있습니다.
- JSDoc 주석은 중간 단계이기 떄문에 너무 공들일 필요는 없다 최종 목표는 .ts로 된 타입스크립트 코드임을 명심하자

## Item60 allowJs로 타입스크립트와 자바스크립트 같이 사용하기

# 5장 any 다루기

<aside>
💡 마이그레이션을 할 때 코드의 일부분에 타입 체크를 비활성화시켜 주는 any 타입이 중요한 역할을 한다. 또한 any를 현명하게 사용하는 방법을 익혀야만 효과적인 타입스크립트 코드를 작성할 수 있다.

</aside>

## Item 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

```tsx
function processBar(b: Bar) {
  /*...*/
}

function f() {
  const x = expressionReturningFoo();
  processBar(x);
  // ~'Foo' 형식의 인수는 'Bar' 형식의 매개변수에 할당될 수 없습니다.
}
```

- 문맥상으로 x라는 변수가 동시에 Foo 타입과 Bar 타입에 할당 가능하다면, 오류를 제거하는 방법은 두 가지이다.

```tsx
function fl() {
  const x: any = expressionReturningFoo(); // 이렇게 하지 맙시다’.
  processBar(x);
}
function f2() {
  const x = expressionReturningFoo();
  processBar(x as any); // 이게 낫습니다.
}
```

- f2가 더 나은 이유는 any 타입이 processBar 함수의 매개변수에서만 사용된 표현식이므로 다른 코드에는 영향을 미치지 않기 때문이다.
- f1에서는 함수의 마지막까지 x의 타입이 any인 반면, f2에서는 processBar 호출 이후에 x가 그대로 Foo 타입이다. 함수 바깥으로 영향을 미치지 않는다.
- 강제로 타입 오류를 제거하려면 any 대신 @ts-ignore 사용하는 것이 좋다.
- 객체와 관련된 any사용법을 살펴보자.

```tsx
const config: Config = {
	a： 1,
	b： 2,
	c: {
		key: value
		// ~’foo' 속성이 'Foo* 타입에 필요하지만’Bar' 타입에는 없습니다.
	}
}；

const config: Config = {
	a： 1,
	b: 2,
	c: {
		key: value
	}
} as any; // 이렇게 하지 맙시다!

const config: Config = {
	a: 1,
	b: 2, // 이 속성은 여전히 체크됩니다.
	c: {
		key: value as any
	}
}；
```

- 객체 전체를 any로 단언하면 다른 속성들 역시 타입 체크가 되지 않는다. 그러므로 최소한 범위에만 any를 사용하자

## Item 39 any를 구체적으로 변형해서 사용하기

- any는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입이다.
- 일반적인 상황에서는 any 보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높기 떄문에 더 구체적인 타입을 찾아 타입 안전성을 높이도록 해보자.
- any 타입의 값을 그대로 정규식이나 함수에 넣는 것은 권장되지 않는다.

```tsx
function getLengthBad(array: any) {
  // 이렇게 맙시다!
  return array.length;
}

function getLength(array: any[]) {
  return array.length;
}
```

- getLength가 더 좋은 함수인 이유는 세 가지다.
  - 함수 내의 array.length 타입이 체크된다.
  - 함수의 반환 타입이 any 대신 number로 추론된다.
  - 함수 호출될 때 매개변수가 배열인지 체크된다.
- 함수 매개변수를 구체화할 때, 배열의 배열 형태라면 any[] [] 처럼 선언하면 된다. 그리고 함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 {[key: string] : any}처럼 선언하면 된다.

```tsx
function hasTwelveLetterKey(o: { [key: string]: any }) {
  for (const key in o) {
    if (key.length === 12) {
      return true;
    }
  }
  return false;
}
```

- 함수의 매개변수가 객체지만 값을 알 수 없다면 비기본형타입을 포함하는 object 타입을 사용할 수 있다.
- object 타입은 객체의 키를 열거할 수는 있지만 속성에 접근할 수 없다는 점에서 {[key: string]: any}와 약간 다르다.

```tsx
function hasTwelveLetterKey(o: object) {
  for (const key in o) {
    if (key.length === 12) {
      console.log(key, o[key]);
      //----- ’{}' 형식에 인덱스 시그니처가 없으므로
      // 요소에 암시적으로 'any' 형식이 있습니다.
      return true;
    }
  }
  return false;
}
```

- 함수 타입에서도 단순히 any를 사용하면 안된다. 최소한으로나마 구체화할 수 있는 세 가지 방법이 있다.

```tsx
type Fn0 = () => any; // 매개변수 없이 호출 가능한 모든 함수
type Fnl = (arg: any) => any; // 매개변수 1 개
type FnN = (.. .args: any[]) => any; // 모든 개수의 매개변수
																			// "Function" 타입과 동일합니다.
```

- 마지막줄을 보면 …args의 타입을 any[]로 선언했다. any로 선언해도 동작하지만 any[]로 선언하면 배열 형태라는 것을 알 수 있어 더 구체적이다.

```tsx
const numArgsBad = (.. .args: any) => args.length; // any를 반환합니다.
const numArgsGood = (.. .args: any []) => args, length; // number를 반환합니다.
```

## Item 40 함수 안으로 타입 단언문 감추기

- 함수 내부에서는 타입단언을 사용하고 함수 외부로 드러나는 타입 정의를 정확히 명시하는 정도로 끝내는 게 낫다. 프로젝트 전반에 위험한 타입 단언문이 드러나 있는 것보다, 제대로 타입이 정의된 함수 안으로 타입 단언문을 감추는 것이 더 좋은 설계이다.

```tsx
declare function cacheLast<T extends Function>(fn: T): T;

declare f나nction shallowEqual(a: any, b: any): boolean;
function cacheLast<T extends Function>(fn: T): T {
	let LastArgs: any[]|null = null;
	let lastResult: any;
	return function(...args: any[]) {
								// ~~~~~~~~~~
								// '(...args: any[]) => any' 형식은 'T* 형식에 할당할 수 없습니다.
		if (SlastArgs || !shallowEqual(lastArgs, args)) {
			lastResult = fn(...args);
			lastArgs = args;
		}
			return lastResult;
	};
}
```

- 타입스크립트는 반환문에 있는 함수와 원본 함수 T타입이 어떤 관련이 있는지 알지 못하기 때문에 오류가 발생했다.

```tsx
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null;
  let lastResult: any;
  return function (...args: any[]) {
    if (!lastArgs || !shallowEq나al(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastRes나It;
  } as unknown as T;
}
```

- 타입 정의에는 any가 없기 때문에 cacheLast를 호출하는 쪽에는 any가 사용됐는지 알지 못한다.
- 타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현식적인 해결책이 되기도 한다. 불가피하게 사용해야 한다면, 정확한 정의를 가지는 함수 안으로 숨기도록 하자.

## Item41 any의 진화를 이해하기

- 타입스크립트에서 변수의 타입은 변수를 선언할 때 결정된다. 새로운 값이 추가되도록 확장할 수는 없다. 그러나 any 타입과 관련해서 예외인 경우가 존재한다.

```tsx
function range(start, limit) {
  const out = [];
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out;
}
```

- 이 코드를 타입스크립트로 변환하면 예상한 대로 동작한다.

```tsx
function range(start: n나mber, limit: number) {
  const out = [];
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out; // 반환 타입이 number[]로 추론됨.
}
```

- out의 타입이 처음에는 any 타입 배열인 []로 초기화되었는데, 마지막에는 number[]로 추론되고 있다.
- 코드에 out이 등장하는 세 가지 위치를 조사해 보면 이유를 알 수 있다.

```tsx
function range(start: number, limit: number) {
  const out = []; // 타입이 any[]
  for (let i = start; i < limit; i++) {
    out.push(i); // out의 타입이 any[]
  }
  return out; // 타입이 number[]
}
```

- out의 타입은 any[]로 선언되었지만 number 타입의 값을 넣는 순간부터 타입은 number[]로 진화한다.
- 타입의 진화는 타입 좁히기와 다르다 배열에 다양한 타입의 요소를 넣으면 배열의 타입이 확장되며 진화한다.

```tsx
const result = []; // 타입이 any[]
result.push("a");
result; // 타입이 string[]
result.push(1);
result; // 타입이 (string | number) []
```

- 조건문에서는 분기에 따라 타입이 변한다.

```tsx
let val; //타입이 any
if (Math.random() < 0.5) {
  val = /hello/;
  val; // 타입이 RegExp
} else {
  val = 12;
  val; // 타입이| number
}
val; // 타입이 number | RegExp
```

- 일반적인 타입은 정제 되지만 암시적 any와 any[] 타입은 진화할 수 있다. 이러한 동작이 발생하는 코드를 인지하고 이해할 수 있어야 한다.
- any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법이다.

## Item 42 모르는 타입의 값에는 any 대신 unknown을 사용하기

- unknown 은 any 대신 사용할 수 있는 안전한 타입이다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우라면 unknown을 사용하면 된다.

```tsx
interface Book {
	name: string;
	author: string;
}

const book: Book = parseYAML('
	name: Wuthering Heights
	author: Emily Bronte
')；

function parseYAML(yaml: string): any {
	const book = parseYAMLC
		name: Jane Eyre
		author: Charlotte Bronte
	')；
	alert (book, title); // 오류 없음,런타임에 "undefined" 경고
	book (‘read1); // 오류 없음,런타임에 "TypeError: book 은 함수가 아닙니다" 예외 발생
}

function safeParseYAMUyaml: string): unknown {
	return parseYAML(yaml);
}
	const book = safeParseYAML(`
		name: The Tenant of Wildfell Hall
		author: Anne Bronte
	`)；
	alert(book.title);// 개체가’unknown' 형식입니다.
	book("read");// 개체가 'unknown' 형식입니다.
```

- any가 위함한 이유는 두 가지 특징으로부터 비롯된다.
  - 어떠한 타입이든 any타입에 할당 가능하다.
  - any타입은 어떠한 타입으로도 할당 가능하다.
- 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 unkown을 사용하면 된다.
- {}, object, unknown의 차이점을 이해해야 한다.

## Item 43 몽키 패치보다는 안전한 타입을 사용하기

- 자바스크립트의 가장 유명한 특징 중 하나는, 객체와 클래스에 임의의 속성을 추가할 수있을 만큼 유연하다는 것이다. 객체에 속성을 추가할 수 있는 기능은 종종 웹 페이지에서 window나 document 에 값을 할당하여 전역 변수를 만드는 데 사용된다.

```tsx
window.monkey = "Tamarin";
document.monkey = "Howler";
```

- 또는 DOM 엘리먼트에 데이터를 추가하기 위해서도 사용된다.

```tsx
const el = document.getElementByld("colobus");
el.home = "tree";
```

- 전역변수를 사용하면 은연중에 프로그램 내에서 서로 멀리 떨어진 부분들 간에 의존성을 만들게 된다. 그러면 함수를 호출할 때마다 부작용을 고려해야한다.
- 타입스크립트까지 더하면 또 다른 문제가 발생한다. 타입 체커는 Document와 HTMLElement의 내장 속성에 대해서는 알고 있지만, 임의로 추가한 속성에 대해서는 알지 못한다.

```tsx
document.monkey = "Tamarin";
// ~~~~'Document' 유형에 'monkey' 속성이 없습니다.
```

- 이 오류를 해결하는 가장 간단한 방법은 any 단언문을 사용하는 것이다.

```tsx
(document as any).monkey = "Tamarin"; // 정상
```

- any를 사용함으로써 타입 안정성을 상실하고, 언어 서비스를 사용할 수 없게 된다.

```tsx
(document as any).monky = "Tamarin"; // 정상,오타
(document as any).monkey = /Tamarin/; // 정상, 잘못된 타입
```

- 최선의 해결책은 document또는 DOM으로부터 데이터를 분리하는 것이다.
- 분리할 수 없는 경우 두가지 차선책이 존재한다.
  - interface의 특수 기능 중 하나인 보강을 사용하는것,
  ```tsx
  interface Document {
    /** 몽키 패치의 속(genus) 또는 종(species) */
    monkey: string;
  }
  docume;
  nt.monkey = "Tamarin"; // 정상
  ```
  - 보강을 사용하는 방법이 any보다 나은점은 다음과 같다.
    - 타입이 더 안전하다. 타입 체커는 오타나 잘못된 타입의 할당을 오류로 표시한다.
    - 속석에 주석을 붙일 수 있다.
    - 속성에 자동완성을 사용할 수 있다.
    - 몽키 패치가 어떤 부분에 적용되었는지 정확한 기록이 남는다.
  - 그리고 모듈 관점에서(타입스크립트 파일이 import / export를 사용하는 경우), 제대로 동작하게 하려면 global 선언을 추가해야 한다.
  ```tsx
  export {};
  declare global {
    interface Document {
      /** 몽키 패치의 속(genus) 또는 종(species) */
      monkey: string;
    }
  }

  document.monkey = "Tamarin"; // 정상
  ```
  - 두 번째, 더 구체적인 타입 단언문을 사용하는 것이다.
  ```tsx
  interface MonkeyDocument extends Document {
    /** 몽키 패치의 속(genus) 또는 종(species) */
    monkey: string;
  }
  (document as MonkeyDocument).monkey = "Macaque";
  ```

## Item 44 타입 커버리지를 추적하여 타입 안전성 유지하기

- noImplicitAny를 설정하고 모든 암시적 any 대신 명시적 타입 구문을 추가해도 any 타입과 관련된 문제들로 부터 안전하다고 할 수 없다. 그 이유는 두가지 경우가 있다.
  - 명시적 any 타입
    - 아이템 38과 아이템 39의 내용에 따라 any 타입의 범위를 좁히고 구체적으로 만들어도 여전히 any 타입이다.
  - 서드파티 타입 선언
    - 이 경우는 @types 선언 파일로부터 any 타입이 전파되기 때문에 특별히 조심해야 한다.
- any 타입은 타입 안전성과 생산성에 부정적 영향을 미칠 수 있으므로, 프로젝트에서 any의 개수를 추적하는 것이 좋다.
- npm의 type-cover-age 패키지를 활용하여 any를 추적할 수 있는 몇가지 방법이 있다.

```tsx
$ npx type-coverage
9985 / 10117 98.69%
```

- 이 프로젝트의 10,117개 심벌 중 9,985(98.69%)가 any가 아니거나 any의 별칭이 아닌 타입을 가지고 있음을 알 수 있다. 실수로 any타입이 추가된다면 백분율이 감소하게 된다.
- 타입 커버리지를 사용할 때 —detail플래그를 붙이면, any타입이 있는 곳을 모두 출력해 준다.

```tsx
$ npx type-cove rage —detail
path/to/code.ts:1:10 getColumnlnfo
path/to/module.ts:7:1 pt2
...
```

- 작성한 프로그램의 타입이 얼마나 잘 선언되었는지 추적해야 한다. 추적함으로써 any의 사용을 줄여 나갈 수 있고 타입 안전성을 꾸준히 높일 수 있다.

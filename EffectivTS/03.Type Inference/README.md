# 3장 타입 추론

## Item 19 추론 가능한 타입을 사용해 장황한 코드 방지하기

- 타입스크립트를 처음 접한 개발자는 변수를 선언할 때마다 타입을 명시해야 한다고 생각한다. 모든 변수에 타입을 선언하는 것은 비생산적이며 형편없는 스타일로 여겨진다.
- 타입 추론이 된다면 명시적 타입 구문은 필요하지 않는다. 오히려 방해가 될뿐이다.
- 타입스크립트는 더 복잡한 객체도 추론할 수 있다.

```tsx
const person: {
	name: string;
	born: {
		where: string;
		when: string;
		}；
	died: {
		where: string;
		when: string;
	}
} = {
	name: 'Sojourner Truth',
	born: {
		where: 'Swartekill, NY',
		when: 'c.1797',
	},
	died: {
		where: 'Battle Creek, MI',
		when: 'Nov. 26, 1883’
	}
}；
```

- 타입을 생략하고 작성해도 충분하다.

```tsx
const person = {
	name: 'Sojourner Truth',
	born: {
		where: ’Swartekill, NY',
		when: 'c.1797',
	},
	died: {
		where: 'Battle Creek, MI',
		when: 'Nov. 26, 1883'
	}
}；
```

- 타입스크립트는 예상한 것보다 더 정확하게 추론하기도 한다.

```tsx
const axisl: string = 'x'; // 타입은 string
const axis2 = ’y'; // 타입은 "y“
```

- axis2 변수를 string으로 예상하기 쉽지만 타입스크립트가 추론한 ‘y’가 더 정확한 타입이다.
- 타입이 추론되면 리팩터링이 용이해진다.

```tsx
interface Product {
  id: number;
  name: string;
  price: number;
}
function logProduct(prod나ct: Product) {
  const id: number = product.id;
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}
```

## Item 20 다른 타입에는 다른 변수 사용하기

- 자바스크립트에서는 한 변수를 다른 목적을 가지는 다른 타입으로 재사용해도 된다. 반면 타입스크립트에서는 두 가지 오류가 발생한다.

```tsx
let id = "12-34-56";
fetchProduct(id);
id = 123456;
// ~~~'123456' 형식은 'string' 형식에 할당할 수 없습니다.
fetchProductBySerialNumber(id);
// ~~ 'string' 형식의 인수는
// 'number* 형식의 매개변수에 할당될 수 없습니다
```

- 타입스크립트는 “12-34-56-”이라는 값을 보고 id의 타입을 string으로 추론했다. string타입에는 number 타입을 할당할 수 없기 때문에 오류가 발생한다.
- “변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다”는 중요한 관점을 알 수 있다. 타입을 바꿀 수 있는 한 가지 방법은 범위를 좁히는 것인데, 새로운 변수값을 포함하도록 확장하는 것이 아니라 타입을 더 작게 제한하는 것이다.
- id의 타입을 바꾸지 않으려면 모두 포함하는 유니온타입으로 확장하면 된다.

```tsx
let id: string | number = "12-34-56";
fetchProduct(id);
id = 123456; // 정상
fetchProductBySerialNumber(id); // 정상

let id: string | number = "12-34-56";
fetchProduct(id);
id = 123456; // 정상
fetchProductBySerialNumber(id); // 정상
```

- 앞의 예제에서 첫 번째와 id와 재사용한 두 번째 id는 서로 관련이없다.
- 변수를 무분별하라게 재사용하면 타입 체커와 사람 모두에게 혼란을 줄 뿐이다.
- 다른 타입에는 별도의 변수를 사용하는 게 바람직 한 이유
  - 서로 관련이 없는 두 개의 값을 분리한다.(id와 serial)
  - 변수명을 더 구체적으로 지을수 있다.
  - 타입 추론을 향상시키며, 타입 구문이 불필요해진다.
  - 타입이 좀 더 간결해 진다(string|number 대신 string과 number를 사용)
  - let 대신 const로 변수를 선언하게 된다. const로 변수를 선언하면 코드가 간결해지고, 타입 체커가 타입을 추론하기도 좋다.

## Item 21 타입 넓히기

- 아이템 7에서 설명한 것처럼 런타임에 모든 변수는 유일한 값을 가진다. 그러나 타입스크립트가 작성된 코드를 체크하는 정적 분석 시점에 변수는 ‘가능한’ 값들의 집합인 타입을 가진다.
- 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추해야 한다는 뜻이다. 타입스크립트에서 이러한 과정을 ‘넓히기’라고 부른다. 이 과정을 이해하면 오류 원인을 파악하고 타입 구문을 더 효과적으로 사용할 수 있다.
- 타입 추론의 강도를 직접 제어하려면 타입스크립트의 기본 동작을 재정의해야한다.
- 첫 번째, 명시적 타입 구문을 제공하는것이다.

```tsx
const v: { x: 1|3|5 } = {
	X： 1,
}; // 타입이 { x: 1|3|5; }
```

- 두 번째, 타입 체커에 추가적인 문맥을 제공하는 것이다.
- 세 번째, const 단언문을 사용하는 것이다. const 단언문과 변수 선언에 쓰이는 let이나 const와 혼동해서는 안된다. const 단언문은 온전히 타입 공간의 기법이다.

## Item22 타입 좁히기

- 타입 넓히기의 반대는 타입 좁히기이다. 타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말한다.

```tsx
const el = document.getElementByld(" foo"); // 타입이 HTMLElement | null
if (el) {
  el; // 타입이 HTMLElement
  el.innerHIML = "Party Time".blinkO;
} else {
  el; // 타입이 null
  alert("No element #foo");
}
```

- el이 null이라면, 분기문의 첫 번째 블록이 실행되지 않는다. 더 좁은 타입이 되어 작업이 쉬워진다. 타입 체커는 일반적으로 이러한 조건문에서 타입 좁히기를 잘해내지만 타입 별칭이 존재한다면 그렇지 못할 수도 있다.
- 분기문에서 예외를 던져서 변수 타입을 좁힐 수 있고. instanceof를 사용해 타입을 좁힐 수도 있다.

```tsx
const el = document.getElementById('foo'); // 타입이 HTMLElement | null
if (!el) throw new Error('Unable to find #foo');
el; // 이제 타입은 HIMLElement
el.innerHTML = 'Party Time’.blink();

function contains(text: string, search: string|RegExp) {
	if (search instanceof RegExp) {
		search // 타입이 RegExp
		return !!search.exec(text);
}
	search // 타입이 string
	return text.includes(search);
}
```

- 일반 적으로 명시적 ‘태그’를 붙이는 방법도 있다.

```tsx
interface UploadEvent { type: 'upload'; filename: string; contents: string }
interface DownloadEvent { type: 'download'; filename: string; }
type AppEvent = UploadEvent | DownloadEvent;
function handleEvent(e: AppEvent) {
	switch (e.type) {
		case 'download':
			e // 타입이 DownloadEvent
			break;
		case 'upload’:
			e; // 타입이 UploadEvent
			break;
	}
}
```

- 이 패턴은 ‘태그된 유니온’ 또는 ‘구별된 유니온’이라고 불리며 타입스크립트 어디에서나 찾아볼 수 있다.
- 만약 타입스크립트가 타입을 식별하지 못하면 식별을 돕기 위해 커스텀 함수를 도입할 수 있다.

```tsx
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return "value" in el;
}
function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    el; // 타입이 HTMLInputElement
    return el.value;
  }
  el; // 타입이 HTMLElement
  return el.textContent;
}
```

- 이러한 기법을 ‘사용자 정의 타입 가드’라고 한다. 반환 타입의 el is HTML InputElement는 함수의 반환이 true인 경우, 타입 체커에게 매개변수의 타입을 좁힐 수 있다고 알려준다.

## Item 23 한꺼번에 객체 생성하기

- 일반 적으로 변수의 타입은 변경되지 않는다. 즉, 객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입 추론에 유리하다.

```tsx
const pt = {};
pt.x = 3;
// ~ '{}' 형식에 'x' 속성이 없습니다.
pt.y = 4;
// ~ '{}' 형식에 'y' 속성이 없습니다
```

- 첫 번째 줄의 pt 타입은 {} 값을 기준으로 추론되기 때문에 존재하지 않는 속성을 추가할 수 는 없다.

```tsx
interface Point {
  x: number;
  y: number;
}
const pt: Point = {};
// —~~'{}' 형식에 'Point1 형식의 x, y 속성이 없습니다.
pt.x = 3;
pt.y = 4;
```

- 이 문제들은 객체를 한번에 정의하면 해결할 수 있다.

```tsx
const pt = {
	x: 3,
	y： 4,
}； // 정상
```

- 객체들을 조합해서 큰 객체를 만들어야 한느 경우에도 여러 단계를 거치는 것은 좋지않다.
- 객체 전개 연산자 …를 사용하면 큰 객체를 한꺼번에 만들어 낼 수 있다.

```tsx
const pt = { x: 3, y: 4 };
const id = { name: "Pythagoras" };
const namedPoint = {};
Object.assign(namedPoint, pt, id);
namedPoint.name;
// '{}' 형식에' name'속성이 없습니다.

const namedPoint = { ...pt, ...id };
namedPoint.name; // 정상,타입이 string
```

- 타입에 안전한 방식으로 조건부 속성을 추가하려면, 속성을 추가하지 않는 null 또는 {}으로 객체 전개를 사용하면 된다.

```tsx
declare let hasMiddle: boolean;
const firstLast = {first: ’Harry', last: 'Truman'};
const president = {...firstLast, ...(hasMiddle ? {middle: 'S' } : {})}；
```

## Item 24 일관성 있는 별칭 사용하기

```tsx
const borough = { name: "Brooklyn", location: [40.688, -73.979] };
const loc = borough.location;
```

- borough.location 배열에 loc라는 별칭을 만들었다. 별칭의 값을 변경하면 원래 속성값에서도 변경된다.
- 그런데 별칭을 남발해서 사용하면 제어 흐름을 분석하기 어렵다. 타입스크립트에서도 마찬가지로 별칭을 신중하게 사용해야한다. 그래야 코드를 잘 이해할 수 있고, 오류도 쉽게 찾을 수 있다.
- 별칭은 타입스크립트가 타입을 좁히는 것을 방해한다. 따라서 변수에 별칭을 사용할 떄는 일관되게 사용해야한다.
- 비구조화 문법을 사용해서 일관딘 이름을 사용하는게 좋다.
- 함수 호출이 객체 속성의 타입 정제를 무효화할 수 있다는 점을 주의해야 한다. 속성보다 지역변수를 사용하면 타입 정제를 믿을 수 있다.

## Item 25 비동기 코드에는 콜백 대신 async 함수 사용하기

- ES5 또는 더 이전 버전을 대상으로 할 때, 타입스크립트 컴파일러는 async와 await가 동작하도록 정교한 변환을 수행한다. 다시 말해, 타입스크립트는 런타임에 관계 없이 async/await를 사용할 수 있다.
- 콜백 보다는 프로미스나 async/await를 사용해야 코드 작성, 타입 추론하기 쉽다.
- 병렬 페이지를 로드 하고싶다면 프로미스를 조합하면 된다.

```tsx
async function fetchPages() {
	const [responsel, response?, responses] = await Promise.all([
		fetch(urll), fetch(url2), fetch(url3)
	])；
	// ...
}
```

- 이런 경우는 await와 구조 분해 할당이 찰떡궁합이다. 그러나 콜백 스타일로 동일한 코드를 작성하려면 더 많은 코드와 타입 구문이 필요하다

```tsx
function fetchPagesCB() {
	let numDone = 0;
	const responses: string[] = [];
	const done = () => {
		const [responsel, response》, responses] = responses;
		// ...
}；
	const urls = [urll, url2, url3];
	urls.forEach((url, i) => {
		fetchURL(urlf r => {
			responses[i] = url;
			numDone++;
			if (numDone === urls.length) done();
		})；
	})；
}
```

- 가끔 프로미스를 직접 생성해야 할 때, 특히 setTimeout 같은 콜백 API를 래핑할 경우가 있다. 그러나 선택의 여지가 있다면 일반적으로는 프로미스를 생성하기보다는 async/await를 사용해야한다. 그 이유는 다음과 같다
  - 일반적으로 더 간결하고 직관적인 코드가 된다.
  - async 함수는 항상 프로미스를 반환하도록 강제된다.

## Item 26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

- 타입스크립트는 타입을 추론할 때 단순히 값만 고려하지 않는다. 값이 존재하는 곳의 문맥까지도 살핀다. 타입 추론에 문맥이 어떻게 사용되는지 이해하고 있다면 제대로 대처할 수 있다.

```tsx
// 인라인 형태
setLanguage("JavaScript");
// 참조 형태
let language = "JavaScript";
setLanguage(language);
```

- 타입스크립트에서는 다음 리팩터링이 여전히 동작한다

```tsx
function setLanguage(language: string) {
  /*...*/
}
setLanguage(" JavaScript"); // 정상
let language = "JavaScript";
setLanguage(language); // 정상
```

- 문자열 타입을 더 특정해서 문자열 리터럴 타입의 유니온으로 바꾼다고 가정해보자

```tsx
type Language = 'JavaScript' | 'TypeScript' | 1 Python1;
function setLang나age(language: Language) {/*...*/}
set Language (' JavaSc ript ’); // 정상
let language = 'JavaScript';
setLanguage(language);
// 'string' 형식의 인수는
// 'Language' 형식의 매개변수에 할당될 수 없습니다.
```

- 인라인 형태에서 타입스크립트는 함수 선언을 통해 매개변수가 Language 타입이어야 한다는 것을 알고 있다.
- 이 값을 변수로 분리해내면, 타입스크립트는 할당 시점에 타입을 추론한다. 이번 경우는 string으로 추론했고, Language타입으로 할당이 불가능하므로 오류가 발생했다.
- 이 문제를 해결하기 위해서는 타입 선언에서 language의 가능한 값을 제한하는 것이다.

```tsx
let language: Language = "JavaScript";
setLanguage(language); // 정상
```

- 오타를 표히해 주는 장점도 있다.
- 두 번쨰 해법은 language를 상수로 만드는 것이다.

```tsx
const language = "JavaScript";
setLanguage(language); // 정상
```

- const를 사용하여 타입 체커에게 language는 변경할 수 없다고 알려준다.
- 따라서 타입스크립트는 language에 대해서 더 정확한 타입인 문자열 리터럴 ‘javascript’로 추론할 수 있다.

## Item 27 함수형 기법과 라이브러리로 타입 흐름 유지하기

- 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기보다는 내장된 함수형 기법과 로대시 같은 유틸리티 라이브러리를 사용하는것이 좋다.

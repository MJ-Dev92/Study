# 2장 타입스크립트의 타입 시스템

<aside>
💡 2장에서는 타입 시스템의 기초부터 살펴보자. 타입 시스템이란 무엇인지, 어떻게 사용해야 하는지, 무엇을 결정해야 하는지, 가급적 사용하지 말아야 할 기능은 무엇인지 알아보자

</aside>

## Item6 편집기를 사용하여 타입 시스템 탐색하기

- 타입 스크립트를 설치하면, 다음 두 가지를 실행할 수 있다.
  - 타입스크립트 컴파일러(tsc)
  - 단독으로 실행할 수 있는 타입스크립트 서버(tsserver)
- 타입스크립트 서버는 ‘언어 서비스’를 제공한다. 코드 자동 완성, 명세 검사, 검색, 리팩터링이 포함된다. 보통은 편집기를 통해서 언어 서비스를 사용하는데, 타입스크립트 서버에서 언어 서비스를 제공하도록 설정하는게 좋다
- 편집기는 타입스크립트가 언제 타입 추론을 수행할 수 있는지 대한 개념을 잡게 해준다.
- 보통의 경우 심벌 위에 마우스 커서를 대면 타입스크립트가 그 타입을 어떻게 판단하고 있는지 확인할 수 있다.
- 그림 2-2를 보면 함수의 반환 타입이 number라는 것이다. 이 타입이 기대한 것과 다르다면 타입 선언을 직접 명시하고, 실제 문제가 발생하는 부분을 찾아 봐야한다.
- 편집기상의 타입 오류를 살펴보는 것도 타입 시스템의 성향을 파악하는 데 좋은 방법입니다. 예를 들어, 다음은 id에 해당하거나 기본값인 HTMLELement를 반환하는 함수이다.

```jsx
function getElement(elOrId: string | HTMLElement | null): HTMLElement {
  if (typeof elOrld === "object") {
    return elOrld;
    // 'HTMLElement | null' 형식은 'HTMLElement1 형식에 할당할 수
    // 없습니다.
  } else if (elOrld === null) {
    return document.body;
  } else {
    const el = document.getElementByld(elOrid);
    return el;
    // --------------- 'HTMLElement | null' 형식은'HTMLElement'형식에 할당할 수 없습니다
  }
}
```

- 첫 번째 if 분기문의 의도는 단지 HTMLELement라는 객체를 골라내는 것인데 자바스크립트에서 typeof null은 ‘object’이므로, elOrId는 여전히 분기문 내에서 null일 가능성이 있다.
- 그러므로 처음에 null 체크를 추가해서 바로잡는다. 두 번째 오류는 document.getElementById가 null을 반환할 가능성이 있어서 발생했고, 첫 번째 오류와 동일하게 null 체크를 추가하고 예외를 던져야 한다.

## Item7 타입이 값들의 집합이라고 생각하기

- 런타임에 모든 변수는 자바스크립트 세상의 값으로부터 정해지는 각자의 고유한 값을 가진다.
- 그러나 코드가 실해되기 전, 즉 타입스크립트가 오류를 체크하는 순간에는 ‘타입’을 가지고 있다. ‘할당 가능한 값들의 집합’이 타입이라고 생각하면 된다. 이 집합은 타입의 ‘범위’ 라고 부르기도 한다.
- 가장 작은 집합은 아무 값도 포함하지 않는 공집합이며, 타입스크립트에서는 never 타입이다. never 타입으로 선언된 변수의 범위는 공집합이기 떄문에 아무런 값도 할당할 수 없다.

```jsx
const x: naver = 12;
// ~ '12' 형식은 'never' 형식에 할당할 수 없다
```

- 그 다음으로 작은 집합은 한 가지 값만 포함하는 타입이다. 이들은 타입스클비트에서 유닛타입이라고도 불리틑 리터럴 타입이다.

```jsx
type A = "A";
type B = "B";
type Twelve = 12;
```

- 두 개 혹은 세 개로 묶으려면 유니온타입을 사용한다.

```jsx
type AB = "A" | "B";
type AB12 = "A" | "B" | 12;
```

- 유니온 타입은 값 집합들의 합집합을 일컫습니다.
- 집합의 관점에서, 타입 체커의 주요 역할은 하나의 집합이 다른 집합의 부분 집합인지 검사하는 것이라고 볼 수 있다.

```tsx
//정상, {"A", "B"}는 {"A", "B"}의 부분 집합입니다.
const ab: AB = Math.random() < 0.5 ? "A" : "B";
const abl2: AB12 = ab; // 정상, {"A", "B"}는 {"A", "B", 12}의 부분 집합입니다.

declare let twelve: AB12;
const back: AB = twelve;
// ------ 'AB12' 형식은 'AB' 형식에 할당할 수 없습니다.
// ‘12’ 형식은 'AB' 형식에 할당할 수 없습니다.
```

- 특정 상황에서만 추가 속성을 허용하지 않는 잉여 속성 체크만 생각하다 보면 간과하기 쉽다.
- 연산과 관련된 이해를 돕기 위해 값의 집합을 타입이라고 생각해 보자

```tsx
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & Lifespan;
```

- & 연산자는 두 타입의 인터섹션을 계산한다. 언뜻 보기에 PersonSpan 타입을 공집합으로 예상하기 쉽다. 그러나 타입 연산자는 인터페이스의 속성이 아닌, 값의 집합에 적용된다. 그리고 추가적인 속성을 가지는 값도 여전히 그 타입에 속한다. 그래서 Person과 Lifespan을 둘 다 가지는 값은 인터섹션 타입에 속하게 된다.

```tsx
const ps: PersonSpan = {
name: 'Alan Turing',
birth: new Date('1912/06/23'),
death: new Date(11954/06/07'),
}; // 정상
```

- 당연히 앞의 세 가지보다 더 많은 속성을 가지는 값도 PersonSpan 타입에 속한다. 인터섹션 타입의 값은 각 타입 내의 속성을 모두 포함하는 것이 일반적
- 두 인터페이스의 유니온에서는 그렇지 않다.
- 유니온 타입에 속하는 값은 어떠한 키도 없기 때문에, 유니온에 대한 keyof는 공집합이어야만 한다.

```tsx
keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

- ‘서브타입’ 어떤 집합이 다른 집합의 부분집합이라는 의미

```tsx
interface VectorlD {
  x: number;
}
interface Vector2D extends VectorlD {
  y: number;
}
interface Vector3D extends Vector2D {
  z: rnjmber;
}
```

- Vector3D는 Vector2D의 서브타입, Vector2D는 Vector1D의 서브타입

```tsx
interface VectorlD {
  x: number;
}
interface Vector2D {
  x: number;
  y: number;
}
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
```

## Item8 타입 공간과 값 공간의 심벌 구분하기

- 타입스크립트의 심벌은 타입 공간이나 값 공간 중의 한 곳에 존재한다 심ㅓㄹ은 이름이 같더라도 속하는 공간에 따라 다른 것을 나타낼 수 있기 때문에 혼란스러울 수 있다.

```tsx
interface Cylinder {
  radius: number;
  height: number;
}
const Cylinder = (radius: number, height: number) => ({ radius, height });
```

- interface에서 Cylinder은 타입으로 쓰이고, const에서는 이름은 같지만 값으로 쓰인다 서로 아무런 관련이 없으며 타입으로 쓰일수도 값으로 쓰일 수도 있다. 이런 점이 가끔 오류를 야기한다.

```tsx
type Tl = 'string literal*;
type T2 = 123;
const vl = 'string literal';
const v2 = 123;
```

- 두 공간에 대한 개념을 잡으려면 타입스크립트 플레이그라운드를 활용하면 된다. 타입스크립트 플레이그라운드는 타입스크립트 소스로부터 변환된 자바스크립트 결과물을 보여준다.
- class와 enum은 상황에 따라 타입과 값 두 가지 모두 가능한 예약어이다. 다음 예제는 Cylinder 클래스는 타입으로 쓰였다.

```tsx
class Cylinder {
  radius = l;
  height = l;
}
function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape; // 정상,타입은 Cylinder shape, radius // 정상,타입은 number
  }
}
```

- 클래스가 타입으로 쓰일 떄는 형태(속성과 메서드)가 사용되는 반면, 값으로 쓰일 때는 생성자가 사용된다.
- 연산자 중에서도 타입에서 쓰일 떄와 값에서 쓰일 떄 다른 기능을 하는 것들이 있다. 그 예 중 하나로 typeof를 들 수 있다

```tsx
type Tl = typeof p; //타입은 Person
type T2 = typeof email;
// 타입은(P： Person, subject: string, body: string) => Response
const vl = typeof p; // 값은 "object"
const v2 = typeof email; // 값은 "function"
```

- 타입의 관점에서 typeof는 값을 읽어서 타입스크립트 타입을 반환한다. 타입 공관의 typeof는 보다 큰 타입의 일부분으로 사용할 수 있고, type 구문으로 이름을 붙이는 용도로 사용할 수 있다.
- 값의 관점에서 typeof는 자바스크립트 런타입 typeof 연산자가 된다. 값 공간의 typeof는 심벌의 런타임 타입을 가리키는 문자열을 반환한하며, 타입스크립트 타입과는 다르다. 타입스크립트 타입은 종류가 무수히 많은 반면, 자바스크립트에는 과거부터 지금까지 단 6개의 런타임 타입만이 존재한다.
- 속성 접근자인 []는 타입으로 쓰일 때에도 동일하게 동작한다. 그러나 obj[’field’]와 obj.field는 값이 동일하더라도 타입은 다를 수 있다. 따라서 속성을 얻을 때에는 반드시 첫 번째 방법을 사용해야 한다.
- 타입 스크립트 코드가 잘 동작하지 않는다면 타입 공간과 값 공간을 혼동해서 잘못 작성했을 가능성이 크다. 자바스크립트에서는 객체 내의 각 속성을 로컬 변수로 만들어 주는 구조분해 할당을 사용할 수있지만 타입스크립트에서 구조분해 할당을 하면, 이상한 오류가 발생한다.

```tsx
function email({
  person: Person,
  // ----바인딩 요소’Person’에 암시적으로 'any' 형식이 있습니다
  subject: string,
  // --------- 'string1 식별자가 중복되었습니다.
  //바인딩 요소’string’에 암시적으로 'any' 형식이 있습니다.
  body: string,
}) // --------- ’string' 식별자가 중복되었습니다.
//바인딩 요소‘string1 에 암시적으로 'any' 형식이 있습니다.
{
  /*...*/
}
```

- 값의 관점에서 Person과 string이 해석되었기 떄문에 오류가 발생한다. Person이라는 변수명과 string이라는 이름을 가지는 두 개의 변수를 생성하려 한 것이다. 문제를 해결하려면 타입과 값을 구분해야 한다.

```tsx
function email({
  person,
  subject,
  body,
}: {
  person: Person;
  subject: string;
  body: string;
}) {
  // ...
}
```

## Item9 타입 단언보다는 타입 선언을 사용하기

- 타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 두 가지 이다.

```tsx
interface Person {
  name: string;
}
const alice: Person = { name: "Alice" }; // 타입은 Person
const bob = { name: "Bob" } as Person; // 타입은 Person
```

- 이 두 가지 방법은 결과가 같아 보이지만 그렇지 않습니다. 첫 번째는 변수에 ‘타입 선언’을 붙여서 그 값이 선언된 타입임을 명시한다. 두 번째는 ‘타입 단언’을 수행한다 그러면 타입스크립트가 추론한 타입이 있더라도 Person타입으로 간주한다.
- 타입 단언보다 타입 선언을 사용하는게 낫다

```tsx
const alice: Person = {};
// ------- 'Person' 유형에 필요한 'name' 속성이 '{}‘ 유형에 없습니다.
const bob = {} as Person; // 오류 없음
```

- 타입 선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사한다. 앞의 예제는 그러지 못했기 때문에 타입스크립트가 오류를 표시했다. 타입 단언은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 하는 것 입니다.
- 타입 선언문에서는 잉여 속성 체크가 동작했지만, 단언문에서는 적용되지 않습니다.
- 타입 단언이 꼭 필요한 경우가 아니라면, 안전성 체크도 되는 타입 선언을 사용하는 것이 좋습니다.
- 자주 쓰이는 특별한 문법(!)을 사용해서 null이 아님을 단언하는 경우도 있다.

```tsx
const elNull = document. getElementByld (1 foo'); // 타입은 HTMLElement | null
const el = document. getElementByld (' foo')!; // 타입은 HTMLElement
```

- 변수의 접두사로 쓰인 !는 boolean의 부정문이다. 그러나 접미사로 쓰인 !는 그 값이 null이 아니라는 단언문으로 해석 된다.
- 모든 타입은 unknown의 서브 타입이기 때문에 unknown이 포함된 단언문은 항상 동작한다. unknown 단언은 임의의 타입 간에 변환을 가능케 하지만, unknown을 사용한 이상 적어도 무언가 위험한 동작을 하고 있다는걸 알 수 있다.

## Item10 객체 래퍼 타입 피하기

- 기본형 값에 메서드를 제공하기 위해 객체 래퍼타입이 어떻게 쓰인느지 이해해야 합니다. 직접 사용하거나 인스턴스를 생성하는 것은 피해야 한다.
- 타입스크립트 객체 래퍼 타입은 지양하고, 대신 기본형 타입을 사용해야 합니다. String 대신 string,Number 대신 number, Boolean 대신 boolean, Symbol 대신 symbol, BigInt 대신 bigint를 사용해야 한다.

## Item 11 잉여 속성 체크의 한계 인지하기

- 타입이 명시된 변수에 객체 리터럴을 할당할 때 타입스크립트는 해당 타입의 속성이 있는지, 그리고 ‘그 외의 속성은 없는지’ 확인 한다.

```tsx
interface Room {
	numDoors: number;
	ceilingHeightFt: number;
}
const r: Room = {
	numDoors: 1,
	ceilingHeightFt: 10,
	elephant: 'present',
// ~~~ 개체 리터럴은 알려진 속성만 지정할 수 있으며
// 'Room'형식에 'elephant'이（가） 없습니다.
}；

const obj = {
	numDoors: 1,
	ceilingHeightFt: 10,
	elephant: 'present',
}；
const r: Room = obj; // 정상
```

- obj의 타입은 Room 타입의 부분 집합을 포함하므로, Room에 할당 가능하며 타입 체커도 통과한다.
- 첫 번째 예제에서는, 구조적 타입 시스템에서 발생할 수 있는 중요한 종류의 오류를 잡을 수 있도록 ‘잉여 속성 체크’라는 과정이 수행되었다.
- 잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않음으로써, 앞에서 다룬 예제 같은 문제점을 방지할 수 있다.
- 잉여 속성 체크는 타입 단언문을 사용할 때에도 적용되지 않습니다.

## Item12 함수 표현식에 타입 적용하기

- 자바스크립트에서는 함수 문장과 함수 표현식을 다르게 인식한다.
- 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다는 장점 때문에 타입스크립트에서는 함수표현식을 사용하는게 좋다.
- 다른 함수의 시그니처를 참조하려면 typeof fn을 사용하면 된다.

## Item13 타입과 인터페이스의 차이점 알기

- 타입스크립트에서 명명된 타입을 정의하는 방법은 두 가지가 있다.

```tsx
type TState = {
  name: string;
  capital: string;
};

interface IState {
  name: string;
  capital: string;
}
```

- 명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없다
- 인덱스 시그니처는 인터페이스와 타입에서 모두 사용할 수 있다.

```tsx
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}
```

- 또한 함수 타입도 인터페이스나 타입으로 정의할 수 있다.

```tsx
type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}
const toStrT: TFn = (x) => "" + x; // 정상
const toStrl: IFn = (x) => "" + x; // 정상
```

- 이런 단순한 함수 타입에는 타입 별칭이 더 나은 선택이겠지만, 함수 타입에 추가적인 속성이 있다면 타입이나 인터페이스 어떤 것을 선택하든 차이가 없다.
- 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지 못한다는 것이다. 복잡한 타입을 확장하고 싶다면 타입과 &를 사용해야한다.

```tsx
type NamedVariable = (Input | Output) & { name: string };
```

- 이 타입은 인터페이스로 표현할 수 없다. type 키워드는 유니온이 될 수도있고, 매핑된 타입 또는 조건부 타입 같은 고급 기능에 활용되기도 한다.
- 튜플 배열 타입도 type 키워드를 이용해 간결하게 표현할 수 있다.

```tsx
type Pair = [number, number];
type StringList = string[];
type NamedNums = [string, ...number[]];
```

- 인터페이스는 타입에 없는 몇 가지 기능이 있다. 그중 하나는 바로 ‘보강’이 가능하다는 것이다. 이번 아이템 처음에 등장했던 State예제에 population 필드를 추가할 때 보강 기법을 사용할 수 있다.

```tsx
interface IState {
	name: string;
	capital: string;
}
interface IState {
	population: number;
}
const Wyoming: IState = {
	name: 'Wyoming',
	capital: 'Cheyenne1,
	population: 500_000
}; // 정상
```

- 이 예제처럼 속성을 확장하는 것을 ‘선언 병합’이라고 한다. 타입 선언 파일을 작성할 떄는 선언 병함을 지원하기 위해 반드시 인터페이스를 사용해야 하며 표준을 따라야 한다.
- 프로젝트에서 어떤 문법을 사용할지 결정할 때 한 가지 일관된 스타일을 확립하고, 보강 기법이 필요한지 고려해야 한다.

## Item14 타입 연산과 제너릭 사용으로 반복 줄이기

```tsx
console.log('Cylinder 1x1',
	'Surface area:', 6.283185 *1*1+ 6.283185 *1*1,
	'Volume:3.14159 *1*1*1);
console.log('Cylinder 1 x 2
	'Surface area:6.283185 *1*1+ 6.283185 *2*1,
	'Volume:3.14159 *1*2*1);
console.log('Cylinder 2xl\
	’Surface area:6.283185 *2*1+ 6.283185 *2*1,
	’Volume:3.14159 *2*2*1);
```

- 비슷한 코드가 반복되면 보기가 불편하다, 코드에서 함수, 상수, 루프의 반복을 제거해 코드를 개선해 보자

```tsx
const surfaceArea = (r, h) => 2 * Math.PI * r * (r + h);
const volume = (r, h) => Math.PI * r * r * h;
for (const [r, h] of [
  [1, 1],
  [1, 2],
  [2, 1],
]) {
  console.log(
    `Cylinder ${r} x ${h}`,
    `Surface area: ${surfaceArea(r, h)}`,
    `Volume: ${volume(r, h)}`
  );
}
```

- 이게 바로 같은 코드를 반복하지 말라는 DRY(don’t repeat yourself)원칙이다.
- 타입 중복은 코드 중복만큼 많은 문제를 발생시킨다. 반복을 줄이는 가장 간단한 방법은 타입에 이름을 붙이는 것이다.

```tsx
function distance(a: { x: n나mber; y: n나mber }, b: { x: n나mber; y: number }) {
  return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}
```

- 코드를 수정해 타입에 이름을 붙여 보자

```tsx
interface Point2D {
  x: number;
  y: number;
}
function distance(a: Point2D, b: Point2D) {
  /*...*/
}
```

- 이 코드는 상수를 사용해서 반복을 줄이는 기법을 동일하게 타입 시스템에 적용한 것이다.
- 전체 애플리케이션의 상태를 표현하는 State타입과 단지 부분만 표현하는 TopNavState가 있는 경우를 살펴보자

```tsx
interface State {
	userid: string;
	pageTitle: string;
	recentFiles: string!];
	pageContents: string;
}

interface TopNavState {
	userid: string;
	pageTitle: string;
	recentFiles: string[];
}
```

- State의 부분집합으로 TopNavState를 정의하는 것이 바람직해 보인다. 이 방법이 전체 앱의 상태를 하나의 인터페이스로 유지할 수 있게 해준다.
- State를 인덱싱하여 속성의 타입에서 중복을 제거할 수 있다

```tsx
type TopNavState = {
	userid: State['userid'];
	pageTitle: State['pageTitle'];
	recentFiles: State['recentFiles'];
}；
```

- 여전히 반복되는 코드가 존재한다. 이때 ‘매핑된 타입’을 사용하면 좀 더 나아진다.

```tsx
type TopNavState = {
	[k in 'userid' | 'pageTitle' | 'recentFiles']: State[k]
}；
```

- 매핑된 타입은 배열의 필드를 루프 도는 것과 같은 방식이다. 이 패턴은 표준 라이브러리에서도 일반적으로 찾을 수 있으며, Pick이라고 한다.

```tsx
type Pick<T, K> = { [k in K]: T[k] };
```

- 정의가 완전하지는 않지만 다음과 같이 사용할 수 있다.

```tsx
type TopNavState = Pick<Stater ’userid' | 'pageTitle' | 1recentFiles'>;
```

- 여기서 Pick은 제너릭 타입이다. Pick을 사용하는 것은 함수를 호출하는 것과 마찬가지다.
- 생성하고 난 다음에 업데이트가 되는 클래스를 정의한다면, update 메서드 매개변수의 타입은 생성자와 동일한 매개변수이면서, 타입 대부분이 선택적 필드가 됩니다.

```tsx
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
interface OptionsUpdate {
  width?: number;
  height?: number;
  color?: string;
  label?: string;
}
class UlWidget {
  constructor(init: Options) {
    /*•••*/
  }
  update(options: OptionsUpdate) {
    /* ... */
  }
}
```

- 매핑된 타입과 keyof를 사용하면 Options으로부터 OptionUpdate를 만들 수 있다.

```tsx
type OptionsUpdate = { [k in keyof Options]?: Options[k] };
```

- keyof는 타입을 받아서 속성 타입의 유니온을 반환한다.

```tsx
type OptionsKeys = keyof Options;
// 타입이 "wid나!,, | "height" | "color" | "label"
```

- ‘?’는 각 속성을 선택적으로 만든다. 이 패턴 역시 아주 일반적이며 표준라이브러리에 Partial이라는 이름으로 포함되어 있다.
- 제너릭 타입에서 매개변수를 제한할 수 있는 방법이 필요하다. 제너릭 타입에서 매개변수를 제한할 수 있는 방법은 extends를 사용하는 것이다. extends를 이용하면 제너릭 매개변수가 특정 타입을 확장하낟고 선언할 수 있다.
- 타입이 값의 집합이라는 관점에서 생각하면 extends를 ‘확장’이 아니라 ‘부분 집합’이라는 걸 이해하는데 도움이 된다.

## Item15 동적 데이터에 인덱스 시그니처 사용하기

- 자바스크립트의 장점중 하나는 바로 객체를 생성하는 문법이 간단하다는 것이다.

```tsx
const rocket = {
	name: 'Falcon 9',
	variant: 'Block 5',
	thrust: '7,607 kN’,
};
```

- 타입스크립트에서는 타입에 ‘인덱스 시그니처’를 명시하여 유연하게 매핑을 표현할 수 있다.

```tsx
type Rocket = {[property: string]: string};
	const rocket: Rocket = {
	name: 'Falcon 9',
	variant: 'vl.0',
	thrust: ‘4,940 kN',
}； // 정상
```

- [property: string]: string이 인데스 시그니처이며, 다음 세 가지 의미를 담고 있다.
- 키의 이름, 키의 타입, 값의 타입 이렇게 타입 체크가 수행되면 네 가지 단점이 드라난다.
- 잘못된 키를 포함해 모든 키를 혀용한다. name 대신 Name으로 작성해도 유효한 Rocket타입이 된다.
- 특정 키가 필요하지 않다. {}도 유효한 Rocket 타입이다.
- 키마다 다른 타입을 가질 수 없다. 예를 들어, thrust는 string이 아니라 number여야 할 수도 있다.
- 타입 스크립트 언어 서비스는 다음과 같은 경우에 도움이 되지 못한다.
  - name: 을 입력할 때, 키는 무엇이든 가능하기 떄문에 자동 완성 기능이 동작하지 않는다.
- 인덱스 시그니처는 부정확하므로 더 나은 방법을 찾아야한다.

## Item 16 number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

- 일반적으로 string 대신 number를 타입의 인덱스 시그니처로 사용할 이유는 많지 않다. 만약 숫자를 사용하여 인덱스할 항목을 지정한다면 Array또는 튜플 타입을 대신 사용하게 된다.
- 배열은 객체이므로 키는 숫자가 아니라 문자열이다. 인덱스 시그니처로 사용된 number 타입은 버그르 잡기 위한 순수 타입스크립트 코드이다.

## Item17 변경 관련된 오류 방지를 위해 readonly 사용하기

- 만약 함수가 매개변수를 수정하지 않는다면 readonly 로 선언하는 것이 좋다 readonly 매개변수는 인터페이스를 명확하게 하며, 매개변수가 변경되는 것을 방지한다.
- readonly를 사용하면 변경하면서 발생하는 오류를 방지할 수 있고, 변경이 발생하는 코드도 쉽게 찾을 수 있다.
- const와 readonly는 얕게 동작하는 것을 명심해야 한다.

```tsx
function arraySum(arr: readonly number[]) {
  let sum = 0,
    num;
  while ((num = arr.pop()) !== undefined) {
    //~~~'readonly number[]' 형식에 'pop' 속성이 없습니다.
    sum += num;
  }
  return sum;
}
```

- 이 오류 메세지를 자세히 살펴보자 readonly number[]는 ‘타입’이고, number[]와 구분되는 몇 가지 특징이 있다.
- number[]는 readonly number[]보다 기능이 많기 때문에, readonly number[]의 서브타입이 된다. 따라서 변경 가능한 배열을 readonly 배열에 할당할 수 있다. 하지만 그 반대는 불가능하다.

```tsx
const a: number[] = [1, 2, 3];
const b: readonly number[] = a;
const c: number[] = b;
// ~ 'readonly number[]' 타입은' readonly1 이므로
// 변경 가능한'number[]' 타입에 할당될 수 없습니다.
```

## Item 18 매핑된 타입을 사용하여 값을 동기화하기

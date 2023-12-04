# 1장 타입스크립트 알아보기

<aside>
💡 타입스크립트란 무엇이고, 타입스크립트를 어떻게 여겨야 하는지, 자바스크립트와는 어떤 관계인지 등을 알아보자

</aside>

## item 1 타입스크립트와 자바스크립트의 관계 이해하기

- 타입크립트는 자바스크립트의 상위집합이다. 자바스크립트 프로그램에 문법 오류가 없다면, 유효한 타입스크립트 프로그램이라고 할 수 있다.
- 타입스크립트는 자바스크립트의 상위집합이기 떄문에 .js파일에 있는 코드는 이미 타입스크립트라고 할 수 있다. 이러한 특성은 기존에 존재하는 자바스크립트 코드를 타입스크립트로 마이그레이션하는데 엄청난 이점이 된다. 기존 코드를 그대로 유지하면서 일부분에만 타입스크립트 적용이 가능하기 때문이다.
- 타입 시스템의 목표중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것이다. 타입스크립트가 정적 타입 시스템이라는 것은 바로 이런 특징을 말한다. 그러나 타입 체커가 모든 오류를 찾아내지는 않는다.
- 오류가 발생하지 않지만 의도아 다르게 동작하는 코드도 있다.

```jsx
const states = [
  { name: "Alabama", capital: "Montgomery" },
  { name: "Alaska", capital: "Juneau" },
  { name: "Arizona", capital: "Phoenix" },
  // ...
];
for (const state of states) {
  console.log(state.capitol);
}

// 실행시
undefined;
undefined;
undefined;
```

- 앞의 코드는 유효한 자바스크립트이며 어떠한 오류도 없이 실행된다. 그러나 루프 내의 state.capitol은 의도한 코드가 아닌 게 분명하다. 이러한 경우에 타입스크립트타입 체커는 추가적인 타입 구문 없이도 오류를 찾아낸다.

```jsx
for (const state of states) {
console•log(state.capitol);
// ~~~ ’capitol' 속성이 ... 형식에 없습니다
// ’capital’을사용하시겠습니까?
}
```

- 타입스크립트는 타입 구문 없이도 오류를 잡을 수 있지만, 티압 구문을 추가한다면 훨씬 더 많은 오류를 찾아 낼 수 있다. 코드의 ‘의도’가 무엇인지 타입 구문을 통해 타입스크립트에게 알려줄 수 있기 때문에 코드의 동작과 의도가 다른 부분을 찾을 수 있다.

```jsx
const states = [
{name: 'Alabama', capitol: 'Montgomery'},
{name: 'Alaska', capitol: ’Juneau’},
{name: 'Arizona', capitol: 'Phoenix'},
// ...
]；
for (const state of states) {
console.log(state.capital);
//----------- 'capital1 속성이 ... 형식에 없습니다.
// 'capitol*을 사용하시겠습니까?
}

```

- 위 와 같이 타입스크립트는 오타인지 정확한 판단을 못한다. 따라서 명시적으로 states를 선언하여 의도를 분명하게 하는 것이 좋다.

```jsx
nterface State {
name: string;
capital: string;
}
const states: Stated = [
{name: 'Alabama', capitol: 'Montgomery'}r
									//~~~~~~~~~~~~~~~~~
{name: 'Alaska', capitol: 'Juneau'},
									// ~~~~~~~~~~~~~~
{name: 'Arizona', capitol:'Phoenix'},
									//~~~~~~~~~~~개체 리터럴은 알려진 속성만 지정할 수 있지만
									// State' 형식에 capitol'이（가） 없습니다
								  // 'capital'울（를） 쓰려고 했습니까?
// ...
]；
for (const state of states) {
console.log(state.capital);
}
```

- 이제 오류가 어디에서 발생했는지 찾을 수 있고, 제시된 해결책도 올바르다. 의도를 명확히 해서 타입스크립트가 잠재적 문제점을 찾을 수 있게 했습니다.

```jsx
const states: State[] = [
{name: 'Alabama', capital: 'Montgomery'},
{name: 'Alaska', capitol: 1 Juneau'},
									// ~~~~~~'cspital'을쓰려고 했습니까?
{name: 'Arizona', capital: 'Phoenix'},
// ...
]；

```

- 타입스크립트 채택 여부는 온전히 나의 선택에 달렸다. 타입스크립트의 도움을 받으면 오류가 적은 코드를 작성할 수 있다. 그러나 앞의 이상한 코드들처럼 불필요한 매개변수를 추가해서 함수 호출하는 것을 당연하게 여긴다면 차라리 타입스크립트를 쓰지 않는 게 낫다.
- 타입 시스템이 정적 타입의 정확성을 보장해 줄 것 같지만 그렇지도 않다. 애초에 타입 시스템은 그런 목적으로 만들어지지도 않았다. 정확성을 보장하는 것이 중요하다면 Reason이나 Elm 같은 언어를 선택하는 것이 좋다.
- 이 언어들은 런타입 안정성을 보장하는 대신 자바스크립트의 상위집합이 아니기 때문에 마이그레이션 과정이 훨씬 복잡하다.

## Item 2 타입스크립트 설정 이해하기

- 다음 코드가 오류 없이 타입 체커를 통과할 수 있을지 생각해 보자.
  ```jsx
  function add(a, b) {
    return a + b;
  }
  add(10, null);
  ```
- 타입스크립트 컴파일러는 매우 많은 설정을 가지고 있다. 현재 시점에는 설정이 거의 100개에 이른다. 이설정들은 커맨드 라인을 사용해 할 수 있다.

```jsx
$ tsc —noImplicitAny program.ts

tsconfig.json 설정 파일을 통해서도 가능하다.

{
	"compileroptions": {
	"noImplicitAny": true
	}
}
```

- 가급적 설정 파일을 사용하는 것이 좋다. 그래야만 타입스클비트를 어떻게 사용할 계획인지 동료들이나 다른 도구들이 알 수 있다. 설정 파일은 tsc —init만 실행하면 간단히 생성된다.
- 타입스크립트의 설정들은 어디서 소스 파일으 찾을지, 어떤 종류의 출력을 생성할지 제어하는 내용이 대부분이다. 그런데 언어 자체의 핵심 요소들을 제어하는 설정도 있다.
- 타입 스크립트는 어떻게 설정하느냐에 따라 완전히 다른 언어처럼 느껴질 수 있다. 설정을 제대로 사용하려면 noImplicitAny와 strictNullCjecks를 이해해야 한다.
- noImplicitAny는 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어한다. 다음 코드는 noImplicitAny가 해제되어 있을 때에는 유효하다.

```jsx
function add(a, b) {
  return a + b;
}
```

- any 타입을 매개변수에 사용하면 타입 체커는 속절없이 무력해진다. any는 유용하지만 매우 주의해서 사용해야 한다.
- any 코드를 넣지 않았지만, any 타입으로 간주되기 떄문에 이를 암시적 any라고 부릅니다. 그런데 같은 코드임에도 noImplicitAny가 설정되었다면 오류가 됩니다. 이 오류들은 명시적으로 : any라고 선언해 주거나 더 분명한 타입을 사용하면 해결할 수 있다.

```jsx
function add(a: number, b: number) {
  return a + b;
}
```

- 타입스크립트는 타입 정보를 가질 떄 가장 효과정이기 떄문에, 되도록이면 noImplicitAny를 설정해야 합니다. 코드를 작성 할 때마다 타입을 명시하도록 해야 한다. 그러면 타입스크립트가 문제를 발견하기 수월해지고, 코드의 가독성이 좋아지며, 개발자의 생산성이 향항된다.
- noImplicitAny 설정 해제는, 자바스크립트로 되어 있는 기존 프로젝트를 타입스크립트로 전환하는 상황에만 필요합니다.
- strictNullChecks는 null과 undefined가 모든 타입에서 허용되는지 확인하는 설정이다.
- 다음은 strictNullChecks가 해제되었을 때 유효한 코드입니다.

```jsx
const x: number = null; // 정상, null은 유효한 값입니다.
// 그러나 strictNullChecks를 설정하면 오류가 됩니다.
const x: number = null;
// ~ 'null' 형식은 'number' 형식에 할당할 수 없습니다.
```

- null을 허용하려고 한다면, 의도를 명시적으로 드러냄으로써 오류르 고칠 수 있다.
- 만약 null을 허용하지 않으려면, 이 값이 어디서부터 왔는지 찾아야 하고, null을 체크하는 코드나 단언문을 추가해야 한다.

```jsx
const el = document.getElementByld('status');
	el.textcontent = 'Ready';
// — 개체가 'null’인 것 같습니다.
	if (el) {
		el.textcontent = 'Ready'; // 정상, null은 제외됩니다.
	}
	el!.textcontent = 'Ready'; // 정상, el이 null이 아님을 단언합니다.
```

- strictNullChecks는 null과 undefined 관련된 오류를 잡아 내는 데 많은 도움이 되지만, 코드 작성을 어렵게 한다. 새 프로젝트를 시작한다면 가급적 strictNullChecks를 설정하는 것이 좋다.
- 언어에 의미적으로 영향을 미치는 설정들이 많지만 noImplicitAny와 strictNullChecks만큼 중요한 것은 없다. 이 모든 체크를 설정하고 싶다면 strict 설정을 하면 된다. 타입스크립트에 struct 설정을 하면 대부분의 오류를 잡아낸다.

## Item3 코드 생성과 타입이 관계없음을 이해하기

- 큰 그림에서 보면, 타입스크립트 컴파일러는 두 가지 역할을 수행한다.
  - 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일한다.
  - 코드의 타입 오류를 체크한다.
- 여기서 놀라운 점은 이 두가지가 서로 완벽히 독립적이라는 것이다.

### **타입 오류가 있는 코드도 컴파일이 가능하다.**

- 컴파일은 타입 체크와 독립적으로 동작하기 떄문에, 타입 오류가 있는 코드도 컴파일이 가능하다.

### **런타임에는 타입체크가 불가능하다.**

```jsx
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    // -------------- 'Rectangle’은 형식만 참조하지만,
    // 여기서는 값으로 사용되고 있습니다.
    return shape.width * shape.height;
    // --------- 'Shape' 형식에 'height' 속성이 없습니다.
  } else {
    return shape.width * shape.width;
  }
}
```

- 타입스크립트의 타입은 제거 가능합니다. 실제로 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거되어 버립니다.
- 앞의 코드에서 다루고 있는 shape 타입을 명확하게 하려면, 런타임에 타입 정보를 유지하는 방법이 필요하다. 하나의 방법은 height 속성이 존재하는지 체크해 보는 것입니다.

```jsx
function caleuLateArea(shape: Shape) {
  if ("height" in shape) {
    shape; // 타입이 Rectangle
    return shape.width * shape.height;
  } else {
    shape; // 타입이 Square
    return shape.width * shape.width;
  }
}
```

- 속성 체크는 런타임에 접근 가능한 값에만 관련되지만, 타입 체커 역시도 shape의 타입을 Rectangle로 보정해 주기 때문에 오류가 사라진다.
- 타입 정보를 유지하는 또 다른 방법으로는 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 ‘태그’ 기법이 있다.

```jsx
interface Square {
  kind: "square";
  width: number;
}
interface Rectangle {
  kind: "rectangle";
  height: number;
  width: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape.kind === "rectangle") {
    shape; // 타입이 Rectangle
    return shape.width * shape.height;
  } else {
    shape; // 타입이 Square
    return shape.width * shape.width;
  }
}
```

- 타입과 값을 둘 다 사용하는 기법도 있다 타입을 클래스로 만들면 된다. Square와 Rectangle을 클래스로 만들면 오류를 해결할 수 있다.

```jsx
class Square {
	constructor(public width: number) {}
}
class Rectangle extends Square {
	constructor^니blic width: number, public height: number) {
		super(width);
	}
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
	if (shape instanceof Rectangle) {
		shape; // 타입이 Rectangle
		return shape.width * shape.height;
	} else {
		shape; // 타입 Square
		return shape.width * shape.width; // 정상
	}
}

```

- 인터페이스는 타입으로만 사용 가능하지만, Reactangle을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없다.
- 타입 연선은 런타임에 영향을 주지 않는다.
  - 다음 코드는 타입 체커를 통과하지만 잘못된 방법
  ```jsx
  function asNumber(val: number | string): number {
  	return val as number;
  }
  ```
- 코드에 아무런 정제 과정이 없다 as number는 타입 연산이고 런타임 동작에는 아무런 영향을 미치지 않는다. 값을 정제하기 위해서는 런타임의 타입을 체크해야하고 자바스크립트 연산을 통해 변환을 수행해야 한다.

### 런타임 타입은 선언된 타입과 다를 수 있다.

- 타입스크립트에서는 런타임 타입과 선언된 타입이 맞지 않을 수 있습니다.
- 타입이 달라지는 혼란스러운 상황을 가능한 한 피해야 합니다. 선언된 타입이 언제든지 달라 질 수 있다.

### 타입스크립트 타입으로는 함수를 오버로드할 수 없다.

- c++같은 언어는 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용한다. 이를 ‘함수 오버로딩’이라고 한다. 그러나 타입스크립트에서는 타입과 런타임의 동작이 무관하기 때문에, 함수 오버로딩은 불가능하다.
- 하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만, 구현체는 오직 하나뿐이다.

```jsx
function add(a: number, b: number): number;
function add(a: string, b: string): string;

function add(a, b) {
	return a + b;
}
const three = add(l, 2); // 타입이 number
const twelve = add('1', '2'); // 타입이 string
```

- add에 대한 처음 두 개의 선언문은 타입 정보를 제공할 뿐이다. 이 두 선언문은 타입스크립트가 자바스크립트로 변환되면서 제거되며, 구현체만 남게된다.

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.

## Item4 구조적 타이핑에 익숙해지기

- 자바스크립트는 본질적으로 덕 타이핑 기반이다. 구조적 타이핑을 제대로 이해한다면 오류인 경우와 오류가 아닌 경우의 차이를 알 수 있고, 더욱 견고한 코드를 작성할 수 있다.

```jsx
interface Vector2D {
	x: number;
	y: number;
}

function calculateLength(v: Vector2D) {
	return Math.sqrt(v.x * v.x + v.y * v.y);

// 이제 이름이 들어간 벡터를 추가합니다.
interface NamedVector {
	name: string;
	x: number;
	y: number;
}

const v: NamedVector = { x: 3, y: 4, name: 'Zee' };
	calculateLength(v); // 정상, 결과는 5

```

- NamedVector는 number 타입의 x와 y 속성이 있기 때문에 calculatelength 함수로 호출 가능하다.
- 흥미로운 점은 Vector2D와 NamedVector의 관게를 전혀 선언하지 않았다는 것이다. 타입스크립트 타입 시스템은 자바스크립트의 런타임 동작을 모델링 한다. NamedVector의 구조가 Vector2D와 호환 되기 떄문에calculatelength 호출이 가능하다.
- 여기서 ‘구조적 타이핑’이라는 용어가 사용된다.
- 구조적 타이핑 때문에 문제가 발생하기도 한다.

## Item5 any 타입 지양하기

- 타입스크립트의 타입 시스템은 점진넉이고 선택적이다. 코드에 타입을 조금씩 추가할 수 있기 때문에 점진적이며, 언제든지 타입 체커를 해제할 수 있기 때문에 선택적이다. 이 기능들의 핵심은 any타입이다.
- 일부 특별한 경우를 제외하고 any를 사용하면 타입스크립트의 수많은 장점을 누릴 수 없다.

### any 타입에는 타입 안전성이 없다.

- age는 number 타입으로 선언 되었다. any를 사용하면 string타입을 할당할 수 있게 된다. 타입 체커는 선언에 따라 number타입으로 판단할 것이고 혼돈은 걷잡을 수 없게 된다.

### any는 함수 시그니처를 무시해 버린다.

```jsx
function calculateAge(birthDate: Date): number {
	// ...
}
let birthDate: any = '1990-01-19';
calculateAge（birthDate）; // 정상
```

- birthDate 매개변수는 string이 아닌 Date 타입이어야 한다. any 타입을 사용하면 calculateAge의 시그니처를 무시하게 된다. 자바스크립트에서는 종종 함시적으로 타입이 변환 되기 때문에 이런 경우 특히 문제가 될 수 있다.

### any 타입에는 언어 서비스가 적용되지 않는다.

- 타입스크립트 언어 서비스는 자동완성 기능과 적절한 도움말을 제공한다. 그러나 any 타입인 심벌을 사용하면 아무런 도움을 받지 못한다.
- 타입스클비트의 모토는 ‘확장 가능한 자바스크립트이다.’ ‘확장’의 중요한 부분은 바로 타입스크립트 경험의 핵심 요소인 언어 서비스이다. 언어 서비스를 제대로 누려야 생산성이 향상된다.

### any 타입은 코드 리팩터링 때 버그를 감춘다.

### any 타입설게를 감춰버린다.

### any는 타입시스템의 신뢰도를 떨어뜨린다.

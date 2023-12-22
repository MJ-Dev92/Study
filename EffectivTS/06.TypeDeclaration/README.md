# 6장 타입 선언과 @types

<aside>
💡 6장에서는 타입스크립트에서 의존성이 어떻게 동작하는지 설명하여 의존성에 대한 개념을 잡을 수 있게 한다. 의존성 관리를 하다가 맞닥뜨릴 수 있는 몇 가지 문제를 보여 주고 해결하는 방법을 찾아봅시다. 제대로 된 타입 선언문을 작성하여 공개하는 것은 프로젝트뿐만 아니라 타입 스크립트 전체 커뮤니티에 기여하는 일이기도 합니다.

</aside>

## Item 45 devDependencies에 typescript와 @types 추가하기

- 세 가지 의존성
  - dependencies
  - devDependencies
  - peerDependencies
- 이 세 가지 의존성 중에서는 dependencies와 devDependencies가 일반적으로 사용된다.
- 개발자라면 라이브러리를 추가할 때 어떤 종류의 의존성을 사용해야 하는지 알고 있어야한다. 타입스크립트는 개발 도구일 뿐이고 타입 정보는 런타임에 존재하지 않기 때문에, 타입스크립트와 관련된 라이브러리는 일반적으로 devDependencies에 속한다.
- 모든 타입스크립트 프로젝트에서 공통적으로 고려해야 할 의존성 두 가지를 살펴보자
  - 첫 번째, 타입스크립트 자체 의존성을 고려해야 한다. 타입스크립트를 시스템 레벨로 설치할 수도 있지만, 다음 두가지 이유 때문에 추천하지 않는다.
    - 팀원들 모두가 항상 동일한 버전을 설치한다는 보장이 없다
    - 프로젝트를 셋업할 때 별도의 단계가 추가된다.
  - 따라서 타입스크립트를 시스템 레벨로 설치하기 보다는 devDependencies에 넣는게 좋다. devDependencies에 포함되어 있다면, npm install을 실행할 때 팀원들 모두 항상 정확한 버전의 타입스크립트를 설치할 수 있다.
  - 두 번째, 타입 의존성(@types)을 고려해야 한다. 원본 라이브러리 자체가 dependencies에 있더라도 @types 의존성은 devDependencies에 있어야 한다.
  - 예를 들어, 리액트의 타입 선언과 리액트를 의존성에 추가하려면 다음처럼 실행한다.
  ```html
  $ npm install react $ npm install --save-dev @types/react
  ```
  - 그러면 다음과 같은 package.json 파일이 생성된다.
  ```tsx
  {
  	"devDependencies": {
  	"@types/react": "
  	스16.8.19",
  	"typescript": "^3.5.3"
  },
  	"dependencies": {
  		"react": "^16.8.6"
  	}
  }
  ```
  - 이 예제의 의도는 런타임에 @types/react와 typescript에 의존하지 않겠다는 것이다.

## Item 46 타입 선언과 관련된 세 가지 버전 이해하기

- 실제로 타입스크립트는 알아서 의존성 문제를 해결해 주기는 커녕, 의존성 관리를 오히려 더 복잡하게 만든다. 왜냐하면 타입스크립트를 사용하면 다음 세 가지 사항을 추가로 고려해야하기 때문이다.
  - 라이브러리의 버전
  - 타입 선언(@types)의 버전
  - 타입스크립트의 버전
- 타입스크립트에서 일반적으로 의존성을 사용하는 방식은 다음과 같다. 특정 라이브러리를 dependencies로 설치하고, 타입 정보는 devDependencies로 설치한다.

```html
$ npm install react + react@16.8.6 $ npm install —save-dev @types/react +
@types/react@6.8.19
```

- 메이저 버전과 마이너 버전이 일치하지만 패치 버전은 일치하지 않는다는 점에 주목하자 앞선 예제의 경우 라이브러리 자체보다 타입 선언에 더 많은 업데이트가 있다.
- 그러나 실제 라이브러리와 타입 정보의 버전이 별도로 관리되는 방식은 다음 네 가지 문제가 있다.
  - 첫 번째, 라이브러리를 업데이트했지만 실수로 타입 선언은 업데이트하지 않는 경우이다. 이런 경우 라이브러리 업데이트와 관련된 새로운 기능을 사용하려 할 떄마다 타입 오류가 발생하게 된다.일반적인 해결책은 타입 선언도 업데이트하여 라이브러리와 버전을 맞추는 것이다.
  - 두 번째, 라이브러리보다 타입 선언의 버전이 최신인 경우이다. 이런 경우는 타입 정보 없이 라이브러리를 사용해 오다가 타입 선언을 설치하려고 할 때 뒤늦게 발생한다. 첫 번째 문제와 비슷하지만 버전의 대소 관계가 반대다. 해결책은 라이브러리와 타입 선언의 버전이 맞도록 라이브러리 버전을 올리거나 타입 선언의 버전을 내리는 것이다.
  - 세 번째, 프로젝트에서 사용하는 타입스크립트 버전보다 라이브러리에서 필요로 하는 타입스크립트 버전이 최신인 경우이다. 이 오류를 해결하려면 프로젝트의 타입스크립트 버전을 올리거나, 라이브러리 타입 선언의 버전을 원래대로 내리거나, declare module 선언으로 라이브러리의 타입 정보를 없애 버리면 된다.
  - 네 번째, @types 의존성이 중복될 수도 있다. @types/bar가 현재 프로직트와 호환 되지 않는 버전의 @types/foo에 의존한다면 npm은 중접된 폴더에 별도로 해당 버전을 설치하여 문제를 해결하려고 한다.
  ```html
  node_modules/ @types/ foo/ index.d.ts @1.2.3 bar/ index.d.ts node_modules/
  @types/ foo/ index.d.ts @2.3.4
  ```
  - 전역 네임스페이스에 있는 타입 선언 모듈이라면 대부분 문제가 발생한다. 해결책은 보통 @types/foo를 업데이트하거나 @types/bar를 업데이트해서 서로 버전이 호환되게 하는 것이다.
  - 타입 선언을 라이브러리에 포함하는 것과 DefinitelyTyped에 공개하는 것 사이의 장단점을 이해해야한다. 타입스크립트로 작성된 라이브러리라면 타입 선언을 자체적으로 포함하고, 자바스크립트로 작성된 라이브러리라면 타입 선언을 DefinitelyTyped에 공개하는 것이 좋다.

## Item 47 공개 API에 등장하는 모든 타입을 익스포트하기

- 타입스크립트를 사용하다 보면, 언젠가는 서드파티의 모듈에서 익스포트되지 않은 타입 정보가 필요한 경우가 생긴다. 다른 관점으로 생각해 보면, 라이브러리 제작자는 프로젝트 초기에 타입 익스포트부터 작성해야 한다는 의미이다.

```tsx
interface SecretName {
  first: string;
  last: string;
}
interface SecretSanta {
  name: SecretName;
  gift: string;
}
export function getGift(name: SecretName, gift: string): SecretSanta {
  // ...
}
```

- 해당 라이브러리 사용자는 SecretName와 SecretSanta를 직접 임포트할 수 없고, getGift만 임포트가 가능하다. 추출하는 한 가지 방법은 Parameters와 ReturnType 제너릭 타입을 사용하는 것이다.

```tsx
type MySanta = ReturnType<typeof getGift>; // SecretSanta
type MyName = Parameters<typeof getGift>[0]; // SecretName
```

- 공개 메서드에 등장한 어떤 형태의 타입이든 익스포트하자. 어차피 라이브러리 사용자가 추출할 수 있으므로, 익스포트하기 쉽게 만드는 것이 좋다.

## Item 48 API 주석에 TSDoc 사용하기

```tsx
// 일반주석
// 인사말을 생성합니다. 결과는 보기 좋게 꾸며집니다.
function greet(name: string, title: string) {
  return "Hello ${title} ${name}";
}

//JSDoc 스타일 주석
/** 인사말을 생성합니다. 결과는 보기 좋게 꾸며집니다. */
function greetJSDoc(name: string, title: string) {
  return "Hello ${title} ${name}";
}
```

- 대부분의 편집기는 함수가 호출되는 곳에서 함수에 붙어 있는 JSDoc스타일의 주석을 툴팁으로 표시해준다. 그러나 인라인 주석은 편집기가 표시해주지 않는다
- 타입스크립트 언어 서비스가 JSDoc 스타일을 지원하기 때문에 적극적으로 활용하는 것이 좋다. 만약 공개 API에 주석을 붙인다면 JSDoc 형태로 작성해야 한다. 타입스크립트 관점에서는 TSDoc이라고 불린다.
- @param, @returns 구문과 문서 서식을 위해 마크다운을 사용할 수 있다.
- 주석에 타입 정보를 포함하면 안된다.

## Item 49 콜백에서 this에 대한 타입 제공하기

- let이나 const로 선언된 변수가 렉시컬 스코프인 반면, this는 다이나믹 스코프이다. 다이나믹 스코프의 값은 ‘정의된’ 방식이 아니라 ‘호출된’ 방식에 따라 달라진다.
- this는 전형적으로 객체의 현재 인스턴스를 참조하는 클래스에서 가장 많이 쓰인다.

```tsx
class C {
	vals = [1, 2, 3];
	logSquaresO {
		for (const val of this.vals) {
			console.log(val * val);
		}
	}
}
const c = new C();
c. logSquares();

// 실행시
// 1
// 4
// 9
```

- logSquares를 외부 변수에 넣고 호출해보자

```tsx
const c = new C();
const method = c.logSq나ares;
method();

// Uncaught TypeError: undefined의 'vals' 속성을 읽을 수 없습니다.0
```

- c.logSquares()가 실체로는 두 가지 작업을 수행하기 때문에 문제가 발생한다.
- C.prototype.logSquares를 호출하고, this의 값을 c로 바인딩한다. 앞의 코드에서는 logSquares의 참조 변수를 사용함으로써 두 가지 작업을 분리했고, this의 값은 undefined로 설정된다.
- call을 사용하면 명시적으로 this를 바인딩하여 문제를 해결할 수 있다.

```tsx
const c = new C();
const method = c.logSq나ares;
method.call(c); // 제곱을 출력합니다.
```

- this는 어떤 것이든 바인딩할 수 있다. 심지어 DOM에서도 this를 바인딩할 수 있다.
- this 바인딩이 동작하는 원리를 이해해야한다.
- 콜백 함수에서 this를 사용한다면, 타입 정보를 명시해야 한다.

## Item 50 오버로딩 타입보다는 조건부 타입을 사용하기

- 다음 예제의 double 함수에 타입 정보를 추가해보자

```tsx
function double(x) {
  return x + x;
}
```

- double 함수에는 string 또는 number 타입의 매개변수가 들어올 수 있다.

```tsx
function double(x: number|string): number|string;
function double(x: any) { return x + x; }

const num = double(12); // string | number
const str = double('x’); // string | number
```

- duble에 number타입을 매개변수로 넣으면 number 타입을 반환한다. 그리고 string 타입을 매개변수로 넣으면 string 타입을 반환한다. 그러나 선언문에는 number 타입을 매개변수로 넣고 string 타입을 반환하는 경우도 포함되어 있다.
- 제너릭을 사용하면 이러한 동작을 모델링할 수 있다.

```tsx
function double<T extends number|string>(x: T): T;
function double(x: any) { return x + x; }

const num = double(12); // 타입이 12
const str = doublet 'x'); // 타입이 "x"
```

- 타입을 구체적으로 만들어 보려는 시도는 좋으나 너무 과했다. 또 다른 방법은 여러가지 타입 선언으로 분리하는 것이다. 타입스크립트에서 함수의 구현체는 하나지만 타입 선언은 몇 개든지 만들 수 있다

```tsx
function double(x: number): number;
function double(x: string): string;
function double(x: any) {
  return x + x;
}

const num = double(12); // 타입이 number
const str = double("x"); // 타입이 string
```

- 아직 버그가 남아있다. 타입스크립트는 오버로딩 타입 중에서 일치하는 타입을 찾을 때까지 순차적으로 검색한다. 그래서 오버로딩 타입의 마지막 선언까지 검색했을 때, string|number 타입은 string에 할당할 수 없기 때문에 오류가 발생한다.
- 세 번째 가장 좋은 해결책은 조건부 타입을 사용하는 것이다.

```tsx
function double<T extends number | string>(
  x: T
): T extends string ? string : number;
function double(x: any) {
  return x + x;
}
```

- 이 코드는 제너릭을 사용했던 예제와 유사하지만, 반환 타입이 더 정교하다. 조건부 타입은 자바스크립트의 삼항 연산자(?:)처럼 사용하면 된다.
  - T가 string의 부분 집합이면(string, 또는 문자열 리터럴, 또는 문자열 리터럴의 유니온) 반환 타입이 string이다.
  - 그 외의 경우는 반환 타입이 number이다.
- 조건부 타입이라면 앞선 모든 예제가 동작한다.

```tsx
const num = double(12); // number
const str = double("x"); // string

// function f(x: string | number): string | n나mber
function f(x: number | string) {
  return double(x);
}
```

- 오버로딩 타입이 작성하기 쉽지만, 조건부 타입은 개별 타입의 유니온으로 일반화하기 때문에 타입이 더 정확해진다.
- 각각의 오버로딩 타입이 독립적으로 처리되는 반면, 조건부 타입은 타입 체커가 단일 표현식으로 받아들이기 때문에 유니온 문제를 해결할 수 있다.

## Item 51 의존성 분리를 위해 미러 타입 사용하기

- 필수가 아닌 의존성을 분리할 때는 구조적 타이핑을 사용하자
- 공개한 라이브러리를 사용하는 자바스크립트 사용자가 @types 의존성을 가지지않게 해야 한다. 그리고 웹 개발자가 NodeJS관련된 의존성을 가지지 않게 해야한다.

## Item 52 테스팅 타입의 함정에 주의하기

- 프로젝트를 공개하려면 테스트 코드를 작성하는 것은 필수이며, 타입 선언도 테스트를 거쳐야 한다. 그러나 타입 선언을 테스트하기는 매우 어렵다
- 궁극적으로는 dtslint 또는 타입 시스템 외부에서 타입을 검사하는 유사한 도구를 사용하는 것이 더 안전하고 간단하다.

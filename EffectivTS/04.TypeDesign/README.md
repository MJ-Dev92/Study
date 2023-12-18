# 4장 타입 설계

<aside>
💡 4장을 이해하고 타입을 제대로 작성한다면, 인용문에서 비유한 것처럼 테이블(코드의 타입)뿐만 아니라 순서도(코드의 로직) 역시 쉽게 이해할 수 있을 것이다.

</aside>

## Item 28 유효한 상태만 타입을 지향하기

- 효과적으로 타입을 설계하려면, 유효한 상태만 표현할 수 있는 타입을 만들어 내는 것이 가장 중요하다. 타입 설계가 잘못된 상황을 알아보고, 예제를 통해 잘못된 설계를 바로잡아 보자.
- 애플리케이션에서 페이지를 선택하면, 페이지 내용을 로드하고 화면에 표시한다고 가정한 페이지의 상태는 다음처럼 설계 했다.

```tsx
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}

function renderPage(state: State) {
  if (state.error) {
    return `Error! Unable to load ${currentPage}: ${state.error}`;
  } else if (state.isLoading) {
    return `Loading ${currentPage}...`;
  }
  return `<hl>${currentPage}</hl>\n${state.pageText}`;
}
```

- isLoding이 true이고 동시에 error 값이 존재하면 로딩 중인 상태인지 오류가 발생한 상태인지 명확히 구분이 안간다. 필요한 정보가 부족하기 때문이다.

```tsx
async function changePage(state: State, newPage: string) {
  state.isLoading = true;
  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
    }
    const text = await response.text();
    state.isLoading = false;
    state.pageText = text;
  } catch (e) {
    state.error = "" + e;
  }
}
```

- changePage에는 많은 문제점이 있다.
  - 오류가 발생했을 때 state.isLoading을 false로 설정하는 로직이 빠져있다.
  - state.error를 초기화하지 않았기 때문에, 페이지 전환 중에 로딩 메세지 대신 과거의 오류 메세지를 보여주게 된다.
  - 페이지 로딩 중에 사용자가 페이지를 바꿔버리면 새 페에지에 오류가 뜨거나, 응답이 오는 순서에 따라 다른 페이지가 전환 될 수 있는 예상 어려운 일이 벌어진다.
- 문제는 바로 상태 값의 두 가지 속성이 동시에 정보가 부족하거나, 두 가지 속성이 충돌할 수 있다는 것이다. 무효한 상태가 존재하면 render()와 changePage() 둘 다 제대로 구현할 수 없게 된다.

```tsx
function renderPage(state: State) {
	const {currentPage} = state;
	const requeststate = state.requests[currentPage];
	switch (requeststate.state) {
		case  'pending':
			return 'Loading ${currentPage}...';
		case 'error':
			return `Error! Unable to load ${currentPage}: ${requeststate•error}`;
		case 'ok':
			return `<h1>${currentPage}</hl>\n${requeststate.pageText}`;
	}
}
async function changePage(state: State, newPage: string) {
	state.requests[newPage] = {state: ’pending1};
	state.currentPage = newPage;
	try {
		const response = await fetch(getUrlForPage(newPage));
		if (!response.ok) {
			throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
		}
		const pageText = await response.text();
		state.requests[newPage] = {state: 'ok', pageText};
	} catch (e) {
		state.requests[newPage] = {state: 'error', error: '' + e};
	}
}
```

- 현재 페이지가 무엇인지 명확하며, 모든 요청은 정확히 하나의 상태로 맞아 떨어진다. 무효가 된 요청이 실행되긴 하겠지만 UI에는 영향을 미치지 않는다.
- 유효한 상태만 표현하는 타입을 지향해야 한다. 코드가 길어지거나 표현하기 어렵지만 결국은 시간을 절약하고 고통을 줄일 수 있다

## Item 29 사용할 때는 너그럽게, 생성할 때는 업격하게

<aside>
💡 TCP 구현체는 견고성의 일반적 원칙을 따라야 한다. 당신의 작업은 엄격하게 하고, 다른 사람의 작업은 너그럽게 받아들여야 한다.

</aside>

- 함수의 시그니처에도 비슷한 규칙을 적용해야 한다. 함수의 매개변수는 타입의 범위가 넓어도 되지만, 결과를 반환할 때는 일반적으로 타입의 범위가 더 구체적이어야 한다.
- 3D 매핑 API는 카메라의 위치를 지정하고 경계 박스의 뷰포트를 계산하는 방법을 제공한다.

```tsx
declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;
```

- 카메라의 위치를 잡기 위해 viewportForBounds의 결과가 setCamera로 바로 전달될 수 있다면 편리할 것이다.

```tsx
interface CameraOptions {
  center?: LngLat;
  zoom?: number;
  bearing?: number;
  pitch?: number;
}
type LngLat =
  | { Ing: number; tat: number }
  | { Ion: number; tat: number }
  | [number, number];

type LngLatBounds =
  | { northeast: LngLat; southwest: LngLat }
  | [LngLat, LngLat]
  | [number, number, number, number];
```

- LngLat 타입도 setCamera의 매개변수 범위를 넓혀준다. 매개 변수로 {lng, lat}객체, {lon, lat}객체, 또는 순서만 맞다면[lng, lat] 쌍을 넣을 수도 있다. 이러한 편의성을 제공하여 함수 호출을 쉽게할 수 있다.
- 이름이 주어진 모서리, 위도/경도 쌍, 또는 순서만 맞다면 4튜플을 사용하여 경계를 지정할 수 있다.
- 보통 매개변수 타입은 반환 타입에 비해 범위가 넓은 경향이 있다 선택적 속성과 유니온 타입은 반환 타입보다 매개변수 타입에 더 일반적이다.
- 매개변수와 반환 타입의 재사용을 위해서 기본 형태(반환 타입)와 느슨한 형태(매개변수 타입)를 도입하는 것이 좋다.

## Item 30 문서에 타입 정보를 쓰지 않기

- 다음 코드에서 잘못된 부분을 찾아보자

```tsx
/**
* 전경색(foreground) 문자열을 반환합니다.
* 0개 또는1개의 매개변수를 받습니다.
* 매개변수가 없을 때는 표준 전경색을 반환합니다.
• 매개변수가 있을 때는 특정 페이지의 전경색을 반환합니다.
*/
function getForegroundColor(page?: string) {
  return page === "login" ? { r: 127, g: 127, b: 127 } : { r: 0, g: 0, b: 0 };
}
```

- 코드와 주석의 정보가 맞지 않는다. 둘 중 어느 것이 옳은지 판단하기에는 정보가 부족하며, 잘못된 상태라는 것만은 분명하다.
- 주석에는 세 가지 문제점이 있다.
  - 함수가 string 형태의 색깔을 반환한다고 적혀 있지만 실제로 {r, g, b} 객체를 반환한다.
  - 주석에는 함수가 0개 또는 1개의 매개변수를 받는다고 설명하고 있지만, 타입 시그니처만 보아도 명확하게 알 수 있는 정보다.
  - 불필요하게 장황하다. 함수 선언과 구현체보다 주석이 더 길다.
- 주석은 다음과 같이 개성할 수 있다.

```tsx
/** 애플리케이션 또는 특정 페이지의 전경색을 가져옵니다. */
function getForegroundColor(page?: string): Color {
  // ...
}
```

- 특정 매개변수를 설명하고 싶다면 JSDoc의 @param 구문을 사용하면 된다.
- 주석과 변수명에 타입 정보를 적는 것은 피해야 한다. 타입 선언이 중복 되는 것으로 끝나면 다행이지만 최악의 경우는 타입 정보에 모순이 발생하게 된다.
- 타입이 명확하지 않은 경우는 변수명에 단위 정보를 포함하는 것을 고려하는 것이 좋다.(Ex. timeMs 또는 temperatureC).

## Item 31 타입 주변에 null 값 배치하기

- 숫자들의 최솟값과 최댓값을 계산하는 extent 함수를 가정해보자

```tsx
function extent(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
    }
  }
  return [min, max];
}
```

- 이 코드는 타입체커를 통과하고(strictNullChecks 없이), 반환 타입은 number[]로 추론 된다. 하지만 여기에는 버그와 함께 설계적 결함이 있다.
- 최솟값이나 최댓값이 0인 경우, 값이 덧씌워져 버린다. 예를 들어, extent([0,1,2])의 결과는 [0, 2]가 아니라 [1, 2]가 된다.
- strictNullChecks 설정을 켜면 앞의 두 가지 문제점이 드러난다.

```tsx
const [min, max] = extent([0, 1, 2]);
const span = max - min;
// ~~ 개체가 'undefined'인 것 같습니다.
```

- extent 함수의 오류는 undefined를 min에서만 제외했고 max에서는 제외하지 않았기 떄문에 발생했다. 두 개의 변수는 동시에 초기화되지만, 이러한 정보는 타입 시스템에서 표현할 수 없다.
- 해결방법으로 min과 max를 한 객체안에 넣고 null이거나 null이 아니게 하면 된다.

```tsx
function extent(nums: number[]) {
  let result: [number, number] | null = null;
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])];
    }
  }
  return result;
}
```

- null아님 단언을 사용하면 min과 max를 얻을 수 있다.
- extent의 결괏값으로 단일 객체를 사용함으로써 설계를 개선했고, 타입스크립트가 null 값 사이의 관계를 이해할 수 있도록 했으며 버그도 제거했다.
- 한 값의 null 여부가 다른 값의 null 여부에 암시적으로 관련되도록 설계하면 안 된다.
- API 작성 시에는 반환 타입을 큰 객체로 만들고 반환 타입 전체가 null이거나 null이 아니게 만들어야 한다. 사람과 타입 체커 모두에게 명료한 코드가 될 것이다.

## Item32 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

- 유니온 타입의 속성을 여러 개 가지는 인터페이스에서 속성 간의 관계가 분명하지 않기 때문에 실수가 자주 발생하므로 주의해야 한다.
- 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기도 좋다
- 타입스크립트가 제어 흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야한다. 태그된 유니온은 타입스크립트와 매우 잘 맞기 떄문에 자주 볼 수 있는 패턴이다.

## Item33 string 타입보다 더 구체적인 타입 사용하기

- string 타입으로 변수를 선언하려 한다면, 혹시 그보다 더 좁은 타입이 적절하지는 않을지 검토해 봐야 한다.

```tsx
interface Album {
	artist: string;
	title: string;
	releaseDate: string; // YYYY-MM-DD
	recordingType: string; // 예를 들어, "live" 또는 "studio"
}

const kindOfBlue: Album = {
	artist: 'Miles Davis',
	title: 'Kind of Blue',
	releaseDate: 'August 17th, 1959', // 날짜 형식이 다릅니다.
	recordingType: 'Studio', // 오타（대문자 S）
}；

function recordRelease(title: string, date: string) {/*...*/}
recordRelease(kindOfBlue.releaseDate, kindOfBlue.title); // 오류여야 하지만 정상
```

- recordRelease 함수의 호출에서 매개변수들의 순서가 바뀌었지만, 둘 다 문자열이기 때문에 타입 체커가 정상으로 인식한다. 이를 ‘문자열을 남발하여 선언되었다’고 표현하기도 한다.
- releaseDate 필드와 recordingType을 적절한 타입으로 정의할 수 있다.

```tsx
type RecordingType = "studio" | "live";

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}
```

- 이러한 방식은 세 가지의 장점을 더 가지고 있다.
  - 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지된다.
  - 타입을 명시적으로 정의하고 해당 타입의 의키를 설명하는 주석을 붙여 넣을 수 있다.
  - keyof 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해진다.

## Item 34 부정확한 타입보다는 미완성 타입을 사용하기

- 일반적으로 타입이 구체적일수록 버그를 더 많이 잡고 타입스크립트가 제공하는 도구를 활용할 수 있다. 그러나 타입 선언의 정밀도를 높이는 일에는 주의를 기울여야 한다.

```tsx
interface Point {
  type: "Point";
  coordinates: number[];
}
interface LineString {
  type: "LineString";
  coordinates: number[][];
}
interface Polygon {
  type: "Polygon";
  coordinates: number[][][];
}
type Geometry = Point | LineString | Polygon; // 다른 것들도 추가될 수 있습니다.
```

- 큰 문제는 없지만 좌표에 쓰이는 number[]가 약간 추상적이다. 여기서 number[]는 튜플 타입으로 선언하는게 좋다.

```tsx
type GeoPosition = [number, number];
interface Point {
  type: "Point";
  coordinates: GeoPosition;
}
// ...
```

- 타입을 더 구체적으로 개선 해서 더 나은 코드가 된거 같지만 GeoJSON의 위치 정보에는 세 번째 요소인 고도가 있을 수 있고 또 다른 정보가 있을 수 있다.
- 결과적으로 타입이 부정확해졌다. 타입을 현재의 타입 선언을 그대로 사용하려면 사용자들은 타입 단언문을 도입하거나 as any를 추가해서 타입체커를 완전히 무시해야 한다.
- 타입 안전성에 불쾌한 골짜기는 피해야 한다. 타입이 없는 것보다 잘못된게 더 나쁘다.
- 정확한 타입을 모델링할 수 없다면, 부정확하게 모델리하지 말아야 한다. 또한 any와 unknown를 구별해서 사용하자.

## Item 35 데이터가 아닌 API와 명세를 보고 타입 만들기

- 타입 설계를 할때 예시 데이터가 아니라 명세를 참고해 타입을 생성해야한다. 명세를 참고해 타입을 생성하면 타입스크립트는 사용자가 실수를 줄일 수 있게 도와준다.
- 코드의 구석 구석까지 타입 안정성을 얻기 위해 API 또는 데이터 형식에 대한 타입 생성을 고려해야 한다.
- 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 떄문에 데이터보다는 명세로부터 코드를 생성하는 것이 좋다.

## Item 36 해당 분야의 용어로 타입 이름 짓기

- 이름 짓기 역시 타입 설계에서 중요한 부분이다. 엄선된 타입, 속성, 변수의 이름은 의도를 명확히 하고 코드와 타입의 추상화 수준을 높여 준다.

```tsx
interface Animal {
	name: string;
	endangered: boolean;
	habitat: string;
}
const leopard: Animal = {
	name: 'Snow Leopard',
	endangered: false,
	habitat: 'tundra',
}；
```

- 이 코드는 네 가지 문제가 있다.
  - name은 매우 일반적인 용어이다. 동물인지 학명인지 일반적인 명칭을 알 수 없다.
  - endangered 속성이 멸종 위기를 표현하기 위해 boolean 타입을 사용한 것이 이상하다.
  - habitat 속성은 너무 범위가 넓은 string타입, 서식지라는 뜻 자체도 불분명하기 때문에 다른 속성들보다 훨씬 모호하다.
  - 객체의 이름과 속성의 name이 다른 의도로 사용된 것인지 불분명하다.
- 반면 다음 코드의 타입 선언은 의미가 분명하다.

```tsx
interface Animal {
	commonName: string;
	genus: string;
	species: string;
	status: Conservationstatus;
	climates: KoppenClimate[];
}
type Conservationstatus = 'EX' | ‘EW1 | 'CR' | 'EN' | 'VU' | 'NT' | 'LC;
type KoppenClimate = |
	'Af' | 'Am' | 'As' | 'Aw' |
	'BSh' | 'BSk' | 'BWh' | 'BWk' |
	'Cfa' | 'Cfb' | 'Cfc' | 'Csa' | 'Csb' | 'Csc' | 'Cwa' | 'Cwb' | 'Cwc' |
	'Dfa' | 'Dfb' | 'Dfc' | 'Dfd' |
	'Dsa' | 'Dsb' | 'Dsc' | 'Dwa' | 'Dwb' | 'Dwc' | 'Dwd' |
	'EF' | 'ET';

const snowLeopard: Animal = {
	commonName: 'Snow Leopard',
	genus: 'Panthera',
	species: 'Uncia',
	status: 'VU', //취약종(vulnerable)
	climates: ['ET', 'EF', 'Dfd'], // 고산대(alpine) 또는 아고산대(subalpine)
}；
```

- 이 코드는 다음 세 가지를 개선 했다.
  - name은 더 구체적인 용어로 대체했다.
  - endangered는 동물 보호 등급에 대한 IUCN의 표준 분류 체계인 ConsercvationStatus타입의 status로 변경되었다.
  - habitat은 기후를 뜨하는 climates로 변경되었으며, 쾨펜 기후 분류를 사용했다.
- 코드로 표현하고자 하는 모든 분야에는 주제를 설명하기 위한 전문 용어들이 있다. 자체적으로 용어를 만들어 내려고 하지 말고, 해당 분야에 이미 존재하는 용어를 사용해야한다.
- 이런 용어들을 사용하면 사용자와 소통에 유리하며 타입의 명확성을 올릴 수 있다.
- 타입, 속성, 변수에 이름을 붙일 때 명심해야 할 세 가지 규칙이 있다.
  - 동일한 의미를 나타낼 때는 같은 용어를 사용해야 한다. 동의어를 사용하면 글을 읽을 때는 좋을 수 있지만, 코드에서 좋지 않다. 정말로 의미적으로 구분이 되어야 하는 경우에만 다른 용어를 사용해야 한다.
  - data, info, thing, item, object, entity같은 모호하고 의미 없는 이름은 피해야 한다.
  - 이름을 지을 때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지를 고려해야 한다

## Item 37 공식 명칭에는 상표를 붙이기

- 구조적 타이핑의 특성 때문에 가끔 코드가 이상한 결과를 낼 수 있다.

```tsx
interface Vector2D {
  x: number;
  y: number;
}
function calculateNom(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({ x: 3, y: 4 }); // 정상,결과는 5
const vec3D = { x: 3, y: 4, z: 1 };
calculateNom(vec3D); // 정상! 결과는 동일하게 5
```

- 수학적으로 따지면 2차원 벡터를 사용해야 이치에 맞다. calculateNorm 함수가 3차원 벡터를 허용하지 않게 하려면 공식 명칭을 사용하면 된다. 공식 명칭 개념을 타입스크립트에서 흉내 내려면 ‘상표(brand)’를 붙이면 된다.

```tsx
interface Vector2D {
	_brand: '2d';
	x: number;
	y: number;
}

function vec2D(x: number, y: number): Vector2D {
	return {x, y, _brand: ‘2d’};
}

function calculateNorm(p: Vector2D) {
	return Math.sqrt(p.x * p.x + p.y * p.y); // 기존과 동일합니다.
}

calculateNom(vec2D(3, 4)); // 정상, 5를 반환합니다.
const vec3D = {x: 3, y: 4, z: 1};
calculateNom(vec3D); // '_brand' 속성이 ... 형식에 없습니다.
```

- 타입스크립트는 구조적 타이핑(덕 타이핑)을 사용하기 떄문에, 값을 세밀하게 구분하지 못하는 경우가 있다. 값을 구분하기 위해 공식 명칭이 필요하다면 상표를 붙이는 것을 고려하자.
- 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다.

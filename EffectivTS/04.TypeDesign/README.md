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

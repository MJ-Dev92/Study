# 1장 디자인 패턴과 프로그래밍 패러다임

## SECTION 1 디자인 패턴

<aside>
💡 디자인 패턴이란 프로그램을 설계할 때 발생했던 문제점들을 객체 간의 상호 관계 등을 이용하여 해결할 수 있도록 하나의 ‘규약’ 형태로 만들어 놓은 것을 의미한다.

</aside>

### 1.1 싱글톤 패턴

- 싱글톤 패턴은 하나의 클래스에 오직 하나의 인스턴스만 가지는 패턴이다. 단 하나의 인스턴스를 만들어 이를 기반으로 로직을 만드는 데 쓰이며, 보통 데이터베이스 연결 모듈에 많이 사용한다.
- 하나의 인스턴스를 만들어 놓고 해당 인스턴스를 다른 모듈들이 공유하며 사용하기 때문에 인스턴스를 생성할 때 드는 비용이 줄어드는 장점이 있다.
- 하지만 의존성이 높아진다는 단점이 있다.

### **자바스크립트의 싱글톤 패턴**

- 자바스크립트에서는 리터럴 {} 또는 new Object로 객체를 생성하게 되면 다른 어떤 객체와도 같지 않기 때문에 이 자체만으로도 싱글톤 패턴을 구현할 수 있다.

```tsx
const obj = {
  a: 27,
};
const obj2 = {
  a: 27,
};

console.log(obj === obj2);
// false
```

```tsx
class Singleton {
  constructor() {
    if (!Singleton.instance) {
      Singleton.instance = this;
    }
    return Singleton.instance;
  }
  getInstance() {
    return this;
  }
}
const a = new Singleton();
const b = new Singleton();
console.log(a === b); // true
```

### 데이터베이스 연결 모듈

```tsx
// DB 연결을 하는 것이기 때문에 비용이 더 높은 작업
const URL = "mongodb://localhost:27017/kundolapp";
const createConnection = (url) => ({ url: url });
class DB {
  constructor(url) {
    if (!DB.instance) {
      DB.instance = createConnection(url);
    }
    return DB.instance;
  }
  connect() {
    return this.instance;
  }
}
const a = new DB(URL);
const b = new DB(URL);
console.log(a === b); // true
```

- 하나의 인스턴스를 기반으로 a, b를 생성하는 것을 볼 수 있다. 이를 통해 데이터베이스 연결에 관한 인스턴스 생성 비용을 아낄 수 있다.

### 싱글톤 패턴의 단점

- 싱글톤 패턴은 TDD를 할 때 걸림돌이 된다. TDD를 할 때 단위 테스트를 주로 하는데, 단위 테스트는 테스트가 서로 독립적이어야 하며 테스트를 어떤 순서로든 실행할 수 있어야한다.
- 하지만 싱글톤 패턴은 미리 생성된 하나의 인스턴스를 기반으로 구현하는 패턴이므로 각 테스트마다 ‘독립적인’ 인스턴스를 만들기가 어렵다

### 의존성 주입

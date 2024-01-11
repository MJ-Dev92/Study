# 1장 디자인 패턴과 프로그래밍 패러다임

## SECTION 1 디자인 패턴

<aside>
💡 디자인 패턴이란 프로그램을 설계할 때 발생했던 문제점들을 객체 간의 상호 관계 등을 이용하여 해결할 수 있도록 하나의 ‘규약’ 형태로 만들어 놓은 것을 의미한다.

</aside>

### 1.1 싱글톤 패턴

<aside>
💡 싱글톤 패턴은 하나의 클래스에 오직 하나의 인스턴스만 가지는 패턴이다. 단 하나의 인스턴스를 만들어 이를 기반으로 로직을 만드는 데 쓰이며, 보통 데이터베이스 연결 모듈에 많이 사용한다.

</aside>

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

- 싱글톤 패턴은 사용하기가 쉽고 굉장히 실용적이지만 모듈 간의 결합을 강하게 만들 수 있다는 단점이 있다. 이때 의존성 주입을 통해 모듈간의 결합을 조금 더 느슨하게 만들어 해결할 수 있다.
- 메인 모듈이 ‘직접’ 다른 하위 모듈에 대한 의존성을 주기보다는 중간에 의존성 주입자가 이 부분을 가로채 메인 모듈이 ‘간접’적으로 의존성을 주입하는 방식
- 이를 통해 메인 모듈은 하위 모듈에 대한 의존성이 떨어지게 된다. 이를 ‘디커플링이 된다’라고 한다

### 의존성 주입의 장점

- 모듈들을 쉽게 교체할 수 있는 구조가 되어 테스팅과 마이그레이션하기 좋다
- 구현화할 때 추상화 레이어를 넣고 이를 기반으로 구현체를 넣어 주기 때문에 애플리케이션 의존성 방향이 일관된다. 또한 애플리케이션을 쉽게 추론할 수 있다.
- 모듈 간의 관계들이 조금 더 명확해진다.

### 의존성 주입의 단점

- 모듈들이 더욱더 분리되므로 클래스 수가 늘어나 복잡성이 증가될 수 있으며 약간의 런타임 패널티가 생기기도 한다.

### 의존성 주입 원칙

- 의존성 주입은 ‘상위 모듈은 하위 모듈에서 어떠한 것도 가져오지 않아야 한다. 또한, 둘 다 추상화에 의존해야 하며, 이때 추상화는 세부 사항에 의존하지 말아야 한다’라는 의존성 주입 원칙을 지켜주면서 만들어야 한다.

### 1.2 팩토리 패턴

<aside>
💡 팩토리 패턴은 객체를 사용하는 코드에서 객체 생성 부분을 뗴어내 추상화한 패턴이자 상속 관계에 있는 두 클래스에 상위 클래스가 중요한 뼈대를 결정하고, 하위 클래스에서 객체 생성에 관한 구체적인 내용을 결정하는 패턴이다.

</aside>

- 상위 클래스에서는 인스턴스 생성 방식에 대해 전혀 알 필요가 없기 때문에 더 많은 유연성을 갖게 된다.
- 객체 생성 로직이 따로 떼어져 있기 때문에 코드를 리팩터링하더라도 한 곳만 고칠 수 있게 되니 유지보수성이 증가 된다.

### 자바스크립트 팩토리 패턴

- 자바스크립트에서 팩토리 패턴을 구현한다면 간단하게 new Object()로 구현할 수 있다.

```tsx
class CoffeeFactory {
  static createCoffee(type) {
    const factory = factoryList[type];
    return factory.createCoffee();
  }
}
class Latte {
  constructor() {
    this.name = "latte";
  }
}
class Espresso {
  constructor() {
    this.name = "Espresso";
  }
}

class LatteFactory extends CoffeeFactory {
  static createCoffee() {
    return new Latte();
  }
}
class EspressoFactory extends CoffeeFactory {
  static createCoffee() {
    return new Espresso();
  }
}
const factoryList = { LatteFactory, EspressoFactory };

const main = () => {
  // 라떼 커피를 주문한다.
  const coffee = CoffeeFactory.createCoffee("LatteFactory");
  // 커피 이름을 부른다.
  console.log(coffee.name); // latte
};
main();
```

- LatteFactory에서 생성한 인스턴스를 CoffeeFactory에 주입하고 있기 때문에 의존성 주입이라고 볼 수 있다.
- static 키워드를 통해 createCoffee() 메서드를 정적 메서드로 정의하면 클래스 기반으로 객체를 만들지 않고 호출이 가능하고, 해당 메서드에 대한 메모리 할당을 한 번만 할 수 있는 장점이 있다.

### 1.3 전략 패턴

<aside>
💡 전략 패턴은 정책 패턴이라고도 하며, 객체의 행위를 바꾸고 싶은 경우 ‘직접’ 수정하지 않고 전략이라고 부르는 ‘캡슐화한 알고리즘’을 컨텍스트 안에서 바꿔주면서 상호 교체가 가능하게 만드는 패턴이다.

</aside>

### passport의 전략 패턴

- 전략 패턴을 활용한 라이브러리로는 passport가 있다.
- passport는 Npde.js에서 인증 모듈을 구현할 때 쓰는 미들웨어 라이브러리로, 여러 가지 ’전략’을 기반으로 인증할 수 있게 한다.
- 다음 코드처럼 ‘전략’만 바꿔서 인증하는 것을 볼 수 있다.

```tsx
var passport = require("passport"),
  LocalStrategy = require("passport-local").Strategy;

passport.use(
  new LocalStrategy(function (username, password, done) {
    User.findOne({ username: username }, function (err, user) {
      if (err) {
        return done(err);
      }
      if (!user) {
        return done(null, false, { message: "Incorrect username." });
      }
      if (!user.validPassword(password)) {
        return done(null, false, { message: "Incorrect password." });
      }
      return done(null, user);
    });
  })
);
```

- passport.use(new LocalStrategy(… 처럼 passport.use()라는 메서드에 ‘전략’을 매개변수로 넣어서 로직을 수행하는 것을 볼 수 있다.

### 1.4 옵저버 패턴

<aside>
💡 옵저버 패턴은 주체가 어떤 객체의 상태 변화를 관찰하다가 상태 변화가 있을 때마다 메서드 등을 통해 옵저버 목록에 있는 옵저버들에게 변화를 알려주는 디자인 패턴이다.

</aside>

- 여기서 주체란 객체의 상태 변화를 보고 있는 관찰자이며, 옵저버들이란 이 객체의 상태 변화에 따라 전달되는 메서드 등을 기반으로 ‘추가 변화 사항’이 생기는 객체들을 의미한다.
- 옵저버 패턴을 활용한 서비스로는 트위터가 있다.
- 옵저버 패턴은 주로 이벤트 기반 시스템에 사용하며 MVC(Model-View-Controller)패턴에도 사용된다.

### 자바스크립트에서의 옵저버 패턴

- 자바스크립트에서의 옵저버 패턴은 프록시 객체를 통해 구현할 수 있다.

### **프록시 객체**

- 프록시 객체는 어떠한 대상의 기본적인 동작의 작업을 가로챌 수 있는 객체를 뜻하며, 자바스크립트에서 프록시 객체는 두 개의 매개변수를 가진다.
- target: 프록시할 대상
- handler: target 동작을 가로채고 어떠한 동작을 할 것인지가 설정되어 있는 함수

```jsx
const handler = {
  get: function (target, name) {
    return name === "name" ? `${target.a} ${target.b}` : target[name];
  },
};
const p = new Proxy({ a: "KUNDOL", b: "IS AUMUMU ZANGIN" }, handler);
console.log(p.name); // KUNDOL IS AUMUMU ZANGIN
```

- new Proxy()로 a와 b 속성을 가지고 있는 객체와 handler 함수를 매개변수로 넣고 p라는 변수를 선언 했다.
- p의 name 속성을 참조하니 a와 b라는 속성밖에 없는 객체가 handler의 ‘name이라는 속성에 접근할 때 a와 b를 합쳐서 문자열을 만들라’ 이렇게 name 속성 등 특정 속성에 접근할 때 그 부분을 가로채서 어떠한 로직을 강제할 수 있는 것이 프록시 객체이다.

### 프록시 객체를 이용한 옵저버 패턴

- 자바스크립트의 프록시 객체를 통해 옵저버 패턴

```jsx
function createReactiveObject(target, callback) {
  const proxy = new Proxy(target, {
    set(obj, prop, value) {
      if (value !== obj[prop]) {
        const prev = obj[prop];
        obj[prop] = value;
        callback(`${prop}가 [${prev}] >> [${value}] 로 변경되었습니다`);
      }
      return true;
    },
  });
  return proxy;
}
const a = {
  형규: "솔로",
};
const b = createReactiveObject(a, console.log);
b.형규 = "솔로";
b.형규 = "커플";
// 형규가 [솔로] >> [커플] 로 변경되었습니다
```

- set() 함수를 통해 속성에 대한 접근을 ‘가로채’서 형규라는 속성이 솔로에서 커플로 되는 것을 감시할 수 있다.

### Vue.js 3.0의 옵저버 패턴

<aside>
💡 프런트엔드에서 많이 쓰는 프레임워크 ref나 reactive로 정의하면 해당 값이 변경되었을 때 자동으로 DOM에 있는 값이 변경되는데, 이는 앞서 설명한 프록시 객체를 이용한 옵저버패턴을 이용하여 구현한다.

</aside>

```jsx
function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>,
  proxyMap: WeakMap<Target, any>
) {
  if (!isObject(target)) {
    if (__DEV__) {
      console.warn(`value cannot be made reactive: ${String(target)}`);
    }
    return target;
  }
  // target is already a Proxy, return it.
  // exception: calling readonly() on a reactive object
  if (
    target[ReactiveFlags.RAW] &&
    !(isReadonly && target[ReactiveFlags.IS_REACTIVE])
  ) {
    return target;
  }
  // target already has corresponding Proxy
  const existingProxy = proxyMap.get(target);
  if (existingProxy) {
    return existingProxy;
  }
  // only a whitelist of value types can be observed.
  const targetType = getTargetType(target);
  if (targetType === TargetType.INVALID) {
    return target;
  }
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  );
  proxyMap.set(target, proxy);
  return proxy;
}
```

- proxyMap이라는 프록시 객체를 사용했고, 객체 내부의 get(), set() 메서드를 사용한 것을 볼 수 있다.

### 1.5 프록시 패턴과 프록시 서버

<aside>
💡 앞서 설명한 프록시 객체는 사실 디자인 패턴 중 하나인 프록시 패턴이 녹아들어 있는 객체이다

</aside>

### 프록시 패턴

- 프록시 패턴은 대상 객체에 접근하기 전 그 접근에 대한 흐름을 가로채 해당 접근을 필터링하거나 수정하는 등의 역할을 하는 계층이 있는 디자인 패턴이다.
- 이를 통해 객체의 속성, 변환 등을 보완하며 보안, 데이터 검증, 캐싱, 로깅에 사용된다.

### 프록시 서버

- 프록시 서버는 서버와 클라이언트 사이에서 클라이언트가 자신을 통해 다른 네트워크 서비스에 간접적으로 접속할 수 있게 해주는 컴퓨터 시스템이나 응용프로그램을 가리킨다.

### 프록시 서버로 쓰는 nginx

- nginx는 비동기 이벤트 기반의 구조와 다수의 연결을 효과적으로 처리 가능한 웹 서버이며, 주로 Node.js 서버 앞단의 프록시 서버로 활용된다.
- Node.js 서버를 구축할 때 앞단에 ngnix를 둔다. 이를 통해 익명 사용자가 직접적으로 서버에 접근하는 것을 차단하고, 간접적으로 한 단계를 더 거치게 만들어 보안을 강화할 수 있다.

## **객체와 자료구조(Objects and Data Structures)**

### `getter`와 `setter`를 사용하자

타입스크립트는 `getter`, `setter` 구문을 지원한다. 캡슐화한 객체에서 데이터를 접근할 때 객체에서 속성을 찾는 것보다, `getter`, `setter`를 사용하는 것이 더 낫다. 그 이유는 다음과 같다.

- 객체 속성을 얻는 것 이상으로 무언가를 더 하고 싶을 때, 코드 안에서 관련된 모든 접근자를 찾아 변경하지 않아도 된다.
- `set`을사용할 때 검증 로직을 추가하는 것이  간단하다.
- 내부의 API를 캡슐화할 수 있다.
- 값을 조회하고 설정할 때 로그를 기록하고 에러를 처리하기 쉽다.
- 서버에서 객체 속성을 불러올 때 lazy load 할 수 있다.

> **`getter`, `setter`**
>
> [참고](https://ko.javascript.info/property-accessors#ref-248)

<br/>

**안좋은예 :**

```javascript
type BankAccount = {
  balance: number;
  // ...
}

const value = 100;
const account: BankAccount = {
  balance: 0,
  // ...
};

if (value < 0) {
  throw new Error('Cannot set negative balance.');
}

account.balance = value;
```

**좋은예 :**

```javascript
class BankAccount {
  private accountBalance: number = 0;

  get balance(): number {
    return this.accountBalance;
  }

  set balance(value: number) {
    if (value < 0) {
      throw new Error('Cannot set negative balance.');
    }

    this.accountBalance = value;
  }

  // ...
}

// 이제 `BankAccount`는 검증 로직을 캡슐화합니다.
// 명세가 바뀐다면, 추가적인 검증 규칙을 추가할 필요가 있습니다.
// 그 때, `setter` 구현부만 수정하면 됩니다.
// 관련있는 다른 코드는 변경할 필요가 없습니다.
const account = new BankAccount();
account.balance = 100;
```



<br/>

> **`private`**
>
> 선언한 클래스 내에서만 접근이 가능하다. 
>
> private 키워드가 붙은 프로퍼티는 `_`(언더바)를 붙이는 것이 통상적이다. 
>
> 외부에서 접근을 할 시에는 `get`과 `set`을이용한다.
>
> [참고](https://heecheolman.tistory.com/65#private)



<br/><br/>



### `private`/`protected` 멤버를 갖는 객체를 생성하자

타입스크립트는 클래스 멤버를 위해 `public`, `protected`, `private` 접근자를 지원한다.

> **접근제어자**
>
> - `public` : 디폴트값. 어디에서나 접근 가능.
> - `protected` : 선언한 클래스를 포함해 상속받은 하위클래스에서만 접근 가능.
> - `private` : 선언한 클래스 내에서만 접근 가능.
>
> [protexcted, private 참고](https://heecheolman.tistory.com/65#protected)

<br/>

**안좋은예 :**

```javascript
class Circle {
  radius: number;
  
  constructor(radius: number) {
    this.radius = radius;
  }

  perimeter() {
    return 2 * Math.PI * this.radius;
  }

  surface() {
    return Math.PI * this.radius * this.radius;
  }
}
```

**좋은예 :**

```javascript
class Circle {
  constructor(private readonly radius: number) {
  }

  perimeter() {
    return 2 * Math.PI * this.radius;
  }

  surface() {
    return Math.PI * this.radius * this.radius;
  }
}
```

<br/>

> **Closure(클로저)**
>
> 자바스크립트에서는 접근제한자가 없기 때문에 클로저를 사용하는 것인데, 주로 closure는 변치 않는 데이터를 사용하기 위해서 사용하게 된다.
>
> Redux 등의 상태관리 라이브러리들이 immutable을 기반으로 만들어지는 것인데, 상태관리 라이브러리를 사용하지 않고 immutable 자체로 개발을 할 수 있어야 한다고 한다.



<br/><br/>



### 불변성을 선호하자

타입스크립트의 타입 시스템은 `interface`/`class`의 개별 속성을 **readonly**로 표시할 수 있다. 이를 통해 예상치 않은 변조를 막을 수 있다.

더욱 나은 방법으로는 타입 `T`를 갖고 mapped types를 사용해 각각의 모든 속성을 읽기 전용으로 표시할 수 있는 `readonly` 내장 타입이 존재한다. ([mapped types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types))

<br/>

**안좋은예 :**

```javascript
interface Config {
  host: string;
  port: string;
  db: string;
}
```

**좋은예 :**

```javascript
interface Config {
  readonly host: string;
  readonly port: string;
  readonly db: string;
}
```



<br/>

배열의 경우에는 `ReadonlyArray<T>`를 사용해서 읽기 전용 배열을 생성할 수 있다. 이는 `push()` 와 `fill()`과 같은 변경을 막는다. 하지만 값 자체를 변경하지 않는 `concat()`, `slice()`와 같은 기능은 사용 가능하다.

<br/>

**안좋은예 :**

```javascript
const array: number[] = [ 1, 3, 5 ];
array = []; // 에러
array.push(100); // 배열은 변경될 것이다.
```

**좋은예 :**

```javascript
const array: ReadonlyArray<number> = [ 1, 3, 5 ];
array = []; // 에러
array.push(100); // 에러
```



<br/>

TypeScript 3.4 is a bit easier에서 읽기 전용의 매개변수를 선언할 수 있다.

```javascript
function hoge(args: readonly string[]) {
  args.push(1); // 에러
}
```

> 리터럴 값을 위해 [const assertions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions)를 사용하자.

<br/>

**안좋은예 :**

```javascript
const config = {
  hello: 'world'
};
config.hello = 'world'; // 값이 바뀐다

const array  = [ 1, 3, 5 ];
array[0] = 10; // 값이 바뀐다

// 쓸 수 있는 객체가 반환된다
function readonlyData(value: number) {
  return { value };
}

const result = readonlyData(100);
result.value = 200; // 값이 바뀐다
```

**좋은예 :**

```javascript
// 읽기 전용 객체
const config = {
  hello: 'world'
} as const;
config.hello = 'world'; // 에러

// 읽기 전용 배열
const array  = [ 1, 3, 5 ] as const;
array[0] = 10; // 에러

// 읽기 전용 객체를 반활할 수 있다
function readonlyData(value: number) {
  return { value } as const;
}

const result = readonlyData(100);
result.value = 200; // 에러
```



<br/><br/>



### 타입 vs 인터페이스



합집합 또는 교집합이 필요할 때 타입을 사용하자. `extends` 또는 `implements`가 필요할 때 인터페이스를 사용하자. 엄격한 규칙은 없으므로 각자에게 맞는 방법을 사용하면 된다.

<br/>

**안좋은예 :**

```javascript
interface EmailConfig {
  // ...
}

interface DbConfig {
  // ...
}

interface Config {
  // ...
}

//...

type Shape = {
  // ...
}
```

**좋은예 :**

```javascript
type EmailConfig = {
  // ...
}

type DbConfig = {
  // ...
}

type Config  = EmailConfig | DbConfig;

// ...

interface Shape {
  // ...
}

class Circle implements Shape {
  // ...
}

class Square implements Shape {
  // ...
}

```



<br/><br/>























































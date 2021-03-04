## 함수(Function)

### 함수의 매개변수는 2개 혹은 그 이하가 이상적이다

함수의 매개변수가 3개 이상인 경우, 각기 다른 인수로 여러 다른 케이스를 테스트해야 하기 때문에 경우의 수가 매우 많아진다. 

따라서 한 개 혹은 두 개의 매개변수가 이상적인 경우이며, 세 개는 피해야 한다. 그 이상의 경우에는 합쳐야 한다. 

많은 매개변수를 사용해야 한다면 객체 리터럴(`{}`)을 사용하자.



함수가 기대하는 속성을 명확하게 하기 위해 **구조 분해(destructuring)** 구문을 사용할 수 있으며, 이 구문에는 몇 가지 장점이 있다.

- 함수 시그니쳐(매개변수의 타입, 반환값의 타입 등)를 볼 때, 어떤 속성이 사용되는지 즉시 알 수 있다.

- 명명된 매개변수처럼 보이게 할 수 있다.

- 구조 분해는 함수로 전달된 매개변수 객체의 특정한 원시 값을 복제하며, 이는 사이트 이펙트를 방지하는데 도움을 준다. (매개변수 객체로부터 구조 분해된 객체와 배열은 복제되지 않는다.)

- 타입스크립트는 사용하지 않은 속성에 대해 경고를 주지만, 구조 분해를 사용하면 경고를 받지 않을 수 있다.

  

<br/>



**안좋은 예 :**

```javascript
function createMenu(title: string, body: string, buttonText: string, cancellable: boolean) {
  // ...
}

createMenu('Foo', 'Bar', 'Baz', true);
```



**좋은 예 :**

```javascript
function createMenu(options: { title: string, body: string, buttonText: string, cancellable: boolean }) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
});
```



[타입 앨리어스](https://sporto.gitbooks.io/elm-tutorial/content/ko/01-foundations/06-type-aliases.html)를 사용해서 가독성을 더 높일 수 있다.

```javascript
type MenuOptions = { title: string, body: string, buttonText: string, cancellable: boolean };

function createMenu(options: MenuOptions) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
});
```



<br/>

<br/>



### 함수는 하나의 행동만 해야한다.

함수가 한 가지 이상의 역할을 수행하게 되면 작성하고 테스트하고 추론하기 어려워진다. 함수가 하나의 행동을 정의할 수 있을 때, 쉽게 리팩토링할 수 있으며 코드를 더욱 명료하게 읽을 수 있다. 



<br/>



**안좋은 예 :**

```javascript
function emailClients(clients: Client[]) {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```



**좋은 예 :**

```javascript
function emailClients(clients: Client[]) {
  clients.filter(isActiveClient).forEach(email);
}

function isActiveClient(client: Client) {
  const clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```



> **clients: Client[]**
>
> 타입이 배열인 경우 간단하게 아래와 같이 선언한다.
>
> ```ts
> let arr: number[] = [1,2,3];
> ```
>
> [참고](https://joshua1988.github.io/ts/guide/basic-types.html#array)



<br/>

<br/>



### 함수명은 함수가 무엇을 하는지 알 수 있어야 한다

**안좋은 예 :**

```javascript
function addToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();

// 무엇이 추가되는지 함수 이름만으로 유추하기 어렵습니다
addToDate(date, 1);
```



**좋은 예 :**

```javascript
function addMonthToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();
addMonthToDate(date, 1);
```



<br/>

<br/>



### 함수는 단일 행동을 추상화 해야한다

추상화된 이름이 여러 의미를 내포하고 있다면 해당 함수는 너무 많은 일을 하게끔 설계된 것이다. 재사용성과 쉬운 테스트를 위해서 함수를 쪼개자.

<br/>

**안좋은 예 :**

```javasc
function parseCode(code: string) {
  const REGEXES = [ /* ... */ ];
  const statements = code.split(' ');
  const tokens = [];

  REGEXES.forEach((regex) => {
    statements.forEach((statement) => {
      // ...
    });
  });

  const ast = [];
  tokens.forEach((token) => {
    // lex...
  });

  ast.forEach((node) => {
    // parse...
  });
}
```



**좋은 예 :**

```javasc
const REGEXES = [ /* ... */ ];

function parseCode(code: string) {
  const tokens = tokenize(code);
  const syntaxTree = parse(tokens);

  syntaxTree.forEach((node) => {
    // parse...
  });
}

function tokenize(code: string): Token[] {
  const statements = code.split(' ');
  const tokens: Token[] = [];

  REGEXES.forEach((regex) => {
    statements.forEach((statement) => {
      tokens.push( /* ... */ );
    });
  });

  return tokens;
}

function parse(tokens: Token[]): SyntaxTree {
  const syntaxTree: SyntaxTree[] = [];
  tokens.forEach((token) => {
    syntaxTree.push( /* ... */ );
  });

  return syntaxTree;
}
```





<br/>

<br/>



### 중복된 코드를 제거하자

중복된 코드는 로직을 변경할 때 한 곳 이상을 변경해야 하기 때문에 좋지 않다.

중복된 코드를 제거한다는 것은 하나의 함수/모듈/클래스를 사용해 여러 가지 사소한 차이점을 처리할 수 있는 추상화를 만드는 것을 의미한다.

<br/>



**안좋은 예 :**

```javascript
function showDeveloperList(developers: Developer[]) {
  developers.forEach((developer) => {
    const expectedSalary = developer.calculateExpectedSalary();
    const experience = developer.getExperience();
    const githubLink = developer.getGithubLink();

    const data = {
      expectedSalary,
      experience,
      githubLink
    };

    render(data);
  });
}

function showManagerList(managers: Manager[]) {
  managers.forEach((manager) => {
    const expectedSalary = manager.calculateExpectedSalary();
    const experience = manager.getExperience();
    const portfolio = manager.getMBAProjects();

    const data = {
      expectedSalary,
      experience,
      portfolio
    };

    render(data);
  });
}
```



**좋은 예 :**

```javascript
class Developer {
  // ...
  getExtraDetails() {
    return {
      githubLink: this.githubLink,
    }
  }
}

class Manager {
  // ...
  getExtraDetails() {
    return {
      portfolio: this.portfolio,
    }
  }
}

function showEmployeeList(employee: Developer | Manager) {
  employee.forEach((employee) => {
    const expectedSalary = employee.calculateExpectedSalary();
    const experience = employee.getExperience();
    const extra = employee.getExtraDetails();

    const data = {
      expectedSalary,
      experience,
      extra,
    };

    render(data);
  });
}
```



> **Union Type**
>
> 자바스크립트의 OR 연산자(`||`)와 같은 의미로, A이거나 B이다 라는 의미의 타입이다.
>
> ```javascript
> function logText(text: string | number) {
>   // ...
> }
> ```
>
> 위 예시 코드의 파라미터 `text`에는 문자열 타입이나 숫자 타입 모두 올 수 있다. 



<br/>

<br/>



###  `Object.assign` 또는 `구조 분해`를 사용해서 기본 객체를 만들자

> **`Object.assign(target, ...sources)`**
>
> - `target` : 대상 객체
> - `sources` : 하나 이상의 출처 객체
>
> 동일한 키가 존재할 경우 대상 객체의 속성은 출처 객체의 속성으로 덮어쓰여진다.
>
> [참고](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)







<br/>



**안좋은 예 :**

```javascript
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu(config: MenuConfig) {
  config.title = config.title || 'Foo';
  config.body = config.body || 'Bar';
  config.buttonText = config.buttonText || 'Baz';
  config.cancellable = config.cancellable !== undefined ? config.cancellable : true;

  // ...
}

createMenu({ body: 'Bar' });
```



**좋은 예 : **

```javascript
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu(config: MenuConfig) {
  const menuConfig = Object.assign({
    title: 'Foo',
    body: 'Bar',
    buttonText: 'Baz',
    cancellable: true
  }, config);

  // ...
}

createMenu({ body: 'Bar' });
```





대안으로, 기본 값을 **구조 분해**를 사용해 해결할 수도 있다.

```javascript
type MenuConfig = { title?: string, body?: string, buttonText?: string, cancellable?: boolean };

function createMenu({ title = 'Foo', body = 'Bar', buttonText = 'Baz', cancellable = true }: MenuConfig) {
  // ...
}

createMenu({ body: 'Bar' });
```



<br/>



### 함수 매개변수로 플래그를 사용하지 말자

플래그를 사용하는 것 자체가 해당 함수가 한 가지 이상의 일을 한다는 것을 뜻한다. 함수는 한 가지 일만 해야한다. Boolean 변수로 인해 다른 코드가 실행된다면 해당 함수를 쪼개도록 하자.

> **플래그**
>
> 상태를 기록하고 처리의 흐름을 제어하기 위한 **boolean형의 변수**를 의미한다.

<br/>



**안좋은 예 :**

```javascript
function createFile(name: string, temp: boolean) {
  if (temp) {
    fs.create(`./temp/${name}`);
  } else {
    fs.create(name);
  }
}
```



**좋은 예 :**

```javascript
function createTempFile(name: string) {
  createFile(`./temp/${name}`);
}

function createFile(name: string) {
  fs.create(name);
}
```



<br/>

<br/>



### 사이드 이펙트를 피하자(part 1)

어떠한 구조체도 없이 객체 사이의 상태를 공유하거나, 어떤 것에 의해서든지 변경될 수 있는 데이터 타입을 사용하거나, 같은 사이드 이펙트를 만들어내는 것을 여러 개 만들거나 하면 안된다.

<br/>

**안좋은 예 :**

```javascript
// 아래의 함수에서 참조하는 전역 변수입니다.
let name = 'Robert C. Martin';

function toBase64() {
  name = btoa(name);
}

toBase64();
// 이 이름을 사용하는 다른 함수가 있다면, 그것은 Base64 값을 반환할 것입니다

console.log(name); // 'Robert C. Martin'이 출력되는 것을 예상했지만 'Um9iZXJ0IEMuIE1hcnRpbg=='가 출력됨
```



**좋은 예 :**

```javascript
const name = 'Robert C. Martin';

function toBase64(text: string): string {
  return btoa(text);
}

const encodedName = toBase64(name);  
console.log(encodeName)  // 'Um9iZXJ0IEMuIE1hcnRpbg=='
console.log(name);  // 'Robert C. Martin'
```



> **`window.btoa(str)`**
>
> Base-64로 문자열을 인코딩한다.
>
> [참고](https://www.w3schools.com/jsref/met_win_btoa.asp)



<br/>

<br/>



### 사이트 이펙트를 피하자(part 2)

자바스크립트에서 기본타입 자료형은 값을 전달하고 객체와 배열은 참조를 전달한다. 

예를 들어 객체와 배열의 경우 어떤 함수가 쇼핑 장바구니 배열을 변경하는 기능을 가지고 있다면, 구매하려는 아이템이 추가됨으로써 `cart` 배열을 사용하는 다른 함수는 이 추가의 영향을 받을 수 있게 된다. 

이 경우 가장 좋은 방법은 `addItemToCart` 함수에서 `cart` 배열을 복제하여 수정하고 복제본을 반환하는 것이다. 이렇게 하면 `cart` 배열을 참조하는 다른 함수가 변경 사항의 영향을 받지 않게 된다. 

<br/>

**안좋은 예 :**

```javascript
function addItemToCart(cart: CartItem[], item: Item): void {
  cart.push({ item, date: Date.now() });
};
```



**좋은 예 :**

```javascript
function addItemToCart(cart: CartItem[], item: Item): CartItem[] {
  return [...cart, { item, date: Date.now() }];
};
```



<br/>

<br/>



### 전역 함수를 사용하지 말자

전역 환경을 사용하게 되면 다른 라이브러리와 충돌이 날 수 있다. 

> > 잘 이해 안감. 다시 공부 할 것.



<br/>

<br/>



### 명령형 프로그래밍보다 함수형 프로그래밍을 지향하자

함수형 언어는 더 깔끔하고 테스트하기 쉽다. 자바스크립트는 함수형 프로그래밍 언어는 아니지만 함수형 프로그래밍처럼 작성할 수 있다.

> **명령형 프로그래밍**
>
> 선언형 프로그래밍과 반대되는 개념으로, 원하는 결과를 달성해 나가는 과정에만 관심을 두는 프로그래밍 스타일이다. 
>
> **명령형은 HOW(어떻게 할 것인지 설명), 선언형은 WHAT(무엇을 할 것인지 정의)**
>
>  
>
> **함수형 프로그래밍**
>
> 함수형 프로그래밍은 선언적 프로그래밍(declarative programming)이라는 더 넓은 프로그래밍 패러다임의 한 가지이다.

<br/>

**안좋은 예 :**

```javascript
const contributions = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];

let totalOutput = 0;

for (let i = 0; i < contributions.length; i++) {
  totalOutput += contributions[i].linesOfCode;
}
```



**좋은 예 :**

```javascript
const contributions = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];

const totalOutput = contributions
  .reduce((totalLines, output) => totalLines + output.linesOfCode, 0);
```



<br/>

<br/>



### 조건문을 캡슐화하자

**안좋은 예 :**

```javascript
if (subscription.isTrial || account.balance > 0) {
  // ...
}
```



**좋은 예 :**

```javascript
function canActivateService(subscription: Subscription, account: Account) {
  return subscription.isTrial || account.balance > 0;
}

if (canActivateService(subscription, account)) {
  // ...
}
```





<br/>

<br/>



### 부정문을 사용하지 말자

**안좋은 예 :**

```javascript
function isEmailNotUsed(email: string): boolean {
  // ...
}

if (isEmailNotUsed(email)) {
  // ...
}
```



**좋은 예 :**

```javascript
function isEmailUsed(email): boolean {
  // ...
}

if (!isEmailUsed(node)) {
  // ...
}
```





<br/>

<br/>



### 조건문 작성을 피하자

조건문(if문) 대신 **다형성**을 사용해 해결할 수 있다.  

함수는 단 하나의 일만 수행해야 한다. `if`문이 있는 클래스와 함수가 있다면, 해당 함수는 한 가지 이상의 일을 하고 있다는 의미가 되기 때문에 조건문 작성을 피하자는 것이다.





<br/>

**안좋은 예 :**

```javascript
class Airplane {
  private type: string;
  // ...

  getCruisingAltitude() {
    switch (this.type) {
      case '777':
        return this.getMaxAltitude() - this.getPassengerCount();
      case 'Air Force One':
        return this.getMaxAltitude();
      case 'Cessna':
        return this.getMaxAltitude() - this.getFuelExpenditure();
      default:
        throw new Error('Unknown airplane type.');
    }
  }

  private getMaxAltitude(): number {
    // ...
  }
}
```



**좋은 예 :**

```javascript
abstract class Airplane {
  protected getMaxAltitude(): number {
    // shared logic with subclasses ...
  }

  // ...
}

class Boeing777 extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getPassengerCount();
  }
}

class AirForceOne extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude();
  }
}

class Cessna extends Airplane {
  // ...
  getCruisingAltitude() {
    return this.getMaxAltitude() - this.getFuelExpenditure();
  }
}
```





<br/>

<br/>



### 타입 체킹을 피하자

타입스크립트는 자바스크립트의 엄격한 구문적 상위 집합으로, 언어에 선택적인 정적 타입 검사 기능을 추가한다. 타입스크립트의 기능을 최대한 활용하기 위해 항상 변수의 타입, 매개변수, 반환값의 타입을 지정하도록 하자. 리팩토링이 매우 쉬워질 것이다.

<br/>

**안좋은 예 :**

```javascript
function travelToTexas(vehicle: Bicycle | Car) {
  if (vehicle instanceof Bicycle) {
    vehicle.pedal(currentLocation, new Location('texas'));
  } else if (vehicle instanceof Car) {
    vehicle.drive(currentLocation, new Location('texas'));
  }
}
```



**좋은 예 :**

```javascript
type Vehicle = Bicycle | Car;

function travelToTexas(vehicle: Vehicle) {
  vehicle.move(currentLocation, new Location('texas'));
}
```



> **`obj instanceof Class`**
>
> `obj`가 `Class`에 속하거나 `Class`를 상속받는 클래스에 속하면 `true`를 반환한다. 



<br/>

<br/>



### 과도한 최적화를 지양하자

최신 브라우저들은 런타임에서 많은 최적화 작업을 수행한다. 그렇기 때문에 최적화에 많은 시간을 할애한다면 시간낭비일 가능성이 많다. 

최적화가 부족한 부분을 확인할 수 있는 [자료](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers)를 참고해서 최적화가 부족한 부분만 최적화를 하는 것이 좋다. 

<br/>

**안좋은 예 :**

```javascript
// 예전 브라우저에서는 캐시되지 않은 `list.length`를 사용한 각 순회는 비용이 많이 들 것입니다.
// `list.length`의 재계산 때문입니다. 현대 브라우저에서는 이 부분이 최적화됩니다.
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```



**좋은 예 :**

```javascript
for (let i = 0; i < list.length; i++) {
  // ...
}
```







<br/>

<br/>



### 필요하지 않은 코드는 제거하자

사용하지 않는 코드는 중복된 코드만큼 나쁘다. 그렇기 때문에 필요하지 않는 코드는 제거해야 한다. 지운 코드를 다시 확인해야 한다면 버전 기록에서 확인할 수 있다.

<br/>

**안좋은 예 :**

```javascript
function oldRequestModule(url: string) {
  // ...
}

function requestModule(url: string) {
  // ...
}

const req = requestModule;
inventoryTracker('apples', req, 'www.inventory-awesome.io');
```



**좋은 예 :**

```javascript
function requestModule(url: string) {
  // ...
}

const req = requestModule;
inventoryTracker('apples', req, 'www.inventory-awesome.io');
```





<br/>

<br/>



### `iterator`와 `generator`를 사용하자

> **`iterator`**  : 반복기
>
> **`generator`** : 생성기
>
> 반복기와 생성기는 반복 개념을 핵심 언어 내로 바로 가져와 `for...of` 루프의 동작을 사용자 정의하는 메커니즘을 제공한다.
>
> [참고](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Iterators_and_Generators)



스트림과 같이 사용되는 데이터 콜렉션을 사용할 때는 `generator`와 `iterable`을 사용하자. 몇 가지 좋은 이유가 있다.

- 피로출자가 접근할 아이템 수를 결정한다는 의미에서 피호출자를 `generator` 구현으로부터 분리할 수 있다.
- 지연 실행, 아이템은 요구에 의해 스트림 처리될 수 있다.
- `for-of` 구문을 사용해 아이템을 순회하는 내장 지원이 있다.
- `iterable`은 최적화된 `iterator` 패턴을 구현할 수 있다.

> **스트림(Stream)**
>
> 장치(Device - 하드웨어 장치)로부터 데이터를 읽거나 기록할 때 사용하는 중간 매개체 역할을 하는 것.





<br/>

**안좋은 예 :**

```javascript
function fibonacci(n: number): number[] {
  if (n === 1) return [0];
  if (n === 2) return [0, 1];

  const items: number[] = [0, 1];
  while (items.length < n) {
    items.push(items[items.length - 2] + items[items.length - 1]);
  }

  return items;
}

function print(n: number) {
  fibonacci(n).forEach(fib => console.log(fib));
}

// 피보나치 숫자의 첫 번째 10개 숫자를 출력합니다.
print(10);
```



**좋은 예 :**

```javascript
// 피보나치 숫자의 무한 스트림을 생성합니다.
// `generator`는 모든 숫자의 배열을 유지하고 있지 않습니다.
function* fibonacci(): IterableIterator<number> {
  let [a, b] = [0, 1];

  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

function print(n: number) {
  let i = 0;
  for (const fib of fibonacci()) {
    if (i++ === n) break;  
    console.log(fib);
  }  
}

// 피보나치 숫자의 첫 번째 10개 숫자를 출력합니다.
print(10);
```



> **`IterableIterator`**
>
> [Iterator, Generator 참고](https://infoscis.github.io/2017/06/19/TypeScript-handbook-iterators-and-generators/)
>
> [iterable 참고](https://ko.javascript.info/iterable)
>
> [참고](https://mommoo.tistory.com/78)
>
> ====> 추가 공부 필요!!


















































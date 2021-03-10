## SOLID



### 단일 책임 원칙(SRP:Single Responsibility Principle)

클래스를 변경할 때는 단 한 가지 이유만 존재해야 한다. 2개 이상일 경우 클래스가 개념적으로 응집되어 있지 않다는 것이 된다. 클래스를 수정하는데 들이는 시간을 줄이는 것은 중요하다. 만약 한 클래스에 너무 많은 기능이 있고 그 안에서 하나의 기능을 수정할 경우, 다른 종속된 모듈에 어떤 영향을 미치는지 이해하기 어려울 것이다.

<br/>

**안좋은 예:**

```typescript
class UserSettings {
  constructor(private readonly user: User) {
  }

  changeSettings(settings: UserSettings) {
    if (this.verifyCredentials()) {
      // ...
    }
  }

  verifyCredentials() {
    // ...
  }
}
```



**좋은 예:**

```typescript
class UserAuth {
  constructor(private readonly user: User) {
  }

  verifyCredentials() {
    // ...
  }
}


class UserSettings {
  private readonly auth: UserAuth;

  constructor(private readonly user: User) {
    this.auth = new UserAuth(user);
  }

  changeSettings(settings: UserSettings) {
    if (this.auth.verifyCredentials()) {
      // ...
    }
  }
}
```



<br/><br/>



### 개방 폐쇄 원칙(OCP: Open Closed Principle)

> 기존의 코드를 변경하지 않으면서 기능을 추가할 수 있도록 설계 되어야 한다.

> "소프트웨어 개체(클래스, 모듈, 함수 등)는 확장을 위해 개방적이어야 하며 수정시엔 폐쇄적이어야 한다." -Bertrand Meyer-



<br/>

**안좋은 예:**

```typescript
class AjaxAdapter extends Adapter {
  constructor() {
    super();
  }

  // ...
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
  }

  // ...
}

class HttpRequester {
  constructor(private readonly adapter: Adapter) {
  }

  async fetch<T>(url: string): Promise<T> {
    if (this.adapter instanceof AjaxAdapter) {
      const response = await makeAjaxCall<T>(url);
      // response 값을 변경하고 반환한다.
    } else if (this.adapter instanceof NodeAdapter) {
      const response = await makeHttpCall<T>(url);
      // response 값을 변경하고 반환한다.
    }
  }
}

function makeAjaxCall<T>(url: string): Promise<T> {
  // 서버에 요청하고 프로미스를 반환한다.
}

function makeHttpCall<T>(url: string): Promise<T> {
  // 서버에 요청하고 프로미스를 반환한다.
}
```



**좋은 예:**

```typescript
abstract class Adapter {
  abstract async request<T>(url: string): Promise<T>;

  // 하위 클래스와 공유하는 코드 ...
}

class AjaxAdapter extends Adapter {
  constructor() {
    super();
  }

  async request<T>(url: string): Promise<T>{
    // 서버에 요청하고 프로미스를 반환한다.
  }

  // ...
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
  }

  async request<T>(url: string): Promise<T>{
    // 서버에 요청하고 프로미스를 반환한다.
  }

  // ...
}

class HttpRequester {
  constructor(private readonly adapter: Adapter) {
  }

  async fetch<T>(url: string): Promise<T> {
    const response = await this.adapter.request<T>(url);
    // response 값을 변경하고 반환한다.
  }
}
```



<br/><br/>







### 리스코프 치환 원칙(LSP: Liskov Substitution Principle)

리스코프 원칙이란 S가 T의 하위 타입일 경우, 프로그램이 갖춰야 할 속성들(정확성, 수행되는 작업 등)의 변경사항 없이, T 타입의 객체를 S 타입의 객체로 교체(치환)할 수 있어야 한다는 원칙이다.  

예를 들면, 부모 클래스와 자식 클래스가 있을 때 베이스 클래스(부모)와 하위 클래스(자식)는 잘못된 결과 없이 서로 교환해 사용할 수 있다. 

조금 더 쉬운 예를 들면, 정사각형-직사각형을 생각하면 된다. 수학적으로 정사각형은 직사각형이다. 하지만 상속을 통해 "is-a" 관계로 설계한다면 문제가 발생할 수 있다.

**안좋은 예:**

```typescript
class Rectangle {
  constructor(
    protected width: number = 0,
    protected height: number = 0) {

  }

  setColor(color: string): this {
    // ...
  }

  render(area: number) {
    // ...
  }

  setWidth(width: number): this {
    this.width = width;
    return this;
  }

  setHeight(height: number): this {
    this.height = height;
    return this;
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width: number): this {
    this.width = width;
    this.height = width;
    return this;
  }

  setHeight(height: number): this {
    this.width = height;
    this.height = height;
    return this;
  }
}

function renderLargeRectangles(rectangles: Rectangle[]) {
  rectangles.forEach((rectangle) => {
    const area = rectangle
      .setWidth(4)
      .setHeight(5)
      .getArea(); // BAD: `Square` 클래스에서는 25를 반환한다. 하지만 20이 반환되어야 옳다.
    rectangle.render(area);
  });
}

const rectangles = [new Rectangle(), new Rectangle(), new Square()];
renderLargeRectangles(rectangles);
```



**좋은 예:**

```typescript
abstract class Shape {
  setColor(color: string): this {
    // ...
  }

  render(area: number) {
    // ...
  }

  abstract getArea(): number;
}

class Rectangle extends Shape {
  constructor(
    private readonly width = 0,
    private readonly height = 0) {
    super();
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(private readonly length: number) {
    super();
  }

  getArea(): number {
    return this.length * this.length;
  }
}

function renderLargeShapes(shapes: Shape[]) {
  shapes.forEach((shape) => {
    const area = shape.getArea();
    shape.render(area);
  });
}

const shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)];
renderLargeShapes(shapes);
```



<br/><br/>





### 인터페이스 분리 원칙(ISP: Interfave Segregation Principle)

인터페이스 분리 원칙이란 클라이언트는 사용하지 않는 인터페이스에 의존하지 않는다는 것이다. 인터페이스 분리 원칙은 단일 책임 원칙과 많은 관련이 있다. 이 말은 클라이언트가 노출된 메소드를 사용하는 대신, 전체 파이를 얻지 않는 방식으로 추상화를 설계해야 한다는 것이다. 여기에는 클라이언트에게 클라이언트가 실제로 필요하지 않은 메소드르 구현을 강요하는 것도 포함한다.

**안좋은 예:**

```typescript
interface SmartPrinter {
  print();
  fax();
  scan();
}

class AllInOnePrinter implements SmartPrinter {
  print() {
    // ...
  }  
  
  fax() {
    // ...
  }

  scan() {
    // ...
  }
}

class EconomicPrinter implements SmartPrinter {
  print() {
    // ...
  }  
  
  fax() {
    throw new Error('Fax not supported.');
  }

  scan() {
    throw new Error('Scan not supported.');
  }
}
```



**좋은 예:**

```typescript
interface Printer {
  print();
}

interface Fax {
  fax();
}

interface Scanner {
  scan();
}

class AllInOnePrinter implements Printer, Fax, Scanner {
  print() {
    // ...
  }  
  
  fax() {
    // ...
  }

  scan() {
    // ...
  }
}

class EconomicPrinter implements Printer {
  print() {
    // ...
  }
}
```



<br/>



> **extendsd와 implements의 차이**
>
> extends와 implements 모두 **상속**의 형태이다.
>
> extends는 부모에서 선언된 메소드/변수를 자식에서 모두 상속 받아 그대로 사용할 수 있다.
>
> 반면에, implements는 부모의 메소드/변수를 그대로 가져다 사용하는 것이 아니라 반드시 **오버라이드** 해서 사용해야 한다. 또한, implements는 **다중 상속**도 지원한다.
>
> 
>
> [참고](https://juunone.netlify.app/typescript/extends/)

<br/><br/>







### 의존성 역전 원칙(DIP, Dependency Inversion Principle)

의존성 역전 웍칙은 두 사지 필수적인 사항을 가지고 있다.

1. 상위 레벨의 모듈은 하위 레벨의 모듈에 의존하지 않아야 한다. 두 모듈은 모두 추상화에 의존해야 한다.
2. 추상화는 세부사항에 의존하지 않아야 한다. 세부사항은 추상화에 의존해야 한다.

Angular의 의존성 주입(DI) 형태 안에서 이 원칙을 구혀냏 볼 수 있다. 동일한 개념은 아니지만 DIP는 상위 레벨의 모듈이 하위 레벨 모듈의 세부사항에 접근하고 설정하지 못하도록 지킨다. 이것의 장접은 모듈 간의 의존성(결합도)을 감소시키는 데 있다. 모듈 간의 의존성이 높을 수록 코드를 리팩토링하기 어려워 지기 때문에 의존성(결합도)이 높지 않은 코드를 짜야 한다.

DIP는 주로 IoC 컨테이너를 사용함으로써 달성된다. (타입스크립트를 위한 강력한 IoC 컨테이너의 예제는 [InversifyJs](https://www.npmjs.com/package/inversify)이다.)

> **추상 클래스**
>
> <타입스크립트 퀵스타트>  p.202 (7.1.5. 추상 클래스를 이용한 공통 기능 정의) 참고

**안좋은 예:**

```typescript
import { readFile as readFileCb } from 'fs';
import { promisify } from 'util';

const readFile = promisify(readFileCb);

type ReportData = {
  // ..
}

class XmlFormatter {
  parse<T>(content: string): T {
    // XML 문자열을 T 객체로 변환
  }
}

class ReportReader {

  // BAD: 특정 요청의 구현에 의존하는 것을 만들었다.
  // `parse` 메소드에 의존하는 `ReportReader`를 만들어야 한다.
  private readonly formatter = new XmlFormatter();

  async read(path: string): Promise<ReportData> {
    const text = await readFile(path, 'UTF8');
    return this.formatter.parse<ReportData>(text);
  }
}

// ...
const reader = new ReportReader();
await report = await reader.read('report.xml');
```



**좋은 예:**

```typescript
import { readFile as readFileCb } from 'fs';
import { promisify } from 'util';

const readFile = promisify(readFileCb);

type ReportData = {
  // ..
}

interface Formatter {
  parse<T>(content: string): T;
}

class XmlFormatter implements Formatter {
  parse<T>(content: string): T {
    // XML 문자열을 T 객체로 변환
  }
}


class JsonFormatter implements Formatter {
  parse<T>(content: string): T {
    // JSON 문자열을 T 객체로 변환
  }
}

class ReportReader {
  constructor(private readonly formatter: Formatter) {
  }

  async read(path: string): Promise<ReportData> {
    const text = await readFile(path, 'UTF8');
    return this.formatter.parse<ReportData>(text);
  }
}

// ...
const reader = new ReportReader(new XmlFormatter());
await report = await reader.read('report.xml');

// 또는 json 보고서가 필요한 경우
const reader = new ReportReader(new JsonFormatter());
await report = await reader.read('report.json');
```



<br/><br/>






























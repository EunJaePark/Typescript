## 클래스(Class)



### 클래스는 작아야 한다

클래스의 크기는 책임에 의해 측정된다. **단일 책임 원칙**에 따르면 클래스는 작아야 한다.

<br/>

**안좋은 예:**

```typescript
class Dashboard {
  getLanguage(): string { /* ... */ }
  setLanguage(language: string): void { /* ... */ }
  showProgress(): void { /* ... */ }
  hideProgress(): void { /* ... */ }
  isDirty(): boolean { /* ... */ }
  disable(): void { /* ... */ }
  enable(): void { /* ... */ }
  addSubscription(subscription: Subscription): void { /* ... */ }
  removeSubscription(subscription: Subscription): void { /* ... */ }
  addUser(user: User): void { /* ... */ }
  removeUser(user: User): void { /* ... */ }
  goToHomePage(): void { /* ... */ }
  updateProfile(details: UserDetails): void { /* ... */ }
  getVersion(): string { /* ... */ }
  // ...
}
```



**좋은 예:**

```typescript
class Dashboard {
  disable(): void { /* ... */ }
  enable(): void { /* ... */ }
  getVersion(): string { /* ... */ }
}

// 다른 클래스에 남은 메소드를 이동시킴으로써 책임을 분산시키자
// ...
```



<br/><br/>



### 높은 응집도과 낮은 결합도

응집도는 클래스 멤버가 서로에게 연관되어 있는 정도를 정의한다. 이상적으로, 클래스 안의 모든 필드는 각 메소드에 의해 사용되어야 한다. 

결합도는 두 클래스가 서로에게 관련되어 있거나 종속되어 있는 정도를 정의한다. 한 클래스의 변경이 다른 클래스에 영향을 주지 않는다면 클래스의 결합도가 낮다고 할 수 있다.

좋은 소프트웨어 설계는 **높은 응집도**와 **낮은 결합도**를 가진다.

<br/>

**안좋은 예:**

```typescript
class UserManager {
  // Bad: 각 private 변수는 메소드의 하나 혹은 또 다른 그룹에 의해 사용된다.
  // 클래스가 단일 책임 이상의 책임을 가지고 있다는 명백한 증거이다.
  // 사용자의 트랜잭션을 얻기 위해 서비스를 생성하기만 하면 되는 경우,
  // 여전히 `emailSender` 인스턴스를 전달해야 한다.
  constructor(
    private readonly db: Database,
    private readonly emailSender: EmailSender) {
  }

  async getUser(id: number): Promise<User> {
    return await db.users.findOne({ id });
  }

  async getTransactions(userId: number): Promise<Transaction[]> {
    return await db.transactions.find({ userId });
  }

  async sendGreeting(): Promise<void> {
    await emailSender.send('Welcome!');
  }

  async sendNotification(text: string): Promise<void> {
    await emailSender.send(text);
  }

  async sendNewsletter(): Promise<void> {
    // ...
  }
}
```



**좋은 예:**

```typescript
class UserService {
  constructor(private readonly db: Database) {
  }

  async getUser(id: number): Promise<User> {
    return await this.db.users.findOne({ id });
  }

  async getTransactions(userId: number): Promise<Transaction[]> {
    return await this.db.transactions.find({ userId });
  }
}

class UserNotifier {
  constructor(private readonly emailSender: EmailSender) {
  }

  async sendGreeting(): Promise<void> {
    await this.emailSender.send('Welcome!');
  }

  async sendNotification(text: string): Promise<void> {
    await this.emailSender.send(text);
  }

  async sendNewsletter(): Promise<void> {
    // ...
  }
}
```



<br/><br/>







### 상속(inheritance)보다 조합(composition)을 사용하자

(<타입스크립트 퀵스타트> p.187(7.1.3. 상속 관계와 포함 관계) 참고)

객체지향 프로그래밍에서 클래스 간의 관계는 크게 **상속 관계**와 **포함 관계** 두 가지로 나눌 수 있다.

Gang of four의 [디자인 패턴](https://en.wikipedia.org/wiki/Design_Patterns)에 나와있듯이 가능하다면 상속보다 조합을 사용하는 것이 좋다. 조합보다 상속이 더 좋은 경우는 아래와 같다.

- 'has-a' 관계가 아닌 'is-a' 관계일 때.
- 기반이 되는 클래스의 코드를 재사용할 수 있을 때.
- 기반이 되는 클래스를 수정하여 파생된 클래스는 전체적으로 수정하고 싶을 때.

> **extends**
>
> `extends`는 클래스를 다른 클래스의 자식으로 만들기 위해 [class 선언](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/class) 또는 [class 식](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/class)에 사용된다. 

**안좋은 예:**

```typescript
class Employee {
  constructor(
    private readonly name: string,
    private readonly email: string) {
  }

  // ...
}

// `Employee`가 `EmployeeTaxData`를 가지기 때문에 나쁜 예이다. `EmployeeTaxData`는 `Employee`의 타입이 아니다.
class EmployeeTaxData extends Employee {
  constructor(
    name: string,
    email: string,
    private readonly ssn: string,
    private readonly salary: number) {
    super(name, email);
  }

  // ...
}
```



**좋은 예:**

```typescript
class Employee {
  private taxData: EmployeeTaxData;

  constructor(
    private readonly name: string,
    private readonly email: string) {
  }

  setTaxData(ssn: string, salary: number): Employee {
    this.taxData = new EmployeeTaxData(ssn, salary);
    return this;
  }

  // ...
}

class EmployeeTaxData {
  constructor(
    public readonly ssn: string,
    public readonly salary: number) {
  }

  // ...
}
```



<br/><br/>





### 메소드 체이닝을 사용하자

메소드 체이닝은 매우 유용한 패턴으로 많은 라이브러리에서 공통적으로 사용하고 있다. 메서트 체이닝을 이용하면 코드를 간결하고 이해하기 쉽게 만들어준다. 

클래스 함수에서 모든 함수의 끝에 `this`를 리턴해줌으로써 클래스 메소드를 추가로 연결할 수 있다. 

**안좋은 예:**

```typescript
class QueryBuilder {
  private collection: string;
  private pageNumber: number = 1;
  private itemsPerPage: number = 100;
  private orderByFields: string[] = [];

  from(collection: string): void {
    this.collection = collection;
  }

  page(number: number, itemsPerPage: number = 100): void {
    this.pageNumber = number;
    this.itemsPerPage = itemsPerPage;
  }

  orderBy(...fields: string[]): void {
    this.orderByFields = fields;
  }

  build(): Query {
    // ...
  }
}

// ...

const queryBuilder = new QueryBuilder();
queryBuilder.from('users');
queryBuilder.page(1, 100);
queryBuilder.orderBy('firstName', 'lastName');

const query = queryBuilder.build();
```



**좋은 예:**

```typescript
class QueryBuilder {
  private collection: string;
  private pageNumber: number = 1;
  private itemsPerPage: number = 100;
  private orderByFields: string[] = [];

  from(collection: string): this {
    this.collection = collection;
    return this;
  }

  page(number: number, itemsPerPage: number = 100): this {
    this.pageNumber = number;
    this.itemsPerPage = itemsPerPage;
    return this;
  }

  orderBy(...fields: string[]): this {
    this.orderByFields = fields;
    return this;
  }

  build(): Query {
    // ...
  }
}

// ...

const query = new QueryBuilder()
  .from('users')
  .page(1, 100)
  .orderBy('firstName', 'lastName')
  .build();
```



<br/><br/>


































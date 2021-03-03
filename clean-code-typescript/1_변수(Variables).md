### 의미있는 변수 이름을 사용하자
**안좋은 예:** 
```typescript
function between<T>(a1: T, a2: T, a3: T): boolean {
  return a2 <= a1 && a1 <= a3;
}
```
**좋은 예:** 
```typescript
function between<T>(value: T, left: T, right: T): boolean {
  return left <= value && value <= right;
}
```
> **제네릭(Generics)** : 타입을 마치 함수의 파라미터처럼 사용하는 것
```typescript
function getText<string>(text: string): string {
  return text;
}
// <string>  : 입력 값 타입
// (): string  : 반환 값 타입
```

<br/>

### 발음할 수 있는 변수 이름을 사용하자
**좋은 예:**
```typescript
type Customer = {
  generationTimestamp: Date;
  modificationTimestamp: Date;
  recordId: number;
}
```

<br/>

### 동일한 유형의 변수는 동일한 단어를 사용하자
**안좋은 예:**
```typescript
function getUserInfo(): User;
function getClientDetails(): User;
function getCustomerData(): User;
```
**좋은 예:**
```typescript
function getUser(): User;
```

<br/>


### 검색할 수 있는 이름을 사용하자
**안좋은 예:**
```typescript
// 86400000이 도대체 뭐지?
setTimeout(restart, 86400000);
```
**좋은 예:**
```typescript
// 대문자로 이루어진 상수로 선언하세요.
const MILLISECONDS_IN_A_DAY = 24 * 60 * 60 * 1000;

setTimeout(restart, MILLISECONDS_IN_A_DAY);
```

<br/>

### 의도를 나타내는 변수를 사용하자
**안좋은 예:**
```typescript
declare const users: Map<string, User>;

for (const keyValue of users) {
  // users 맵을 순회
}
```
**좋은 예:**
```typescript
declare const users: Map<string, User>;

for (const [id, user] of users) {
  // users 맵을 순회
}
```
> **declare** 
> - 타입스크립트 컴파일러에게 해당 변수가 어딘가에 선언되어 있다고 알려주는 행위. 
> - 전역변수를 사용할 때도 되고 .d.ts 파일을 만들때도 사용된다.
>
> [참고](https://stackoverflow.com/questions/35019987/what-does-declare-do-in-export-declare-class-actions)

<br/>

### 암시하는 이름은 사용하지 말자
명시적인 것이 암시적인 것보다 좋다. 명료하게 작성하자.     
**안좋은 예:**
```typescript
const u = getUser();
const s = getSubscription();
const t = charge(u, s);
```
**좋은 예:**
```typescript
const user = getUser();
const subscription = getSubscription();
const transaction = charge(user, subscription);
```

<br/>

### 불필요한 문맥은 추가하지 말자
클래스/타입/객체의 이름에 의미가 담겨있다면, 변수 이름에서 반복하지 말자.     
**안좋은 예:**
```typescript
type Car = {
  carMake: string;
  carModel: string;
  carColor: string;
}

function print(car: Car): void {
  console.log(`${car.carMake} ${car.carModel} (${car.carColor})`);
}
```
**좋은 예:**
```typescript
type Car = {
  make: string;
  model: string;
  color: string;
}

function print(car: Car): void {
  console.log(`${car.make} ${car.model} (${car.color})`);
}
```
> **viod** : 비어있다는 의미. 
> - 타입스크립트는 function의 parameter와 return value도 타입을 명시해줘야 하는데, 주로 return value가 없을 때 void를 활용한다.

<br/>

### short circuiting이나 조건문 대신 기본 매개변수를 사용하자
기본 매개변수는 short circuiting보다 보통 명료하다.      
**안좋은 예:**
```typescript
function loadPages(count?: number) {
  const loadCount = count !== undefined ? count : 10;
  // ...
}
```
**좋은 예:**
```typescript
function loadPages(count: number = 10) {
  // ...
}
```

<br/>

### 의도를 알려주기 위해 `enum`을 사용하자
예를 들어, 값 자체보다 값이 구별되어야 할 때와 같이 코드의 의도를 알려줄 때 `enum`을 사용하면 된다.      
**안좋은 예:**
```typescript
const GENRE = {
  ROMANTIC: 'romantic',
  DRAMA: 'drama',
  COMEDY: 'comedy',
  DOCUMENTARY: 'documentary',
}

projector.configureFilm(GENRE.COMEDY);

class Projector {
  // Projector의 선언
  configureFilm(genre) {
    switch (genre) {
      case GENRE.ROMANTIC:
        // 실행되어야 하는 로직
    }
  }
}
```
**좋은 예:**
```typescript
enum GENRE {
  ROMANTIC,
  DRAMA,
  COMEDY,
  DOCUMENTARY,
}

projector.configureFilm(GENRE.COMEDY);

class Projector {
  // Projector의 선언
  configureFilm(genre) {
    switch (genre) {
      case GENRE.ROMANTIC:
        // 실행되어야 하는 로직
    }
  }
}
```




***


참고
- [clean-code-typescript](https://738.github.io/clean-code-typescript/)
- [clean-code-javascript](https://github.com/qkraudghgh/clean-code-javascript-ko#%EB%B3%80%EC%88%98variables)

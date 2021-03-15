## 에러 처리(Error Handling)

에러를 뱉는다는 것은 좋은 것이다. 프로그램에서 무언가가 잘못되었을 때 런타임에서 성공적으로 확인되면 현재 스택에서 함수 실행을 중단하고 (노드에서) 프로세스를 종료하고 스택 추적으로 콘솔에서 사용자에게 그 이유를 알려준다.

### `throw` 또는 `reject` 구문에서 항상 `Error` 타입을 사용하자

타입스크립트 뿐만 아니라 자바스크립트는 어떤 객체든지 에러를 `throw` 하는 것을 허용한다. 또한, 프로미스는 어떤 객체라도 거부될 수 있다.

`Error` 타입에는 `throw` 구문을 사용하는 것이 바람직하다. 에러가 상위 코드의 `catch` 구문에서 잡힐 수 있기 때문이다. 문자열 메시지가 잡히는 것은 디버깅을 더 고통스럽게 만든다. 이러한 이유로 `Error` 타입으로 프로미스를 거부해야 한다.

> [throw 참고](https://basarat.gitbook.io/typescript/type-system/exceptions)

> [throw 참고](https://basarat.gitbook.io/typescript/type-system/exceptions)



<br/>

**안좋은 예:**

```typescript
function calculateTotal(items: Item[]): number {
  throw 'Not implemented.';
}

function get(): Promise<Item[]> {
  return Promise.reject('Not implemented.');
}
```



**좋은 예:**

```typescript
function calculateTotal(items: Item[]): number {
  throw new Error('Not implemented.');
}

function get(): Promise<Item[]> {
  return Promise.reject(new Error('Not implemented.'));
}

// 또는 아래와 동일하다:

async function get(): Promise<Item[]> {
  throw new Error('Not implemented.');
}
```



<br/>

`Error` 타입을 사용하는 장점은 `try/catch/finally` 구문에 의해 지원되고, 암시적으로 모든 에러가 디버깅에 강한 `stack` 속성을 가지고 있기 때문이다.

또 하나의 대안이 있다. `throw` 구문을 사용하지 않는 대신, 항상 사용자의 정의 객체를 반환하는 것이다. 타입스크립트는 이것을 훨씬 더 쉽게 만든다. 아래 예제를 보자.

```typescript
type Result<R> = { isError: false, value: R };
type Failure<E> = { isError: true, error: E };
type Failable<R, E> = Result<R> | Failure<E>;

function calculateTotal(items: Item[]): Failable<number, 'empty'> {
  if (items.length === 0) {
    return { isError: true, error: 'empty' };
  }

  // ...
  return { isError: false, value: 42 };
}
```



<br/><br/>



### `catch` 절에서 에러 처리 부분을 비워두지 말자

댠순히 에러를 확인하는 것만으로는 에러를 해결할 수 없다. 또한, 콘솔에 에러를 출력할 경우 콘솔에 출력된 많은 내용 사이에서 해당 에러를 발견하기 어려울 수 있다. 따라서 콘솔에 에러를 출력하는 것은 좋지 않다.

코드를 `try`/`catch`로 감싼다면 해당 코드에서 에러가 발생할 수 있으며, 에러가 발생했을 때에 대한 계획이나 장치가 있어야 한다는 것을 의미한다.



<br/>

**안좋은 예:**

```typescript
try {
  functionThatMightThrow();
} catch (error) {
  console.log(error);
}

// 아래 예제는 훨씬 나쁘다.

try {
  functionThatMightThrow();
} catch (error) {
  // 에러를 무시
}
```



**좋은 예:**

```typescript
import { logger } from './logging'

try {
  functionThatMightThrow();
} catch (error) {
  logger.log(error);
}
```









<br/><br/>



### 요청이 거부된 프로미스 객체를 무시하지 말자

(Promise가 실패된 것을 무시하지 말자.)

마찬가지로 `try/catch` 절에서 받은 에러 처리 부분을 비워두면 안된다.

<br/>

**안좋은 예:**

```typescript
getUser()
  .then((user: User) => {
    return sendEmail(user.email, 'Welcome!');
  })
  .catch((error) => {
    console.log(error);
  });
```



**좋은 예:**

```typescript
import { logger } from './logging'

getUser()
  .then((user: User) => {
    return sendEmail(user.email, 'Welcome!');
  })
  .catch((error) => {
    logger.log(error);
  });

// 또는 async/await 구문을 사용할 수 있다:

try {
  const user = await getUser();
  await sendEmail(user.email, 'Welcome!');
} catch (error) {
  logger.log(error);
}
```




























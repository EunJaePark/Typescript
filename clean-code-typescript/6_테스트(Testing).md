## 테스트(Testing)

테스트는 배포하는 것보다 중요하다. 테스트 없이 배포한다는 것은 코드가 언제 오작동해도 이상하지 않다는 것과 같다. 훌륭한 테스트 프레임워크 뿐만 아니라, 훌륭한 [Coverage 도구](http://gotwarlost.github.io/istanbul/)를 사용해야 한다.

타입스크립트 타입을 지원하는 [좋은 자바스크립트 테스트 프레임워크](http://jstherightway.org/#testing-tools)가 많이 있다. 새로운 기능/모듈을 짤 때 테스트 코드를 항상 작성하도록 하자. 

테스트 기반 개발(TDD)을 선호한다면 훌륭한 개발 방법이 될 수 있다. 하지만 어떠한 기능을 개발하거나 코드를 리팩토링 할 때 무엇보다 목표하는 Coverage를 달성하는 것이 중요하다.



### TDD의 세 가지 법칙

1. 실패하는 단위 테스트를 작성하기 전에는 실제 코드를 작성하지 말아야 한다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 장성해야 한다.
3. 실패하는 단위 테스트를 통과할 정도로만 실제 코드를 작성해야 한다.



<br/><br/>



### F.I.R.S.T 규칙

명료한 테스트는 다음 규칙을 따라야 한다.

- **Fast** : 테스트는 빈번하게 실행되므로 빨라야 한다.
- **Independent** : 테스트는 서로 종속적이지 않아야 한다. 독립적으로 실행하든지 순서 상관없이 모두 실행하든지 동일한 결과가 나와야 한다.
- **Repeatable** : 테스트는 어떠한 환경에서도 반복될 수 있다. 테스트가 실패하는데에 이유가 없어야 한다.(?)
- **Self-Validating** : 테스트는 **통과** 혹은 **실패**로 답해야 한다. 테스트가 통과되었다면 로그 파일을 보며 비교할 필요가 없다.
- **Timely** : 단위 테스트는 실제 코드를 작성하기 전에 작성해야 한다. 실제 코드를 작성한 후에 테스트를 작성한다면, 테스트를 작성하기에 어려움이 있을 것이다.



<br/><br/>







### 테스트 하나에 하나의 개념을 작성하자

테스트는 **단일 책임 원칙**을 따라야 한다. 단위 테스트 하나 당 하나의 assert 구문을 작성하자.

> **assert 함수**
>
> 디버깅 모드에서 개발자가 오류가 생기면 치명적인 것이라는 곳에 심어 놓는 에러 검출용 코드이다.

**안좋은 예:**

```typescript
import { assert } from 'chai';

describe('AwesomeDate', () => {
  it('handles date boundaries', () => {
    let date: AwesomeDate;

    date = new AwesomeDate('1/1/2015');
    assert.equal('1/31/2015', date.addDays(30));

    date = new AwesomeDate('2/1/2016');
    assert.equal('2/29/2016', date.addDays(28));

    date = new AwesomeDate('2/1/2015');
    assert.equal('3/1/2015', date.addDays(28));
  });
});
```



**좋은 예:**

```typescript
import { assert } from 'chai';

describe('AwesomeDate', () => {
  it('handles 30-day months', () => {
    const date = new AwesomeDate('1/1/2015');
    assert.equal('1/31/2015', date.addDays(30));
  });

  it('handles leap year', () => {
    const date = new AwesomeDate('2/1/2016');
    assert.equal('2/29/2016', date.addDays(28));
  });

  it('handles non-leap year', () => {
    const date = new AwesomeDate('2/1/2015');
    assert.equal('3/1/2015', date.addDays(28));
  });
});
```



<br/><br/>





### 테스트의 이름은 테스트 의도가 드러아냐 한다

테스트가 실패할 때, 테스트의 이름은 어떤 것이 잘못되었는지 볼 수 있는 첫 번째 표시가 된다.

**안좋은 예:**

```typescript
describe('Calendar', () => {
  it('2/29/2020', () => {
    // ...
  });

  it('throws', () => {
    // ...
  });
});
```



**좋은 예:**

```typescript
describe('Calendar', () => {
  it('should handle leap year', () => {
    // ...
  });

  it('should throw when format is invalid', () => {
    // ...
  });
});
```



<br/>



<br/><br/>


































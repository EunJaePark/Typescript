## 동시성(Concurrency)

### Callback 대신 Promise를 사용하자

콜백은 깔끔하지 않고, 많이 중첩된 **콜백 지옥**을 만들어낸다. 콜백 방식 보다 promise 방식을 사용해주자. (Promise는 ES2015/ES6에서 내장되어있다.)

<br/>

**안좋은 예:**

```typescript
import { get } from 'request';
import { writeFile } from 'fs';

function downloadPage(url: string, saveTo: string, callback: (error: Error, content?: string) => void) {
  get(url, (error, response) => {
    if (error) {
      callback(error);
    } else {
      writeFile(saveTo, response.body, (error) => {
        if (error) {
          callback(error);
        } else {
          callback(null, response.body);
        }
      });
    }
  });
}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html', (error, content) => {
  if (error) {
    console.error(error);
  } else {
    console.log(content);
  }
});
```



**좋은 예:**

```typescript
import { get } from 'request';
import { writeFile } from 'fs';
import { promisify } from 'util';

const write = promisify(writeFile);

function downloadPage(url: string, saveTo: string): Promise<string> {
  return get(url)
    .then(response => write(saveTo, response));
}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html')
  .then(content => console.log(content))
  .catch(error => console.error(error));  
```

<br/>

프로미스는 코드를 더욱 간결하게 해주는 헬퍼 메소드를 지원한다.

| 패턴                     | 설명                                                         |
| :----------------------- | ------------------------------------------------------------ |
| `Promise.resolve(value)` | 해결(resolve) 된 프로미스로 값을 변환.                       |
| `Promise.reject(error)`  | 거부(reject) 된 프로미스로 에러를 변환.                      |
| `Promise.all(promises)`  | 전달된 모든 프로미스가 이행한 값의 배열을 이행하는 새 프로미스 객체를 반환하거나 거부된 첫 번째 프로미스의 이유로 거부. |
| `Promise.race(promises)` | 전달된 프로미스의 배열에서 가장 먼저 완료된 결과/에러로 이행/거부 된 새 프로미스 객체를 반환. |

`Promise.all`은 병렬적으로 작업을 수행할 때 유용하다. `Promise.race`는 프로미스를 위한 타임아웃과 같은 것을 구현할 때 유용하다.



<br/><br/>



### `async`/`await`은 Promise보다 더 깔끔하다

async/await 구문을 사용하면 연결된 프로미스 구문보다 훨씬 명료하고 이해하기 쉬운 코드를 작성할 수 있다. `async` 키워드가 앞에 붙여진 함수는 `await` 키워드에서 코드 실행을 멈춘다는 것을 자바스크립트 런타임에 알려준다.



<br/>

**안좋은 예:**

```typescript
import { get } from 'request';
import { writeFile } from 'fs';
import { promisify } from 'util';

const write = util.promisify(writeFile);

function downloadPage(url: string, saveTo: string): Promise<string> {
  return get(url).then(response => write(saveTo, response));
}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html')
  .then(content => console.log(content))
  .catch(error => console.error(error));  
```



**좋은 예:**

```typescript
import { get } from 'request';
import { writeFile } from 'fs';
import { promisify } from 'util';

const write = promisify(writeFile);

async function downloadPage(url: string, saveTo: string): Promise<string> {
  const response = await get(url);
  await write(saveTo, response);
  return response;
}

// somewhere in an async function
try {
  const content = await downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html');
  console.log(content);
} catch (error) {
  console.error(error);
}
```




































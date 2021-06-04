### 비동기 import
> import 구문은 정적으로 두 가지 제약이 존재   
> - 모듈 식별자는 실행 중에 생성될 수 없다
> - 모듈의 import는 모든 파일의 최상위에 선언되며, 제어 구문 내에 포함될 수 없다.

- 사용자 운영체제에 의존하거나 현재 사용자 언어를 위한 특정 번역 모듈을 새로 import 해야한다면? 
- 혹은 상대적으로 무거운 모듈을 사용하고자 할 때 기능의 특정 부분에만 접근하려 한다면?
> 비동기 import가 필요하다.   
> 비동기 import는 import()연산자를 사용하며, 문법적으로 모듈 식별자를 인자로 취하고 모듈 객체를 promise로 반환하는 함수와 동일.   

- 여러 언어로 Hello world를 출력하는 커맨드라인 application을 만드는 예제

```ecmascript 6
// strings-kr.js
export const HELLO = '안녕 세계'
// strings-en.js
export const HELLO = 'Hello World'
// strings-es.js
export const HELLO = 'Hola mundo'

// main.js
const SUPPORTED_LANGUAGES = ['kr', 'en', 'es'] // (1)
const selectedLanguage = process.argv[0] // (2)

if (!SUPPORTED_LANGUAGES.includes(selectedLanguage)) { // (3)
    console.error('지원하지 않습니다')
    process.exit(1)
}

const translationModule = `./strings-${selectedLanguage}.js` // (4)
import(translationModule)  // (5)
    .then((string) => {    // (6)
        console.log(strings.HELLO)
    })

```
1. 지원되는 언어 리스트 정의
2. 선택한 언어를 커맨드라인의 첫 번째 인자로 받는다.
3. 지원되지 않는 언어가 선택된 경우를 처리한다.
4. 우선 선택된 언어를 사용하여 import하고자 하는 모듈의 이름을 동적으로 만든다. 상대경로 사용을 위해 ./를 붙인다.
5. 모듈의 동적 import를 위해 import() 연산자를 사용한다
6. 동적 import는 비동기적으로 된다. 그러므로 모듈이 사용될 준비가 되었을 때를 알기 위해서 .then()을 반환된 promise에 사용한다.
모듈이 완전히 적재되었을 때, then()으로 전달된 함수가 실행된다. strings는 동적 import 된 모듈의 네임스페이스가 된다.
마지막으로 strings.HELLO에 접근할 수 있으며 콘솔에 값이 출력된다.
   

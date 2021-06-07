### 모듈 적재 이해하기
> 파일은 종속성 확인을 위한 진입점이며, 인터프리터는 진입점에서부터 필요한 모든 코드가
> 탐색되고 평가될 때 까지 import 구문을 재귀적인 깊이 우선 탐색(DFS)으로 찾는다.

* 로딩 단계 - 1단계 모든 점들을 찾고, 2단계 각 점들을 연결하여 길을 만들고, 3단계 올바른 순서로 길을 걷는다.
    1. 생성(또는 파싱): 모든 import 구문을 찾고 재귀적으로 각 파일로부터 모든 모듈의 내용을 적재
    2. 인스턴스화 : export된 모든 개체들에 대해 명명된 참조를 메모리에 유지. 또한 모든 import 및 export 문에 대한 참조가
    생성되어 이들 간의 종속성 관계(linking)를 추적한다. 이 단계에선 어떤 JavaScript 코드도 실행되지 않는다.
    3. 평가: Node.js는 마지막으로 코드를 실행하여 이전에 인스턴스화 된 모든 개체가 실제 값을 얻을 수 있도록 한다.
    이제 모든 준비가 되었기 때문에 진입점에서 코드 실행이 가능하다.
> ESM은 3단계가 완전히 분리되어 있다. 종속성 그래프가 완전해지기 전까지는 어떠한 코드도 실행되지 않는다.   
> 그러므로 모듈 import와 export는 정적이어야 한다.

#### 읽기 전용 라이브 바인딩
```ecmascript 6
export let count = 0;
export function increment() {
    count++;
}

// main.js
import { count, increment } from './counter.js';
console.log(count) // 0을 출력
increment();
console.log(count) // 1을 출력
count++ // TypeError: Assignment to constant variable;
```

* 스코프 내에 개체가 import 되었을 때, 사용자 코드의 직접적 제어 밖에 있는 binding value는 그것이 원래 존재하던 모듈(live binding)
에서 바뀌지 않는 한, 원래의 값에 대한 바인딩이 변경 불가(read-only binding)하다는 것이다.

* CommonJS는 모듈로부터 require 되었을 때 export 객체 전체가 얕은 복사로 이뤄진다. 이것이 의미하는 것은
숫자난 문자열과 같은 원시 변수에 있는 값이 나중에 바뀌었을 때 이것을 제공한 모듈은 변화를 알지 못한다는 것이다. 이 부분이 ESM과 근본적으로 다르다.
  
  

#### ESM 시스템의 순환 종속성 파싱 모듈 분석 3단계
> main.js 는 a,b.js를 참조   
> a.js는 b를 참조   
> b.js는 a를 참조   

```ecmascript 6
// 예제 코드
// a.js
import * as bModule from './b.js';
export let loaded = false;
export const b = bModule;
loaded = true;

// b.js
import * as aModule from './a.js';
export let loaded = false;
export const a = aModule;
loaded = true;

// main.js
import * as a from './a.js';
import * as b from './b.js';
console.log('a->', a);
console.log('b->', b);
``` 

1. 파싱
  - 진입점(main.js)부터 인터프리터가 dfs 방식으로 모듈을 1 회씩 방문하며 import 구문을 탐색.
    1. main.js에서 발견된 import문이 a.js로 향함
    2. a에서 b로 가는import문 발견
    3. b에서 a로 가는 import문이 있지만 이미 방문했기에 가지 않음
    4. b가 다른import문을 갖고 있지 않으므로 탐색이 a로 돌아감. a는 b로 갔던 것 외에 따로 없으므로 main으로 복귀.
    5. main에서 b의 import지점 발견하지만 이미 탐색한 경로라 생략
> 결과 : main.js->a.js->b.js로 순환이 제거된 선형적 형태의 모듈 그래프가 생성됨  

2. 인스턴스화
  - 인터프리터가 이전 단계에서 얻어진 트리구조를 따라 아래에서 위로 움직임(콜스택)
  - 모든 모듈에서 exports 된 속성을 먼저 찾고 나서 메모리에 exports된 이름의 map을 만든다.
    - 인터프리터는 b->a->main 순으로 exports 를 캐치한다
    - 각각의 스크립트에서 exports 하는 것을 캐치하며 이동한다
    - 마지막 단계에서 exports map은 exports된 이름의 추적만을 유지한다.
  - 연관된 값은 현재로는 인스턴스화 되지 않은 것으로 간주된다.
  - 상세
    1. 모듈 b는 aModule라는 이름으로 a.js에서의 익스포트를 연결
    2. 모듈 a는 bModule라는 이름으로 b.js에서의 익스포트를 연결
    3. 마지막으로 main.js는 b라는 이름으로 b.js의 모든 익스포트를 임폴트.
       비슷하게 a라는 이름으로 a에서의 모든 익스포트를 임포트한다
    4. 이 단계에서는 다음 단계의 마지막에 사용 가능한 값에 대한 참조만을 연결하고 아직 인스턴스화 되지 않음.   

3. 평가
  - 모든 파일의 모든 코드가 원래의 종속성 그래프에서 후위 깊이 우선 탐색으로 아래에서 위로 올라가며 실행됨.
  - 이러한 방식은 메인 비즈니스 로직 수행 전에 익스포트된 모든 값이 초기화 되는 것을 보장
  - 결과
    1. b.js부터 수행되며. 첫 번째 라인은 모듈에서 익스포트되는 loaded값이 false로 평가
    2. 마찬가지로, 익스포트되는 속성 a가 평가. 이번에는 export 맵의 모듈 a.js를 나타내는 모듈 객체에 대한 참조로 평가
    3. loaded 속성의 값이 true로 바뀐다. 이 시점에서 모듈 b.js의 익스포트 상태가 완전히 평가되었습니다.
    4. 이제 a.js로 수행이 이동되고, 다시 loaded를 false로 설정하는 것으로 시작합니다.
    5. 이 때, export b가 익스포트 맵에서 모듈 b.js에 대한 참조로 평가된다.
    6. 마지막으로 loaded 속성은 true로 바뀐다. 이제 우리는 모듈 a.js에서도 완전이 평가된 모든 익스포트를 갖게 된다.
> 결과 : 모든 단계를 거치고 나서 main.js의 코드가 실행되며, 이 때 exports 된 모든 속성값들은 완전히 평가된 상태다.   
> import된 모든 모듈들은 참조로 추적되고 우리는 순환 종속성이 존재하는 상황에서도 모든 모듈이 다른 모듈의 최신 상태를   
> 갖고 있음을 확신할 수 있다.

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
  
  


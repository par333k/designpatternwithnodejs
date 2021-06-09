### ESM과 CommonJS의 차이

* strict 모드에서의 ESM
    - ES 모듈은 암시적으로 strict mode에서 실행되므로 "use strict" 구문을 추가할 필요가 없다.
    - 따라서 ESM에서는 strict mode를 변경할 수 없다.

* ESM에서의 참조 유실
    - strict mode에서 실행되기 떄문에 CommonJS의 몇 가지 중요한 참조가 유실된다(require, export, module, __filename 등)
    - EMS에서는 특별한 객체인 import.meta를 사용하여 현재 파일에 대한 잠조를 얻을 수 있다.
    - ESM에서 require() 함수를 재구성 할 수 있다.
    ```ecmascript 6
    import { createRequire }  from 'module';
    const require = createRequire(import.meta.url);
    // ES 모듈의 문맥상에서 CommonJS의 module의 기능을 import 하는데 require()를 사용할 수 있다. 
    ```
    - this의 동작이 다르다. ESM의 전역범위에서 this 는 undefined인 반면 CommonJS는 this가 exports와 같은 참조를 한다.
* 상호 운용
    - ESM에서 표준 import 문법을 사용해 CommonJS모듈을 import하는 것 또한 가능하다. default exports에 한정된다.
    ```ecmascript 6
    import packageMain from 'commonjs-package' // 작동
    import { method } from 'commonjs-package' // 에러
    ```
    - CommonJS 모듈에서 ES모듈 import는 불가능하다.
    - CommonJS 에서 json 파일을 직접적으로 가져오는 것은 불가능하다.
    - json 파일을 직접 가져오고 싶을 때 위의 참조유실 예제처럼 createRequire를 활용한다. (버전업이 일어나면 해결될 수 있다)


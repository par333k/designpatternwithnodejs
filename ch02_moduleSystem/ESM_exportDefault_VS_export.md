### ESM
> ESM은 ES2015이후의 모듈시스템, Node.js는 CommonJS를 기본으로 쓰기 때문에   
> ESM을 쓰려면 package.json에 type:module 을 기재해야 에러가 나지 않는다
 
* CommonJS에서 쓰이는 module.exports와 비슷하게 쓰이는 것으로 export default가 있다.

```ecmascript 6
// logger.js
export default class Logger {
    constructor(name) {
        this.name = name
    }
    
    log(message) {
        console.log(`[${this.name}] ${message}`)
    }
} 
```
- 예제의 경우 export 되는 객체는 default라는 이름 아래 등록됨, Logger라는 이름은 무시.

```ecmascript 6
// main.js
import MyLogger from './logger.js'
const logger = new MyLogger('info');
logger.log('Hello world')
```
- default export는 이름이 없는 것으로 간주되기 때문에 자유롭게 이름 선언 가능
- default 라는 이름을 개체를 명시적으로 import 할 수는 없다
```ecmascript 6
// 에러난다
import { default } from './logger.js'
```

* export와 default export의 혼합 사용 가능
```ecmascript 6
// mylog는 default로 export, info는 그냥 export
import mylog, { info } from './logger.js'
```

### export default와 export 차이

* 이름을 가진 export는 IDE의 자동 import, 자동완성, 리팩토링 툴에 더 잘 적응한다.
* default export는 주어진 기능이 서로 다른 파일에서 서로 다른 이름을 가질 수 있어 더 복잡하다.
주어진 이름에 어떠한 모듈이 적용될 것인지 추론하기 어렵기 때문.
* default export는 모듈에서 가장 핵심적인 한 가지 기능과 연결하는 편리한 방법
* 그러나 특정 상황에서 사용하지 않는 코드를 제거하는 tree shaking 최적화 작업을 어렵게 함.
    - 예를들어, 모듈이 객체의 속성을 이용해서 모든 기능을 노출시키는 default export만 제공할 수도 있다.
    우리가 이 객체를 import 했을 때 모듈 번들러는 객체의 전체가 사용되는 것으로 간주하여 노출된 기능 중에
      사용되지 않는 코드를 제거할 수 없게 된다.
      
* 명확하게 하나의 기능을 export 하고 싶을 때는 default export를 사용해도 되지만, 대부분의 경우 이름을 사용한 export를 사용하는 것이 좋다.

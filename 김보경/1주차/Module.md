# 모듈 패턴
- 모듈패턴은 코드들을 재사용 가능하면서도 작게 나눌 수 있도록 해줍니다.
- 또한 모듈을 코드로 나누는 과정에 특정 변수들을 파일 내에 private하게 할 수 있는데, 모듈 스코프 내에 변수를 선언하고 명시적으로 외부에 export 하지 않으면 바깥에서 해당 변수에 접근할 수 없습니다.
  - 이를 통해 전역 스코프의 변수들과 이름이 충돌하는 문제를 줄일 수 있습니다.

> 사이트에서는 다루지 않았는데, 모듈타입이 어떻게 발전되어 왔는지 스크립트로더, AMD, UMD, CJS, ESM의 차이를 따로 알아보는게 더 좋을거같습니다.
> [모던 자바스크립트 튜토리얼 - 모듈소개](https://ko.javascript.info/modules-intro)
> [모던 자바스크립트 튜토리얼 - 모듈 내보내고 가져오기](https://ko.javascript.info/import-export)
> [모던 자바스크립트 튜토리얼 - 동적으로 모듈 가져오기](https://ko.javascript.info/modules-dynamic-imports)
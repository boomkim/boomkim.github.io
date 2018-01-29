---
layout: post
title:  자바스크립트는 어디에 넣어야 할까? (head? 아니면 body의 끝?)
categories: javascripts
---
자바스크립트를 작성하고 html 파일에 포함시킬 때 head에 넣는 경우도 보고 body 끝에 넣는 경우도 보았을 것이다. 그동안은 깊게 생각해보지 않고 그냥 `dom 객체가 모두 로드 된 후에 javascript가 로드가 되는게 좋겠지` 싶어서 body태그가 닫히기 전에 넣어놨었다. 그리고 아무런 문제도 없었다.

그런데 계속해서 같은 script를 반복해서 넣다보니 head에 넣는게 구조상으로는 더 편할 수도 있겠다는 생각이 들었다. nodejs를 공부하면서 view 엔진으로 [pug][pug]를 사용하고 있는데 pug에서 물론 `include`를 통해 javascript를 넣는 것을 지원하긴 하지만 그냥 layout에 반복되는 javascript까지 모두 포함시켜 놓고 다른 view에서 extends만 시키는게 더 편하기 때문이다. 헤드, navbar 등 모든 페이지에 사용되는 것들을 하나로 묶고 다른 문서에서는 그냥 내용만 간단하게 쓰고 싶어서 지금까지는 그렇게 했었다. 그래서 그냥 javascript를 head에 넣어도 상관이 없나 궁금해졌다. 

서론이 길었는데 결론부터 말하자면 

## javascript를 head에 링크하는 것은 좋지 않은 생각이다.

이유는 간단하다. 자바스크립트를 head에 링크시키면 dom객체가 로드되기 이전에 자바스크립트가 먼저 로드될 수 있기 때문이다. 또한 javascript가 로드되다가 문제가 발생하면 병렬 다운로드가 중지된다. 웹페이지의 성능에도 영향을 준다는 것이다. 

그렇지만 꼭 body의 끝일 필요는 있을까? 예를 들어 navbar에만 필요한 javascript의 경우 굳이 body의 끝일 필요는 없을것 같다. 만약 기본적인 화면구성에만 필요한 태그, 스크립트들만 모아 하나의 layout을 구성한다면 그 구성 가장 아래에 javascript를 두어서 javascript가 적용될 DOM보다 먼저 load되는 일은 막을 수 있다. 만약 이게 코드 작성자의 스타일이라면 굳이 바꿀 필요는 없는 것 같다. 그렇지만 대세, 그리고 가장 안전한건 `</body> 태그로 닫기 전에 javascript를 넣는 것` 인듯 하다. 

참고: [Javascript - head, body or jQuery?][stackoverflow]


[pug]: https://pugjs.org/
[stackoverflow]: https://stackoverflow.com/questions/10994335/javascript-head-body-or-jquery

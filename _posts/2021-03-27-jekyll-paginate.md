---
layout: post
title:  "Github.io 블로그(with Jekyll) - jekyll-paginate-v2 문제"
date:   2021-03-27 17:30:00
author: Yeonjun
categories: Blog
comments: true
---

Github Blog를 만들면서 겪었던 문제를 포스팅합니다.

먼저, Jekyll은 github에서 제공하는 static website generator입니다. Github repository에서 jekyll blog를 위한 code를 push 하면, github가 static website로 만들어줍니다.

저는 `centraium`이라는 Jekyll Theme를 사용하여 Github Blog를 만드는데 성공했습니다.

**하지만, Local 환경에서는 정상적으로 작동하지만, Github repository에 code를 push 했을 때 Main 화면에서 포스팅을 볼 수 없는 문제에 봉착했습니다.**

---
## 문제
해당 문제의 원인은 centraium이 사용하는 `jekyll-paginate-v2` plugin이 github에서 생성되는 static website에서는 동작하지 않는 것입니다.

jekyll-paginate-v2 github의 README.md를 보면 
> ⚠️ Please note that this plugin is NOT supported by GitHub pages. 

라고 명시되어 있습니다. github에서 제공하는 dependency에 포함되어 있지 않기 때문에, 정상적으로 작동하지 않는 것이었습니다.

---
## 해결

저는 해당 문제를 해결하기 위해 jekyll-paginate-v2 플러그인 대신에 `jekyll-paginate`을 사용했습니다.

_config.yml 파일의 jekyll-paginate-v2을 jekyll-paginate로 바꾸고, 

jekyll-paginate를 Gemfilr과 _config.yml에 설정하고 `bundle install` 명령어를 실행하면 해당 플러그인이 설치됩니다.

Github repository에 code를 push하고, Github Blog에 들어가면 정상적으로 작동하는 것을 볼 수 있습니다.

---
## 결론
처음 문제에 이르렀을 때, 무엇이 문제의 원인인지 몰라서 한참을 헤맸습니다. 문제의 원인이 되는 코드가 무엇인지 파악하는 것이 중요함을 느꼈습니다. 또한, 검색에 의존하지 않고 플러그인의 공식 문서를 먼저 확인했다면 문제를 해결하는데 오랜 시간을 들이지 않았을 것입니다. 공식 문서의 중요성을 느꼈습니다.
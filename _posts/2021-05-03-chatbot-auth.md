---
layout: post
title:  "[카카오 오픈빌더로 챗봇 만들기] 인증된 사용자에게만 서비스 제공하기"
date:   2021-05-03 13:30:00
author: Yeonjun
categories: Chatbot
comments: true
---
Kakao I OpenBuilder를 사용해 챗봇을 만들면서 인증된 사용자에게만 서비스를 제공하는 방법을 기록하고자 글을 작성합니다.

글을 시작하기에 앞서, 본인은 Python3 + Flask 프레임워크를 사용해 챗봇의 백엔드를 구현했음을 말씀드립니다.

---
## 용어 설명
Kakao I OpenBuilder에서는 사용자가 요청하는 서비스 단위를 `스킬` 이라고 부릅니다.
`스킬 서버`는 챗봇 시스템으로부터 스킬 요청을 받고 이에 담긴 정보를 분석하여 적절한 응답을 만들어 전송합니다. 각각의 요청은 HTTP POST를 통해서 전달되고, 요청과 응답 모두 JSON으로 구성된 body를 이용합니다.

---
## 과제 개요
저는 사내 업무 지원 카카오 챗봇을 개발하면서, 팀원들만 이용 가능한 서비스를 구현하고 싶었습니다. 챗봇과 카카오 채널을 연결하기 위해서는 `공개된 카카오 채널`에만 챗봇을 적용시킬 수 있습니다. 공개된 카카오 채널에서 어떻게 사용자 인증을 구현했는지 소개하고자 합니다.

---
## 과정
챗봇 시스템이 스킬 서버에 질의를 할 때 보내는 body<json> 파일을 자세히 보면 다음과 같습니다.

![1](/assets/2021-05-03/image1.png)

여기서 중요하게 봐야할 것은 `userRequest`입니다.
카카오 챗봇 시스템은 채널을 추가한 사용자에게 고유한 user_id를 부여합니다. 스킬 서버는 이 user_id를 통해 사용자를 구분할 수 있습니다.

챗봇 채널을 처음 추가한 사용자에게 사용자 인증을 요구합니다. 사용자 인증 로직은 사내 정보를 활용하여 진행합니다.

사용자가 성공적으로 인증을 완료한다면, 사용자의 user_id를 서버에 저장하여 관리합니다. 또한, 각 스킬에 인증된 사용자의 질의인지 체크하는 로직을 추가했습니다.

```python
# skill.py
if super().checkUserAuth() is False:
            return super().message.getAuthErrMsg()

# checkUserAuth()
def checkUserAuth(self):
        if self.user_id in self.AUTH_LIST:
            return True
        else:
            return False
```
파일에 저장되어 있는 user_id를 List에 저장하고, 사용자의 user_id가 존재하는지 확인합니다.
인증된 사용자가 아니라면, 스킬 서버는 사용자에게 AuthErrMsg를 반환하게 됩니다.

간단하게 카카오 챗봇을 사용하여 사용자 인증 과정을 알아봤습니다.
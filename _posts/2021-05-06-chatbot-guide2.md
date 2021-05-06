---
layout: post
title:  "[카카오 오픈빌더로 챗봇 만들기] 카카오 오픈빌더 챗봇 가이드 - 2(Server Version)"
date:   2021-05-06 10:30:00
author: Yeonjun
categories: Chatbot
comments: true
---
해당 글은 카카오 오픈빌더 챗봇 가이드 - 1(Non Server Version)의 내용에 파이썬을 활용한 백엔드 구성을 추가한 내용입니다.

Kakao I OpenBuilder에서는 사용자가 요청하는 서비스 단위를 스킬 이라고 부릅니다. 스킬 서버는 챗봇 시스템으로부터 스킬 요청을 받고 이에 담긴 정보를 분석하여 적절한 응답을 만들어 전송합니다. 각각의 요청은 HTTP POST를 통해서 전달되고, 요청과 응답 모두 JSON으로 구성된 body를 이용합니다.

## 1. 로컬 웹서버 생성
---
먼저, 로컬에 Flask 패키지를 설치합니다.

터미널에
```
pip install flask
```
명령어를 입력해줍니다.

python3와 Flask 프레임워크를 사용하여 간단한 웹 서버를 만들 수 있습니다.

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/")
def hello():
	return "Hello, Flask!"

@app.route("/testApi",methods=['POST'])
def testApi():
	#카카오 오픈빌더로 응답을 전송하는 함수
	req = request.get_json()
    print(req.text)
	
	

if __name__ == '__main__':
	app.run()
```

http://localhost:5000/testApi를 통해 카카오톡에서 전달된 발화내용을 처리하도록 하겠습니다.

## 2. 서버 배포
---
서버를 배포할 수 있는 방법은 굉장히 많습니다. 저는 그 중에 **Localhost Tunneling**을 통해서 외부에서 Localhost 환경에 접근할 수 있는 url을 생성했습니다.

(제가 사용한 방법 외에 **Heroku**라는 Paas 클라우드 서비스를 사용하면, 빠르게 배포하고 챗봇을 제작할 수 있습니다.)

Localhost Tunneling을 구현한 기술 중에서, pyngrok 패키지를 사용했습니다.

터미널에
```
pip install pyngrok
```
명령어를 입력하여 pyngrok 패키지를 설치합니다.

그 다음, [pyngrok 공식문서](https://pyngrok.readthedocs.io/en/latest/index.html) 에 나온 가이드에 참고합니다.

```python
from flask import Flask, request
import os
def init_webhooks(base_url):
    pass


def create_app():
    app = Flask(__name__)

    # Initialize our ngrok settings into Flask
    app.config.from_mapping(
        BASE_URL="http://localhost:5000",
        USE_NGROK=os.environ.get("USE_NGROK", "False") == "True" and os.environ.get("WERKZEUG_RUN_MAIN") != "true",

    )

    if app.config.get("ENV") == "development" and app.config["USE_NGROK"]:
        # pyngrok will only be installed, and should only ever be initialized, in a dev environment
        from pyngrok import ngrok
        ngrok.set_auth_token(os.getenv('ngrok_api_key'))

        # Get the dev server port (defaults to 5000 for Flask, can be overridden with `--port`
        # when starting the server
        port = sys.argv[sys.argv.index("--port") + 1] if "--port" in sys.argv else 5000

        # Open a ngrok tunnel to the dev server
        public_url = ngrok.connect(port).public_url
        print(" * ngrok tunnel \"{}\" -> \"http://127.0.0.1:{}\"".format(public_url, port))

        # Update any base URLs or webhooks to use the public ngrok URL
        app.config["BASE_URL"] = public_url
        init_webhooks(public_url)

    # ... Initialize Blueprints and the rest of our app

    return app
```
코드 중간에 ngrok.set_auth_token을 설정한 부분이 있습니다. ngrok 공식 홈페이지에서 인증번호를 받은 후, 적용하면 최대 8시간 까지 유지되던 url이 서버 재실행 전까지 유지됨을 보장해줄 수 있습니다.

- **ngrok 인증번호 발급 방법**

    [https://ngrok.com/](https://ngrok.com/) 접속 → 회원가입 → Your Authtoken → 인증 번호 Copy하여 붙여넣기.

- **환경 변수 세팅**

    [pyngrok 공식 문서](https://pyngrok.readthedocs.io/en/latest/integrations.html#flask)에는 Flask와 pyngrok를 위한 사용 방법이 기재되어 있습니다. 앱을 실행하기 전에 환경 변수를 설정해야 됩니다.

    ![1](/assets/2021-05-06-post-image1.png)

    위 그림과 같은 환경 변수를 설정하기 위해 dotenv 패키지를 사용했습니다. dotenv는 환경변수 파일을 외부에 만들고, 관리할 수 있게 해주는 패키지입니다.
    
    터미널에
    ```
    pip install python-dotenv
    ```
    명령어를 입력하여, dotenv 패키지를 설치합니다.

    디렉토리 내에 **.env** 파일을 생성하여 환경 변수를 설정하면 됩니다.
    ```
    USE_NGROK=True
    FLASK_ENV=development
    FLASK_APP=app.py
    ngrok_api_key=(본인이 발급받은 ngrok 인증 번호 입력)
    ```

위의 과정들을 마치고 터미널에
```
flask run
```
명령어를 입력하면, localhost환경에 접속할 수 있는 랜덤한 url이 생성됨을 볼 수 있습니다.

![2](/assets/2021-05-06-post-image2.png)

## 3. 스킬 등록
---
카카오 오픈빌더로 돌아와서 스킬 탭으로 이동합니다.

![3](/assets/2021-05-06-post-image3.png)

새로운 스킬을 만들고, 방금 생성했던 URL 및 정보를 입력하고 저장합니다.

## 4. 블록과 스킬 연결
---
![4](/assets/2021-05-06-post-image4.png)

테스트라는 블록을 새로 만들고, 요일을 입력받을 수 있게 파라미터를 설정했습니다.

이 블록과 3번에서 만든 테스트 스킬을 연결했습니다. 봇 응답 또한 스킬데이터 사용으로 설정하여, 스킬 서버로부터 응답을 받도록 합니다.

## 5. 응답 테스트
---
![5](/assets/2021-05-06-post-image5.png)

현재 블록에서는 date라는 필수 파라미터를 입력 받아, 이를 스킬 서버에 전달합니다. 아직 스킬 서버에서 어떠한 응답 처리를 하지 않았기 때문에 아무내용도 응답하지 않습니다.

서버의 로그를 확인해보면 다음과 같습니다.

![6](/assets/2021-05-06-post-image6.png)

사용자가 어떠한 텍스트를 입력했을 때 위와 같은 JSON 요청이 오는 것을 볼 수 있습니다. 

JSON > action > detailParams > date > value 를 통해 사용자가 입력한 발화를 확인할 수 있습니다.

## 6. 응답 만들기
---

스킬 서버에서 적절한 응답을 전송해줘야 합니다. 응답의 형식은 카카오 오픈빌더 도움말(https://i.kakao.com/docs/skill-response-format)에서 확인할 수 있습니다.

```python
@app.route("/testApi",methods=['Post'])
def testApi():
    req = request.get_json()
    date = req['action']['detailParams']['date']['value']
    res = {
        "version": "2.0",
        "template": {
            "outputs": [
                {
                    "simpleText": {
                        "text": "사용자가 입력한 요일은 " + date + "입니다."
                    }
                }
            ]
        }
    }
    return jsonify(res)
```
응답 타입별 JSON 포맷을 참고하여 JSON 규격을 만듭니다. 사용자가 테스트 블록에 진입하여, 요일을 입력하면 **'사용자가 입력한 요일은 O요일 입니다.'** 라는 메시지 응답이 전달 될 것입니다.

![7](/assets/2021-05-06-post-image7.png)

## 7. 배포
---
지금까지의 과정을 실제 카카오톡에서 사용하기 위해 배포를 진행합니다.

![8](/assets/2021-05-06-post-image8.png)

배포 버튼을 클릭하고 카카오톡 챗봇 채널에 접속하면 구현한 내용을 확인할 수 있습니다.

![9](/assets/2021-05-06-post-image9.png)

테스트라는 블록에 진입하여 요일을 입력한 결과, 올바른 스킬 서버의 응답을 확인할 수 있습니다.
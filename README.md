# M5Stick으로 OnOffStick 만들기

## 들어가기 전에...
이 문서는 M5Stick(주의:M5Stack이나 M5Stick-C가 아님)을 사용하기 위하여 개발환경을 구축하고 활용하는 내용을 담고 있습니다. 원래는 간편하게 [UiFlow를 사용](https://docs.m5stack.com/#/en/quick_start/m5stick/m5stick_quick_start_with_uiflow)하려 하였으나 최신 UiFlow(1.3.x)버전을 지원하는 M5Stick용 펌웨어가 없는 관계로(현재 1.2.3이 최고 버전) UiFlow에 접속은 되나 작업은 불가능한 상황입니다. 따라서 아두이노 스케치 기반으로 개발 환경을 구축하여 프로젝트를 진행하였습니다. 

## 배경
요즘 적외선 리모컨을 가지고 조작하는 기기들이 참 많습니다. 제가 교사로 있는 학교도 다름이 아니라서 교실에만 해도 LG천장형 에어콘, 수업을 하기 위한 프로젝터, 소리를 담당하는 사운드바, 최근에 보급된 꽁기청정기 등이 있어 책상 위에는 이들을 제어하기 위한 리모콘이 잔뜩 있습니다. 보기에도 공간을 많이 차지하고, 또 잘 모르는 사람들에게는 프로젝터를 켜기 위해서 어느 리모콘을 써야할지 고민하게 만들겠지요. 그래서 좀 잘 정리해보려고 리모컨함도 만들고 나름 정리해보려 했으나 근본적인 문제는 해결되지 않았습니다. 

![리모컨함 첫번째 버전](https://user-images.githubusercontent.com/1307187/61166372-44714800-a567-11e9-9f97-382d092faf07.png)

![리모컨함 두번째 버전](https://user-images.githubusercontent.com/1307187/61166377-4fc47380-a567-11e9-97ca-ea79258d80b2.png)

또 심지어 프로젝터의 경우는 학교에 여러 회사의 것이 있는데 이게 끄는 버튼의 동작이 많이 다르기까지 합니다. 이걸 어떻게하면 간단하게 해결할 수 없을까? 하는 차에 떠오르는 장면이 있습니다. 바로 닥터 후의 소닉 스크루 드라이버 리모콘이나 해리포터의 지팡이이처럼 누르면 대충 알아서 켜지고 꺼지면 얼마나 좋을까! 자주 사용하는 리모콘의 신호값을 순서대로 다 쏴버리면 어쨌든 켜지고 꺼지지 않을까? 그래서 이 프로젝트를 시작하게 되었습니다.

![전원아 켜져라!](https://user-images.githubusercontent.com/1307187/61166387-85695c80-a567-11e9-8131-3903d7570da9.png)

## 준비물
- M5Stick : M5Stack이나 M5Stick-C가 아닙니다! [[구매](url)] [[상세정보](url)]
- 제어하려는 리모콘의 전원 On, Off 값들 

> 이 값들을 알려면 IR 리시버가 있어야 하는 문제가 있습니다. 아두이노로 따로 구현하거나, M5Stack의 센서를 빌려와서 쓰거나, 인터넷을 뒤져서 기종을 찾아봐야 합니다.

## 시작하기 전에 M5Stick 둘러보기
M5Stick은 M5Stack을 작게 만든 것으로 세부 사항은 아래 링크의 문서를 읽어보기 바랍니다. 일단 기억할 점은 버튼이 1개고(나머지 한개는 전원/리셋), IR 신호를 보낼 수 있으며(받는건 불가) 배터리를 소량 내장하고 있다는 점입니다. 또한 지자기/가속도 센서를 가지고 있어 자기가 어떤 상태(뒤집혔는지, 어느 방향을 향하고 있는지 등)에 있는지 알 수 있습니다. 

> M5Stick 소개(영문) [https://docs.m5stack.com/#/en/core/m5stick](https://docs.m5stack.com/#/en/core/m5stick)

## 설계
 - 버튼A를 오래 누르면 전원을 켜는 IR 신호가 나가게 됩니다. 이때 IR 신호는 등록된 신호를 순서대로 모두 쏴줘서 맞는게 걸리도록 합니다. 신호가 나갈 때 삐삑 소리가 납니다.
 - 버튼A를 짧게 누르면 전원을 끄는 IR 신호가 나가게 됩니다. 위와 방식이 같습니다.
 - 버튼 A를 짧게 눌렀을 때 꺼지는 것으로 한 이유 :  보통 수업 끝내고 나갈 때 빨리 갈 수 있도록 짧은 버튼을 끄는 것으로 하였습니다. 빨리 정리하고 집에 가야죠?

## 필요한 지식
 - 시리얼 모니터로 상태 찍기(디버깅용)
 - LED 불 켜지게 하기
 - 버튼 A를 누르면 동작하기
 - 버튼 A를 길게 누르면 동작하기
 - 전원을 켜는 IR 신호 배열을 만들기
 - 전원을 끄는 IR 신호 배열을 만들기
 - IR 신호를 보내기
 - 동작할 때 비프음 내기(신호 쏠 때마다. 4개 소면 4번 남)
 - 화면에 한글 표시하기

## 개발환경 구축하기
### 드라이버 설치하기
### 펌웨어 초기화 프로그램 다운로드, 펌웨어 초기화하기
### 아두이노 최신버전 설치하기
### 아두이노에 추가 라이브러리 URL 등록하기
### 아두이노에 필수 라이브러리 설치하기
### 예제 돌려보기

## 기능 구현하기
### 켜지면 LED불이 켜지게 하고 시리얼 모니터 활성화하기
### 버튼 짧게 누르기 동작 구현하기
### 버튼 길게 누르기 동작 구현하기
### IR값 보내고 소리나게 하기
### IR 값들의 배열 목록을 만들기
### IR값을 여러개 보내며 각각 소리나게 하기 
### 화면에 프로그램이 켜졌음을 표시 및 사용법 안내하기

## 참고한 글들 정리

## 기타 M5Stick 관련 URL들 
M5Stick 관련 자료가 생각보다 많이 없어서 모아보았습니다. 

- 공식문서 -  M5Stick UIFlow 사용하기 : https://docs.m5stack.com/#/en/quick_start/m5stick/m5stick_quick_start_with_uiflow


팁
한글 나오게
u8g2.setFont(u8g2_font_unifont_t_korean1);
참고문헌
M5Stack 다운로드 페이지
https://m5stack.com/pages/download

M5Stick 상세페이지
https://docs.m5stack.com/#/en/core/m5stick
M5Stack USB 드라이버
https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers
M5STACK에서 UIFlow 접속하는 방법
https://www.wiznetian.com/m5stack%EC%97%90%EC%84%9C-uiflow-%EC%A0%91%EC%86%8D%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/
[UiFlow]온도/습도 그래프 나타내기
https://www.wiznetian.com/uiflow%ec%98%a8%eb%8f%84-%ec%8a%b5%eb%8f%84-%ea%b7%b8%eb%9e%98%ed%94%84-%eb%82%98%ed%83%80%eb%82%b4%ea%b8%b0/
공식문서 - 펌웨어 올리기
https://docs.m5stack.com/#/en/related_documents/how_to_burn_firmware?id=windows

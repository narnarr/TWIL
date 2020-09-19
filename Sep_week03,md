# Django 학습 내용

### 수강한 강의 목록
* 인터넷은 어떻게 동작할까요?
* Command Line 시작하기
* Django 소개
* 나의 첫 번째 장고 프로젝트

+) 강의 자료: <https://tutorial.djangogirls.org/ko/>

## 인터넷은 어떻게 동작할까요?
01_웹브라우저란?
> 유저가 요청한 주소를 받아 그래픽적으로 해석해서 보여주는 것(응답).   

02_서버란?
> 명령을 기다리는 컴퓨터. 늘 켜져있음.   
> 요청이 들어오면 미리 만들어 둔 응답을 내어줌.

## Command Line 시작하기
명령 프롬프트를 통해 보는 것을 CLI(Command Line), 일반 파일창으로 보는 것을 GUI(Graphical User Interface)라고 함.

### 유용한 명령어 정리
* cd [디렉토리명]: 디렉토리 변경. 상위 폴더로 다시 나올 때는 cd ..\
* (pwd): 현재 위치 확인
* dir(ls): 현재 위치의 디렉토리 혹은 파일 목록 보여주기
* copy(cp) [파일 경로]: 파일 복사
* move(mv) [현 위치 변경할 위치]: 파일 이동
* mkdir [디렉토리명]: 새 디렉토리 생성
* del(rm -r): 디렉토리 혹은 파일 제거 (-r을 써야 하위 디렉토리까지 삭제됨)
* tab: 자동완성 기능

## Django 소개
01_Django 장고란?
> 파이썬으로 만들어진 무료 오픈소스 웹 어플리케이션 프레임워크.   
> 웹 개발 시 자주 사용되는 기능들에 필요한 구성요소들을 갖춘 프레임워크로 개발 간접비용의 부담 줄임.

02_장고는 왜 필요할까?
> 편지(request, 요청)가 도착했는지 확인해주는 메일박스(port, 포트)가 있다.   
> 웹 서버는 편지를 받아서 읽고 웹 페이지와 함께 보낼 답장에 들어갈 콘텐츠를 만드는 역할을 함.

03_서버에 요청한 웹사이트가 장고로 전달되면?
* urlresolver: 집배원. URL을 가져와 어떤 함수(view)를 호출할지 규명(URL conf)
* view 함수: 어떤 URL과 연결되어 있는 함수. 다른 프레임워크에서는 controller라고도 부름

## 나의 첫 장고 프로젝트
#### 프로젝트 시작
> 장고의 기본 골격을 만들어주는 스크립트를 실행   
1. venv\Scripts\activate: 가상라이브러리 활성화   
2. django-admin.py: 스크립트로 디렉토리와 파일들 생성
3. django-admin startproject mysite .: 현재 위치(.)에 새로운 프로젝트 파일 추가   
4. settings.py 설정하기
        
        TIME_ZONE = 'Asia/Seoul': 서버 시간대 변경   
        STATIC_ROOT = os.path.join(BASE_DIR, 'static): 정적 파일과 관련된 모든 설정을 한 곳에서 가능하게 함.
        DEBUG = TRUE: 오류 나면 오류 내용 알려줌? 개발 단계에선 TRUE, 실제로 운영할 땐 FALSE.
        
5. python manage.py runserver: 이제 localhost:8000으로 가면 내 서버 작동하는 것 볼 수 있음.
        
        migrate: 내 DB에 변화된 내용을 실제 테이블에 적용해주는 것.
        migrations: DB 스미카의 버전 컨트롤 시스템. 모델에 적용된 변화를 모아서 각 migrations files에 적용. git commit과 유사.
        migrate 더 공부하기: https://velog.io/@matisse/Django-migrations-%EC%A7%91%EC%A4%91-%ED%83%90%EA%B5%AC 
        
+) python manage.py --help: 장고에서 지원하는 sub-command 목록 볼 수 있음

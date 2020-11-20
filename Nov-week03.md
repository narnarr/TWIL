## Nginx란?

동시 접속 처리에 특화된 오픈소스 웹서버 (비동기 방식)

## Gunicorn란?

파이썬의 WSGI(Web Server Gateway Interface) 라이브러리

## djangogirls 강의를 통해 만든 mysite 프로젝트를 배포해보기

튜터님께서 보내주신 자료를 따라해봤는데 한줄한줄 온갖 에러가 나서 계속 붙들다가 결국 포기했다..ㅠㅠ

그래서 <https://inma.tistory.com/125?category=984128](https://inma.tistory.com/125?category=984128>에 나와있는 내용을 따라했는데 nginx 추가하기 전까지는 잘 되다가 그 이후로는 계속 에러가 난다.

## requirements.txt 작성

나는 이미 프로젝트를 다 만들어놓은 상태이므로 터미널에 아래를 입력하면 자동으로 `requirements.txt` 파일이 생성된다.

```bash
pip freeze requirements.txt
```

## Dockerfile 작성

```docker
FROM python:3.7.3

RUN mkdir /code
WORKDIR /code

ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/
```

- 베이스 이미지로 파이썬 3.7.3버전을 사용할 것이고, 컨테이너에 `code` 디렉토리를 만들어 `requirements.txt` 파일을 담고 또 실행(패키지 설치)한다. 그리고 로컬 위치의 모든 파일들을 `code` 디렉토리 하위로 복사한다.

## docker-compose.yml 작성

```docker
version: '3'
services:
  nginx:
    image: library/nginx:latest
    ports: [80:80]
    volumes:
    - .:/code
    - ./config/nginx:/etc/nginx.conf.d
    depends_on:
      - web
  web:
    build:
      context: .
      dockerfile: Dockerfile
    command: gunicorn mysite.wsgi:application --bind 0.0.0.0:8000
    volumes:
    - .:/code
    expose:
    - "8000"
```

- nginx와 web이라는 서비스를 사용할 것이다. 우선 gunicorn으로 장고 서버를 실행한다. 이후 nginx는 도커허브에서 가장 최신 버전의 이미지를 pull 받고 로컬 컴퓨터 포트 80과 컨테이너 포트 80을 연결한다.
- volume: 원래는 컨테이너가 삭제 또는 재시작되면 컨테이너의 변경된 데이터가 함께 삭제된다. 데이터를 영속적으로 보존하기 위해 볼륨 옵션을 사용하여 호스트의 저장소를 마운트한다.
- mount: 저장 장치 등 여러 하드웨어 장치들을 사용하기 위해 운영체제에 인식시키는 것. 즉 물리적인 장치를 특정한 위치(디렉토리)에 연결시키는 과정.

<궁금한 점>

1. `./config/nginx:/etc/nginx.conf.d` 는 자동으로 생성되는 디렉토리인건지 궁금합니다.
2. 다른 곳에서는 전부 `ports: "80:80"` 를 사용하던데 이렇게 쓰고 docker-compose up —build를 할 경우 아래 에러가 뜹니다.

    ```bash
    ERROR: The Compose file '.\docker-compose.yml' is invalid because:
    services.nginx.ports contains an invalid type, it should be an array
    ```

    그래서 `[80:80]` 으로 수정하면 빌드 되고 `localhost:80` 으로 이동하면 nginx 웹서버가 성공적으로 설치 및 시행됐다고 뜹니다. 근데 `localhost:8000` 으로 이동하면 장고로 만들었던 블로그 페이지가 보이지 않고 사이트에 연결할 수 없다고("localhost에서 연결을 거부했습니다") 뜹니다. 이게 블로그 화면을 보려면 최종적으로 `8000` 에 접속하는게 맞는건가요? 아니면 `80` 으로 접속했을 때 블로그 화면이 떴어야 맞는건가요? 뭐가 잘못된 걸까요,,?ㅠㅠ

## nginx.conf 파일 생성

```bash
upstream web {
    ip_hash;
    server web:8000;
}

server {
    location /static/ {
        alias /code/static;
    }

    location / {
    proxy_pass http://web/;
    }

    listen 80;
    server_name localhost;
}
```

클라이언트의 IP를 암호화(hash)한 후 웹서버로 연결한다. static 파일을 응답해야할 경우 컨테이너의 `code/static` 경로로 전송한다. 그 아래는 프록시 설정인데 잘 이해가 가지 않는다. 마지막은 그냥 포트 설정이다.

<궁금한 점>

1. 브라우저로 `http://web/` 에 접속하면 뭐가 떠야 정상인가요? 아니면 프록시 설정에 왜 이렇게 적어둔건가요?
2. 이 파일을 어느 디렉토리에 저장해야하는 건지 모르겠습니다. 인터넷 찾아보면 `config` 디렉토리를 만드는 것까지는 맞는 것 같은데 어떤 사람은 하위에 `nginx` 를 만들어서 그 안에 넣는 것 같기도 하고 다 다르더라고요.. 일단은 아래처럼 해놓았습니다.

## 디렉토리 구조

```bash
djangogirls
┕ venv
┕ blog
	┕ migrations
	┕ static
	┕ templates
	- __init__.py
	- admin.py
	- apps.py
	- forms.py
	- models.py
	- serializers.py
	- tests.py
	- urls.py
	- views.py
┕ mysite
	- __init__.py
	- asgi.py
	- urls.py
	- wsgi.py
┕ config
	┕ nginx
		- nginx.conf
- db.sqlite3
- docker-compose.yml
- Dockerfile
- manage.py
- requirements.txt
```

# Django

### 수강한 강의 목록
- 장고 관리자
- 장고 URLs
- HTML 시작하기

+) 강의 자료: <https://tutorial.djangogirls.org/ko/django_admin/>

## 장고 관리자
#### 01_관리자 페이지에 모델 추가 및 슈퍼사용자 생성
1. `blog/admin.py` 파일을 다음과 같이 수정하여 `blog/models.py`에서 만든 `Post` 장고 모델을
 가져와서 관리자 페이지에서 볼 수 있도록 모델 등록.
    ```python
    from django.contrib import admin
    from .models import Post
    
    admin.site.register(Post)
    ```
2. 슈퍼 사용자(Superuser) 생성
새 터미널 창에서 venv 활성화시킨 후 `python manage.py createsuperuser` 입력

## 장고 URLs
#### 00_View (구글링)
> 조각조각 나뉜 모델과 템플릿들을 연결하는 역할 수행. 어플리케이션의 로직을 넣는 곳.
> 모델/url에서 필요한 정보(request)를 받아와서 로직을 작성하고 템플릿(HTML)에 전달.

#### 01_URL (구글링)
> 장고는 URLconf를 사용해서 URL과 일치하는 뷰를 찾아서 사용자에게 정보를 보여줌.

#### 02_정규표현식
장고는 URL을 뷰에 매칭시키기 위해 `regex`(정규표현식, regular expressions)을 사용.
- ^ : 정규식 시작 기호
- $ : 정규식 종료 기호
- \d : 숫자
- () : 패턴의 부분을 저장할 때
- r : 이스케이프 기호

#### 03_나의 첫 번째 Django url!
- <http://localhost:8000/> 주소를 블로그 홈 페이지로 지정하고 여기에 글 목록 보여줄 것.
- `mysite/urls.py` 파일을 깨끗한 상태로 유지하기 위해 `blog` 애플리케이션에서 메인파일 `mysite/urls.py`로 url들 가져올 것.
- `blog/views.py` (모델 & 템플릿) -> `blog/urls.py` -> `mysite/urls.py`

1. `mysite/urls.py` 수정하기
    ```python
    from django.contrib import admin
    from django.urls import path, include
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('blog.urls')),
    ]
    ```

2. `blog/urls.py` 생성하기
    ```python
    from django.urls import path
    from . import views
    
    urlpatterns = [
        path('', views.post_list, name='post_list'),  
        # views에서 추후에 정의할 def post_list()
    ]
    ```
= 주소가 ' ', 즉 <http://localhost:8000/>로 들어오는 모든 접속 요청을 `blog.urls`로 전송하고 `view.post_list`를 보여주라고 명령.

## 장고 뷰 만들기
1. `blog/views.py`에 view 추가하기
    ```python
   from django.shortcuts import render
   
    def post_list(request):
    return render(request, 'blog/post_list.html', {})
    ```
= 모델의 요청을 받아 `render` 메소드 호출하고 이를 통해 받은 `blog/post_list.html` 템플릿 보여줌.
+) `render`는 장고가 지원해주는 템플릿 시스템. html 등 복잡한 문자열 응답을 파이썬이 종합하긴 힘들기 때문.

## HTML 시작하기
1. `blog/templates/blog`이라는 새로운 디렉토리 생성. (또 blog라는 이름 사용하는 이유는 관습이라서)
2. `post_list.html` 파일 생성

## 장고 ORM과 쿼리셋
#### 01_ORM
> 프로그래밍 언어별로 sql 라이브러리를 만들어 sql없이 데이터베이스에 접근할 수 있도록 한 것

#### 02_QuerySet
> 장고 모델을 통해 데이터베이스에 쿼리를 한 결과물. 전달받은 모델의 객체 목록.
> 쿼리셋은 데이터베이스로부터 데이터를 읽고, 필터를 걸거나 정렬 요청 가능.

1. 장고 Interactive console: `python manage.py shell` 입력.
    - 콘솔창에서 파이썬언어로 장고 파일 사용할 수 있는 환경 세팅해준다. 그냥 `python` 이라고 입력하면 파이썬 언어는 사용 가능하지만 장고 파일에 접근 불가.
2. 모든 객체 조회하기
    - `from blog.models import Post`: 모델 불러오기
    - `Post.objects.all()`: 이게 쿼리셋! 객체 조회하기.
3. 객체(레코드) 생성하기
    - `from django.contrib.auth.models import User`: `auth`는 `django.contrib`에서 기본적으로 지원해주는 앱이고 그 안에 `User`란 모델이 있음.
    - `me = User.objects.get(username='narnarr`: 예전에 superuser로 등록해놨던 인스턴스 갖고오기.
    - `me = User.objects.first()`: 어차피 superuser 하나밖에 안 만들었으니까 이거 사용해도 똑같다.
    ```python
    Post.objects.create(author=me, title='Sample title', text='Test')
    ```
4. 발행하기
    ```python
    post = Post.objects.get(title="Sample title")
   post.publish()
    ```
5. 필터 사용하기
    - `Post.objects.filter(author=me)`: me가 작성한 글들만 보기.
    - `Post.objects.filter(title__contains='title')`: 'title'이라는 단어를 제목에 포함하는 글들만 보기.
    - `Post.objects.filter(title__icontains='title')`: 대소문자 관계없이(ignore) 보기.
    - __lte: less than equal. 기준 시각 이전에 작성한 글들.
    - __gte: great than equal.
    ```python
    from django.utils import timezone
   Post.objects.filter(published_date__lte=timezone.now())
    ```
 6. 정렬하기
    - 필드명만 적으면 필드명에 대해서 오름차순, 앞에 '-' 붙이면 내림차순.
    - `Post.objects.filter(title__contains='title').order_by('created_date')`: 쿼리셋 여러개 연결한 것. chaining.
 
 
### 궁금한 점
1. 왜 `mysite/urls.py` 파일을 깨끗한 상태로 유지해야 하는지?

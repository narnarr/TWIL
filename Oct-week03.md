# Django 끝!

### 수강한 강의 목록
- 템플릿 동적 데이터와 쿼리셋
- 장고 템플릿
- CSS-예쁘게 만들기
- 템플릿 상속받기
- 프로그램 애플리케이션 확장하기
- 장고 폼

## 템플릿 동적 데이터와 쿼리셋
블로그에 작성한 글들에 각각 다른 url을 부여하기 전 우선 발행일자 순서대로 정렬.    
`blog/views.py`에 posts 변수 추가.
```python
from .models import Post
from django.utils import timezone

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {'post_list': posts})
```
`request`는 사용자가 요청하는 모든 것이고, `posts`는 쿼리셋이다. 이 쿼리셋을 원하는 템플릿에 보내기 위해 딕셔너리 타입으로 return의 마지막에 적어준다.

## 장고 템플릿
#### 01_템플릿 태그
> HTML은 정적이고 파이썬은 동적이라 HTML에 바로 파이썬 코드를 넣을 수 없다.    
> 템플릿 태그는 HTML에서 쓸 수 있는 함수들의 목록을 모아놓은 것. 동적인 웹사이트 만들 수 있게 해준다.

#### 02_post목록 템플릿 보여주기
`blog/templates/blog/post_list.html` 수정.    
`{post_list: posts}`에 담긴 객체들을 for문을 이용해 하나씩 띄운다.
```html
{% for post in post_list %}
    <div>
        <p>published: {{ post.published_date }}</p>
        <h1><a href="">{{ post.title }}</a></h1>
        <p>{{ post.text|linebreaksbr }}</p>
    </div>
{% endfor %}
```
행바뀜을 문단으로 변환하는 방식은 아래 두 가지가 있다.
- `|linebreaks`: `<p>`로 나누기
- `|linebreaksbr`: `<br>`로 나누기

## CSS - 예쁘게 만들기
계속 blog.css파일이 html에 적용이 안 돼서 이것저것 해보다가 `settings.py`에 첫 주차 때 알려주신 코드 말고 아래 코드를 넣었더니 드디어 됐다.
```python
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, "static"),
]
```
프로젝트 전반적으로 사용되는 `static` 경로가 어딘지 설정해주는 코드라고 한다.

원래 코드(아래)는 왜 html이 css파일을 인식하지 못했을까..?
```python
STATIC_ROOT = os.path.join(BASE_DIR, 'static)
```
더 찾아보니 `STATICFILES_DIRS`는 프로젝트 전반적으로 사용되는 `static` 파일들의 경로가 어딘지 설정해주는 코드이고,
`STATIC_ROOT`는 배포할 때 `static` 파일들을 하나로 모아야 하는데, 이때 어디로 모을지 경로를 지정해주는 코드라는 것 같다.
결론적으로는 두 코드 모두 `settings.py` 안에 있어야 하는 듯.

## 템플릿 확장하기(상속받기)
1. 상속받는 템플릿 맨 위에 `{% extends 'blog/base.html' %}`
2. 부모 템플릿에서 자식에게 수정 허용할 부분에 `{% block content %}
{% endblock %}`
3. 자식 템플릿에서 변화 일어나는 부분도 `{% block content %}
{% endblock %}`로 감싸기

## 프로그램 어플리케이션 확장하기
1. `blog/templates/blog/post_list.html`에서 포스트 제목에 링크 추가
    ```html
       <h1><a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
    ```
   - `pk`: Primary key. `Post`모델에서 따로 기본키를 지정하지 않았으므로 장고가 자동으로 `pk`필드 추가하여 글 추가할 때마다 번호 생성.
   
2. `blog/views.py`에 아래 뷰함수 추가
    ```python
       from django.shortcuts import render, get_object_or_404
   
       def post_detail(request, pk):
       post = get_object_or_404(Post, pk=pk)
       return render(request, 'blog/post_detail.html', {'post': post})
    ```
   - 2번째 줄에서 왼쪽 pk는 url주소 통해 인자로 받은 pk이고, 오른쪽 pk는 장고 모델에서 각 포스트에 부여한 pk이다. 
   인자로 받은 pk와 같은 pk를 가진 포스트를 찾아서 띄우고, 만약 없을 경우 에러404 표시하라는 것.
   
3. `post_detail`뷰함수와 url 연결시키기 위해 `blog/urls.py` 수정
    ```python
        urlpatterns = [
        path('', views.post_list, name='post_list'),
        path('post/<int:pk>/', views.post_detail, name='post_detail'),
       ]
    ```
    - 정규표현식 `<int:pk>`: 정수(int)가 와야하고, 이를 `pk`라는 변수로 뷰에 전송함을 의미
    즉, http://localhost:8000/post/5를 입력하면 `post_detail`뷰를 통해서 5번째 포스트 글 보여줌

4. `blog/templates/blog/post_detail.html` 생성 후 코드 추가
 
## 장고 폼
> 장고 admin에 로그인 한 유저가 웹사이트에서도 글을 작성할 수 있게 해줌.    
> 올바른 데이터 유형인지 등 유효성 검사 통과하면 데이터베이스에 저장하는 기능 제공.

#### 01_모델 폼 만들기
아래는 따로 데이터 유형 설정할 필요 없이 이미 만들어놓은 모델을 가져와 간편하게 폼을 만드는 모델 폼이다.
`fields`는 튜플도 되고 리스트도 되는데 여튼 해당 모델의 필드명대로만 작성하면 된다.
```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ('title', 'text',)
```
#### 02_URL 추가하기
`blog/urls.py`에 아래 코드 추가
```python
path('post/new', views.post_new, name='post_new'),
```

#### 03_템플릿 만들기
`post_edit.html` 만들고 아래 코드 추가
```html
{% extends 'blog/base.html' %}

{% block content %}
    <h1>New post</h1>
    <form method="POST" class="post-form">{% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="save btn btn-default">Save</button>
    </form>
{% endblock %}
```

#### 04_폼 저장하는 view함수 추가
`blog/views.py`에 아래 코드 추가
```python
from .forms import PostForm
from django.shortcuts import redirect

def post_new(request):
    if request.method == "POST":
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'blog/post_edit.html', {'form': form})
```
1. `post/new` url을 통해 `post_new` 뷰함수가 실행될 때 `post_edit.html`에서 보낸 request를 인자로 받는다.
2. 아직 저장 버튼 안 눌렀으므로 `else`문으로 가고, 비워진 폼을 보여준다.
3. 유저가 내용을 입력하고 저장을 누르면 `POST` 메소드가 발동?된다.
4. 입력한 내용은 자동으로 `request.POST`에 저장? 업데이트? 됐을테고, 이를 `PostForm`폼과 바인딩시킨다.
5. 유저가 입력한 내용이 올바른 데이터 유형인지(int, date 등) 유효성 검사한다.
6. 올바르다면 보통 바로 `post = form.save()`를 하지만 여기서 만든 `PostForm`폼은 작성자와 발행일자를 유저가 기입하는 것이 아니고
현재 로그인 상태인 유저 정보와 현재 시각을 자동으로 작성자와 발행일자로 넘겨주는 형식이므로 이 정보들을 추가한 다음에 저장해야 한다.
7. 저장 완료되면 `post_detail` 뷰함수로 가라고 지정해준다.

#### 05_폼 수정하는 view함수 추가
위에서 한 내용과 거의 비슷하게 반복된다. 새로 추가한 `post_edit` 뷰함수만 보고 나머지 코드는 생략
```python
def post_edit(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    return render(request, 'blog/post_edit.html', {'form': form})
```
1. `post/<int:pk>/edit/` url을 통해 `post_edit` 뷰함수가 실행될 때 `post_edit.html`에서 보낸 request와 몇번째 게시글인지 `pk`를 인자로 받는다.
2. 존재하는 게시글이라면 `Post`모델 중 pk번째 게시글을 `post`로 지정하고 아니라면 에러창을 띄운다.
3. 아직 저장 버튼을 누르지 않았으므로 `else`문으로 간다. 해당 `post`의 내용을 `PostForm`폼에 담아 보여준다.
4. `if request.method == "POST"`: 유저가 수정 후 저장 버튼을 누른다면 `POST`메소드 실행? 발동..? 뭐라고 표현해야할 지 잘 모르겠다.
5. 유저가 수정한 내용(`request.POST`)를 `PostForm`폼과 바인딩시킨다.
5. 폼에 내용이 있다면 유효성 검사를 하고 발행일자와 작성자 정보까지 묶어서 저장한다.
6. 저장 완료되면 `post_detail`뷰함수를 실행한다.

<궁금한 점>
1. `STATIC_ROOT`와 `STATICFILES_DIRS`에 대해 제가 이해한 바가 맞나요?
2. `post_new`와 `post_edit` 뷰함수에 대해 제가 이해한 바가 맞나요?
3. 어떤 기능 추가할 때마다 `views.py`, `blog/urls.py`, `html` 등의 파일을 수정하는데 일반적으로 파일을 수정하는 순서가 있을까요? 수정하면서 코드가 잘 작동하는지 확인하기 좋은 순서 같은거요..!
### 참고

원문: <[https://www.django-rest-framework.org/tutorial/1-serialization/](https://www.django-rest-framework.org/tutorial/1-serialization/)>

튜1 한글: <[https://nachwon.github.io/rest-2/](https://nachwon.github.io/rest-2/)>

튜2 한글: <[https://jisun-rea.tistory.com/entry/Django-REST-framework-2-Requests-and-Responses](https://jisun-rea.tistory.com/entry/Django-REST-framework-2-Requests-and-Responses)>

튜3 한글: <[https://chrisjune.github.io/drf-03-json-res-view](https://chrisjune.github.io/drf-03-json-res-view)>

# Tutorial 1: Serialization

## 01] auto_now VS auto_now_add

CRUD 작업 시 일반적으로 생성일자와 수정일자 정보를 DB 스키마에 만들어 저장한다

수정일자는 전자를(수정할 때마다 갱신), 생성일자는 후자를(최초 저장시에만)

수정일자에도 `auto_now_add`를 사용하면 안될까?

→ `auto_now_add`는 `add` 파라메터와 함께 확인되어, 즉 `insert` 의 경우에만 현재 날짜로 갱신되어 `update` 시 null로 저장된다.

## 02] class Meta

모델에 메타데이터를 추가할 수 있게 해준다

정렬(ordering), DB 테이블 이름(db_table), 이름 유형(verbose_name, verbose_name_plural) 등 지정할 수 있음

## 03] blank vs allow_blank

- `blank` : model fields에 사용.
- `allow_blank` : serializer fields에 사용.

왜 굳이 이렇게 나눈건지는 모르겠다.

## 04] 파라미터 앞 *, **

- `*args`(arguments) : 파라미처 몇개를 받을지 모를 경우 사용. 튜플 형태로 전달됨.
- `**kwargs`(keyword arguments) : 파라미터명을 같이 보낼 수 있음. 딕셔너리 형태로 전달됨.

더 자세한 내용은 <[https://brunch.co.kr/@princox/180](https://brunch.co.kr/@princox/180)>에서.

## 05] Serializer vs ModelSerializer

- `Serializer` : 새로운 필드 추가 가능한 대신 필드 특성 또 다시 설정해야 함. 원하지 않는 필드여도 일단 입력하고 `required=False`를 해줘야 하는 것 같다?
- `ModelSerialize` : 필드명만 입력해도 된다.

튜토리얼에는 `Serializer` 를 사용해서 그대로 따라했더니 명령 프롬프트 python [manage.py](http://manage.py) shell에서 `snippet` 에 저장한 데이터를 serialize하려니 자꾸 "Meta" class가 없다는 AssertionError가 떴다. 다른 데에서 코드 예시를 찾아봐도 `serializers.Serializer` 를 사용하면 클래스 내부에 `Meta` 클래스를 정의하지 않던데 왜 이런 에러가 뜨는지 모르겠다. 그래서 `ModelSerializer` 를 사용하고 새로운 필드를 따로 추가하였다.

```python
class SnippetSerializer(serializers.ModelSerializer):
    id = serializers.IntegerField(read_only=True)
    
    class Meta:
        model = Snippet
        fields = ('id', 'code', 'linenos', 'language', 'style')
		...
```

페이지를 밑으로 내려보니 `ModelSerializer` 를 사용한 방법도 있었다 ㅂㄷㅂㄷ 근데 보니까 `id` 는 새로운 필드가 아니었다.. 맞다 `id` 는 인스턴스 생성시 자동으로 추가되는 항목이었지.. `ModelSerializer`에 extra field 추가하는 방법 찾으려고 엄청 헤맸는데 바보같은 시간이었다.

근데 문제는 여기서 `fields = ['id', 'title', 'code', 'linenos', 'language', 'style']` 모든 필드를 사용하던데 그럼 `required=False` 는 대체 뭘 하는 놈인거지?

```python
class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')
```

## 06] Snippet 객체

1. 객체 생성

    ```python
    snippet = Snippet(code='print("hello, world")\n')
    snippet.save()
    ```

    객체를 생성하고 왜 꼭 `.save()` 를 해야하는지 모르겠다. 자바로 객체 생성했을 때는 첫 줄만 입력하면 바로 됐었는데.. 파이썬은 객체 생성할 때마다 `.save()` 를 해줘야하나?

2. 직렬화

    ```python
    serializer = SnippetSerializer(snippet)
    serializer.data
    # {'id': 10, 'code': 'print("hello, world")\n', 'language': 'python', 'style': 'friendly'}
    ```

    Snippet 모델 객체가 딕셔너리형으로 변환되었다.

3. JSON형으로 변환

    ```python
    content = JSONRenderer().render(serializer.data)
    content
    # b'{"id":10,"code":"print(\\"hello, world\\")\\n","language":"python","style":"friendly"}'
    ```

    앞에 `b` 는 데이터가 바이트형(아무런 인코딩이 되어있지 않은 데이터)임을 뜻한다.

## 07] 역직렬화 Deserialization

일련의 바이트 상태의 JSON형 데이터로부터 데이터 구조를 추출하는 일. 즉 데이터를 장고 모델 객체로 변환하는 작업이다.

클라이언트 쪽에서 프론트엔드를 통해 데이터를 전달해왔을 때 장고가 알아들을 수 있도록 역직렬화가 필요하다.

직렬화와 순서가 반대로 흘러간다.

1. 바이트 상태의 JSON 데이터를 메모리로 불러온다
2. `JSONParser` 를 이용해 딕셔너리로 바꿔준다
3. `SnippetSerializer` 에 딕셔너리 데이터를 집어넣고 `is_valid` 함수를 이용해 데이터 유효성 검사를 시행한다.
4. 유효하다면 `serializer.validated_data` 속성에 저장한다.
5. `save()` 메서드를 호출해서 `Snippet` 인스턴스를 생성한다.

## 08] CSRF란

CSRF란 사이트 간 요청 위조(Cross-site request forgery)라는 웹사이트 취약점 공격 중 하나다.

사용자가 자신의 의지와는 무관하게 공격적 행위(수정, 삭제, 등록 등)를 요청하는 것을 말한다.

장고는 이런 취약점을 막기 위해 CSRF 토큰 방식을 기본적으로 제공한다.

튜토리얼에서는 CSRF 토큰이 없는 클라이언트도 수정 등 할 수 있게 `[views.py](http://views.py)` 에서 `@csrf_exempt` 를 사용한다.

# Tutorial 2: Requests and Responses

## 01] 요청(Request) 객체

HttpRequest → Request: 더 유연하게 request parsing 할 수 있다.

- `[request.POST](http://request.POST)` : form 데이터만 다루며, POST 메서드에서만 사용 가능하다
- `request.data` : 아무 데이터에서나 다룰 수 있고, POST뿐만 아니라 PUT에서도 사용 가능하다

## 02] 응답(Response) 객체

JSONResponse → Response: 렌더링되지 않은 콘텐츠를 클라이언트가 요청한 형태로 렌더링할 수 있다

## 03] API 뷰 감싸기

wrappers는 다음과 같은 기능을 제공한다.

1. Request instances를 뷰에서 받았는지 확인하고, Response에 콘텐츠를 추가해 콘텐츠 협상?을 할 수 있게 해준다
2. 상황에 맞게 405 에러 메시지 반환, ParseError 반환 등.

튜토리얼1에서 `@csrf_exempt` 사용해놓고선 튜토리얼2에선 이를 `@api_view` 로 대체한다. `@api_view` 에 관리자가 아닌 클라이언트도 데이터를 수정할 수 있게 해주는 기능이 포함되어있는건가?

(import csrf 어찌구 전부 지운걸로 보아 일단 보안을 아예 없애고 나중에 authorization & permissions에서 컨트롤하는 것 같다)

## 04] URL에 Format Suffix 더하기

Format suffix는 url 맨 뒤에 .json이나 .api 등을 붙여 원하는 타입으로 렌더링해서 볼 수 있게 해준다.

# Tutorial 3: 클래스 기반 함수 CBV

튜토리얼 1,2에서 작성한 FBV를 CBV로 바꾼다.

FBV를 사용했을 땐 if문이 여러번 반복되어 지저분했는데 CBV를 사용하니 확실히 눈에 더 잘 들어오는 것 같다.

## 01] Mixin을 활용한 뷰 구현(API뷰 상속)

CBV는 상속을 통해 편리하게 확장할 수 있다는 장점이 있지만 다중상속을 할 경우 이름이 같은 메소드가 있을 때 어떤 메소드를 호출해야하는지 모호해지는 문제가 생긴다. 

믹스인은 여러 클래스 중 중복되는, 다른 클래스에서 재사용할 속성이나 메소드를 묶어놓은 모듈이다. 명확한 이름을 정해주었기에 다중상속이 주는 모호함을 피할 수 있어 확장성을 극대화시킨다.

Mixin 사용 전:

```python
class SnippetList(APIView):
    """
    List all snippets, or create a new snippet.
    """
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

Mixin 사용 후:

```python
class SnippetList(mixins.ListModelMixin, mixins.CreateModelMixin, generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

더 나아가서 DRF에서 제공하는 generic CBV를 사용하면 코드를 더 줄일 수 있다:

```python
class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

짱이다.

# Tutorial 4: 인증과 권한 Authentication and Permissions

아래 조건들을 구현하고자 한다.

- 인증받은 사용자만 데이터 생성 가능
- 작성자만 글 수정 및 삭제 가능
- 인증받지 않은 사용자는 읽기만 가능

## 01] Pygments

Snippets는 코드를 보여주는 모델이므로 우리가 사용하는 코드 에디터에서 프로그래밍 언어별로 다른 하이라이트 기능을 제공하듯이 Snippets도 사용자가 입력한 코드가 어느 언어인지 알아야 하고 그에 맞는 하이라이트 기능을 제공해야 한다. 이를 위해 파이썬에서 제공하는pygments라는 패키지를 사용한다.

아래는 여기에서 참고했다: <[https://overiq.com/pygments-tutorial/](https://overiq.com/pygments-tutorial/)>

- `lexer` : 어휘 분석기. 코드를 토큰 단위(identifier, keyword, literal 등)으로 분리한다?고 설명이 되어있는데 그냥 파이썬, 자바 등등 어떤 언어로 작성된건지 알려달라는 것 같다.
- `formatter` :  `lexer` 가 선택한 토큰을 HTML, JpgImage, BBCode 등 원하는 형식으로 보여줄 수 있게 만들어준다. 예를 들어 위에서 파이썬 언어를 선택했으면 HTML에서 파이썬 코드를 보여줄 수 있게 HTML언어로 작성한다든가 이미지로 보여주고 싶으면 jpg 파일로 만드는 것이다.
- `highlight()` : 코드와 lexer, formatter를 하나로 묶어 최종 output으로 반환하는 함수다.
- `style` : 코드의 색깔 스키마다.
- `linenos` : 디폴트값은 `False`. `True` 일 경우 `HtmlFormatter` 가 줄별로 숫자를 매긴 output을 생성한다. 코드 에디터 맨 왼편에 그 숫자를 말하는 거다.

# Tutorial 5: Relationships & Hyperlinked APIs

지금까지 기본키를 이용해 API 내 요소들의 관계를 나타냈지만 하이퍼링크를 이용하면 더 편리하다고 한다. 보통 하이퍼링크라고 하면 웹사이트로 이동할 수 있는 링크를 떠올리지만 REST API에서의 하이퍼링크, 즉 하이퍼텍스트는 유저가 선택적으로 정보의 표현과 제어를 동시에 할 수 있는 방법이라고 한다. 

하이퍼링크 방식을 사용하기 위해 기존의 `ModelSerializer` 대신 `HyperlinkedModelSerializer`를 사용한다.

하이퍼링크된 API를 사용하려면 기존의 URL 패턴들에게 이름(path명)을 지정해야 한다.

```python
path('snippets/', views.SnippetList.as_view(), name='snippet-list')
```

그리고 많은 인스턴스를 반환할 시 pagination을 통해 나눠볼 수 있게 할 수도 있다.

# Tutorial 6: ViewSets & Routers

## 01] ViewSet으로 Refactor하기

DRF 사용하기 이전의 url 패턴들은 RESTful하지 못하다. url에 `/delete` , `/edit` 등 행위가 적혀있기 때문이다(REST 설계 가이드 내용 참조. 동사 말고 명사 사용). 따라서 DRF에서 제공하는 `ViewSet` 이라는 추상 클래스를 이용해 이를 RESTful하게 바꿔준다.

`ViewSet` 은 말그대로 유사한 로직의 클래스 집합으로, `View` 클래스와 거의 비슷하지만 get과 put 메서드는 지원하지 않고 read와 update 메서드만 지원한다는 점에서 다르다.

기존의 `UserList` 와 `UserDetail` 뷰를 `UserViewSet` 으로 합치자.

기존 코드:

```python
class UserList(generics.ListAPIView):
        queryset = User.objects.all()
        serializer_class = UserSerializer

    class UserDetail(generics.RetrieveAPIView):
        queryset = User.objects.all()
        serializer_class = UserSerializer
```

변경 후:

```python
class UserViewSet(viewsets.ReadOnlyModelViewSet):
    """
    This viewset automatically provides `list` and `retrieve` actions.
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

뷰셋을 이용하니 코드가 반복되는 일이 없어졌다. (재사용)

`SnipperList` , `SnipperDetail` , `SnipperHighlight` 뷰를 `SnipperViweSet` 로 합칠 때는 읽기와 쓰기 기능을 지원하기 위해 `ModelViewSet` 클래스를 이용한다. 

(뷰셋과 모델뷰셋에서 지원하는 메서드는 <[http://www.cdrf.co/3.1/rest_framework.viewsets/ViewSet.html](http://www.cdrf.co/3.1/rest_framework.viewsets/ViewSet.html)>에서 쉽게 볼 수 있다)

## 02] ViewSet과 URLs 바인딩

핸들러 메소드는 URLConf를 정의할 때(=urlspatterns에 적을 때?)만 액션에 바인딩된다. 핸들러 메소드가 뭐지? 아래 코드에서 `'get', 'post'` 를 말하는건가? 액션은 그 뒤에 `'list', 'create'` 등을 말하는 것 같다. 여튼 각 뷰셋이 어떤 메소드를 사용하는지 명시적으로 `[urls.py](http://urls.py)` 에 먼저 적고 `urlpatterns` 안에 url 패턴을 적어서 액션과 핸들러 메소드를 바인딩 시키는 것 같다..

```python
snippet_list = SnippetViewSet.as_view({
    'get': 'list',
    'post': 'create'
})
...
urlpatterns = [path('snippets/', snippet_list, name='snippet-list'),]
```

## 03] 라우터 사용하기

기존에는 바로 위의 코드처럼 `as_view()` 를 통해 각 뷰의 request method마다 대응되는 함수를 연결시켜주었다. 하지만 라우터를 사용하면 뷰셋단위로 간결하게 알아서 연결해준다고 한다.

`[urls.py](http://urls.py)` 가 아래처럼 매우 짧게 변한다.

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from snippets import views

# Create a router and register our viewsets with it.
router = DefaultRouter()
router.register(r'snippets', views.SnippetViewSet)
router.register(r'users', views.UserViewSet)

# The API URLs are now determined automatically by the router.
urlpatterns = [
    path('', include(router.urls)),
]
```

<궁금한 점>

1. Tutorial 1 - 05]에서 `required=False` 가 무슨 역할을 하는지 궁금합니다.
2. 파이썬에서 객체를 생성할 때마다 `.save()` 를 해줘야하는 건가요?
3. 핸들러 메소드가 뭔가요? Tutorial 6 - 02]가 잘 이해가 가지 않습니다ㅠ

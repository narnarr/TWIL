참고 사이트
- <https://medium.com/@whj2013123218/django-rest-api%EC%9D%98-%ED%95%84%EC%9A%94%EC%84%B1%EA%B3%BC-%EA%B0%84%EB%8B%A8%ED%95%9C-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95-a95c6dd195fd>
- <https://velog.io/@hwang-eunji/django-views-%ED%95%A8%EC%88%98%ED%98%95-vs-%ED%81%B4%EB%9E%98%EC%8A%A4%ED%98%95-%EC%A0%9C%EB%84%A4%EB%A6%AD>
- <https://blog.naver.com/PostView.nhn?blogId=pjok1122&logNo=221610312964>


## 왜 REST API를 사용하는가?
1. REST API를 사용하면 백엔드와 프론트엔드의 구분이 잘 되어 일의 효율을 높일 수 있다고 한다.
  - 예를 들어 REST API를 사용하지 않으면 이전에 django blog 어플리케이션에서 아래 코드처럼 Django Template filter를 프론트도 익혀야 하는 불편함이 있다는 것.
  ```html
  {% if post.published_date %}
              <div class="date">
                  {{ post.published_date }}
              </div>
          {% endif %}
  ```
2. 또, 코드의 재활용성이 높아진다. 즉 한 HTML&JS 페이지에서 여러 API에서 정보를 받을 수도 있고 여러 웹페이지에서 동일한 API를 호출할 수도 있다는 것.
  - REST API를 사용하지 않으면 웹페이지 당 한 API에서만 정보를 받을 수 있다는 걸 처음 알았다. 반대도 마찬가지.
  - 여튼 비슷한 페이지를 만들 때마다 매번 코드를 반복할 필요 없이 재활용할 수 있어 효율을 높일 수 있다고 한다.

3. 마지막으로, 멀티 플랫폼에서 용이하게 사용 가능해 확장성이 증가하는 장점이 있다.


## Django REST Framework 사용하기
1. settings.py의 `INSTALLED_APPS`에 `'rest_framework',` 추가

2. Serializer 생성: `blog/serializers.py` 생성
  - Serializer는 데이터를 검정하고 원하는 필드를 선택해 JOSN 형식으로 매핑해주는 것이다. 
  ```pyhton
  from rest_framework import serializers
  from .models import Post
  

  class PostSerializer(serializers.ModelSerializer):
      user = UserSerializer(read_only=True)
      class Meta:
          model = Post
          fields = (
              'author',
              'title',
              'text',
              'created_date',
              'published_date',
          )
          read_only_fields = ('created_date',)
  ```
  - `read_only_fields`는 읽기 전용 필드다.
  - `fields = '__all__'` 혹은 `exclude = 'title'` 등 필드를 선택하는 방식에는 여러 개가 있다.
  - `ModelSerializer` 말고도 `Hyperlinked`를 사용해 serialize 할 수 있다고 한다.
  
3. `views.py`에 아래 코드 추가
  - 지금까지 함수형 view를 작성하다가 처음으로 클래스형 View를 추가하게 되었다.
  - 함수형 뷰: 신속한 개발이 가능하지만 로직이 복잡해진다.
  - 클래스형 뷰: 상속이 가능해 코드 재사용이 용이하다. 체계적으로 구성 가능하다. `urls.py`에 `.as_view()` 메소드와 함께 사용된다.
  ```python
  from rest_framework import viewsets, permissions
  from .serializers import PostSerializer

  class PostView(viewsets.ModelViewSet):
      queryset = Post.objects.all()
      serializer_class = PostSerializer
      permission_classes = (permissions.IsAuthenticated,)
      def perform_create(self, serializer):
          serializer.save(user=self.request.user)
  ```
4. `blog/urls.py` 수정
  ```python
  from .views import PostView

  post_list_api = PostView.as_view({
      'post': 'create',
      'get': 'list'
  })

  post_detail_api = PostView.as_view({
      'get': 'retrieve',
      'put': 'update',
      'patch': 'partial_update',
      'delete': 'destroy'
  })

  urlspatterns = [
      ...
      path('post/api', post_list_api, name='post_list_api'),
      path('post/api/<int:pk>/', post_detail_api, name='post_detail_api'),
  ]
  ```
  - 참고한 블로그 중 어떤 곳에서는 아래 코드도 넣던데 없이도 api 화면이 잘 떠서 왜 넣어야 하는지 모르겠다.
  - 알아보니 로그인 로그아웃에 접근할 수 있도록 만들어주는 것이라고 한다. 
  - 원래 `serializers.py`에 `UserSerializer`도 있었는데 왜인지 에러가 나서 나는 이를 빼고 만들었다.
  ```python
  from rest_framework.urlpatterns import format_suffix_patterns

  urlspatters = [
      path('auth/', include('rest_framework.urls', namespace='rest_framework')),
  ]
  ```
      
여튼 이렇게 얻은 JSON 형식을 다른 플랫폼으로 전송..?하는 등 활용이 쉬워져서 이런 작업을 거치는 것 같다.


< 궁금한 점 >
1. REST API를 사용하지 않으면 한 웹페이지는 하나의 API에서만 정보를 받을 수 있고, 반대로도 한 API는 한 웹페이지에만 정보를 제공할 수 있다는게 정말인가요? 구글링을 통해 안 정보라 확실치가 않네요.


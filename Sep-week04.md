# OOP & Django

### 수강한 강의 목록
* 장고 모델

+) 강의 자료: <https://tutorial.djangogirls.org/ko/>

## 객체지향프로그래밍 (구글링)
> 모든 데이터를 객체로 취급하며, 객체의 상태(state)와 행동(behaviour)을 구체화하는 형태의 프로그래밍이다.   
> (즉, 현실 세계의 사물을 데이터화할 때 그 사물의 어떤 특징을 가져올 것인지 정리하는 것.)

#### 01_클래스란?
> 객체를 정의하고 만들어내기 위한 설계도 혹은 툴.   
- 객체의 상태를 나타내는 필드와 객체의 행동을 나타내는 메소드로 구성.
- 필드란 클래스에 포함된 변수를 의미한다
- 메소드란 어떤 특정 작업을 수행하기 위한 명령문의 집합이다.   

<Java 예시>
```java
public class Player {
    public String name;
    public int age;
    public String team;
    public String position;
}
```

#### 02_객체란?
> 실생활에서 우리가 인식할 수 있는 사물로, 소프트웨어 세계에 구현할 대상.
> 클래스에 선언된 모양 그대로 생성된 실체
- 필드에 구체적인 값이 들어가게 된다.
- 클래스의 인스턴스라고 불린다.   

<Java 예시>
```java
public class SoccerGame {
    public static void main(String[] args) {
        Player player1 = new Player(); //Player란 클래스로 player1이란 객체 생성.
        player1.name = "Son Heung-Min";
        ...
        Player player2 = new Player();
        player2.name = "Lionel Messi";
        ...
    }
}
```

#### 03_인스턴스란?
> 설계도를 바탕으로 소프트웨어 세계에 구현된 구체적 실체.
- 인스턴스는 객체에 포함된다고 볼 수 있다.


## 장고 모델
#### 01_객체란?
> 속성(properties)과 행동(methods)을 모아놓은 것.   

* ex) '블로그 게시글'이란 객체가 있다.
    - 속성: 제목, 내용, 작성자, 작성일, 게시일.   
    - 메서드(행동, 기능): 출판하기.    
    
(이건 객체라기보다는 클래스에 대한 내용 같다)

#### 02_장고 모델이란?
> 객체의 특별한 종류이다???? (강의에서 이렇게만 말하고 넘어감)   
> (제가 이해한 바로는 "장고 모델=일반 프로그래밍 언어의 클래스"인데, 클래스가 객체의 상위 개념인데 어떻게 모델이 객체의 특별한 종류라는건지...)
    
    +G) 장고 내장 ORM(Object Relational Mapping)    
        : ORM은 SQL을 직접 작성하지 않아도 장고 모델을 통해 데이터베이스로 접근 가능하게 해줌(조회, 추가, 수정, 삭제)

#### 03_어플리케이션 만들기
```
(venv)~/djangogirls$ python manage.py startapp blog
```
- 앱 생성할 때마다 settings.py의 INSTALLED_APPS에 앱 추가 사실을 알려야 함.

#### 04_블로그 글 모델 만들기
> 모든 `Model` 객체는 `blog\models.py` 파일에 선언하여 모델 만들어야 함.

```python
from django.conf import settings
from django.db import models
from django.utils import timezone


class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(
            default=timezone.now)
    published_date = models.DateTimeField(
            blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title
```
- `Post`: 모델의 이름으로, 클래스는 항상 대문자로 시작.
- `models`: 이 모델(Post)가 장고 모델임을 의미함. 이게 있어야 장고가 Post가 DB에 저장되어야 함을 앎.
-> 모델=/=장고모델? DB에 저장되지 않는 모델도 있나..?
- `self`: 이 메서드가 속해있는 모델을 가리킴?
- `self.save()`에서 `.save()`는 파이썬 내장 메소드인건가?
- `__str__`는 인스턴스 자체를 출력할 때 형식을 지정해주는 함수.    
+G) `__init__`은 인스턴스 생성 시 자동으로 실행되는 함수.


#### 05_DB에 모델을 위한 테이블 만들기
> DB에 Post 모델 추가하기. 장고 모델에 우리가 방금 만든, 몇가지 변화 생겼다는 것을 알려줘야 함

```commandline
python manage.py makemigrations blog
```
실제 데이터베이스에 모델 추가한 것을 반영하기
```commandline
python manage.py migrate blog
```
        
        +G) migrate 관련
        migrate: 내 DB에 변화된 내용을 실제 테이블에 적용해주는 것.
        migrations: DB 스미카의 버전 컨트롤 시스템. 모델에 적용된 변화를 모아서 각 migrations files에 적용. git commit과 유사.
        더 공부하기: https://velog.io/@matisse/Django-migrations-%EC%A7%91%EC%A4%91-%ED%83%90%EA%B5%AC 
     

< 궁금한 점>
1. '장고 모델'(=클래스)이랑 '장고 내장 ORM'(=SQL 작성 없이 DB 접근)이랑 다른건지..? 아니면 같은 말인지?
2. '모델'이랑 '장고 모델'이 차이가 있는 말인지? DB에 저장되지 않는 모델도 있나요?
3. 내가 직접 입력하지 않았는데 `.~()`의 형태를 띈 것은 전부 내장 메소드라고 봐도 되나요?
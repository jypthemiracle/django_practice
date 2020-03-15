# Django 연습을 위한 미션사항

* 신규 프로젝트에 Django REST Framework를 사용하게 되어, 장고를 학습하기로 한다.
* 아래의 요구사항을 하나씩 해결해나가면서 장고 튜토리얼의 학습주기를 반복적으로 빠르게 구현해본다.

### MVT 패턴

* Django에서는 MVC 패턴이라고 부르지 않고, Model-View-Template 이라고 표현한다.
  * Model은 데이터베이스에 저장되는 데이터를 의미한다.
  * Template은 사용자에게 보여지는 UI 부분을 의미한다. (MVC의 View)
  * View는 실질적으로 프로그램 로직이 동작한 결과를 Template에 전달하는 역할을 한다. (MVC의 Controller)
* URLconf를 이용하여 URL을 분석하고, 해당 분석 결과를 통해 담당할 뷰를 결정한다.

## 애플리케이션 설계하기

> 다음의 도서를 참고하였습니다. '파이썬 웹 프로그래밍(개정판)' - 김석훈 저 [링크](http://m.hanbit.co.kr/store/books/book_view.html?p_code=B4329597070)

### 요구사항

* 설문에 해당하는 질문을 보여준 후, 질문에 포함되어 있는 답변 항목에 투표하면 그 결과를 알려주는 웹 화면을 제작한다.

* index.html 에는 최근 실시하고 있는 질문의 리스트를 보여준다.

  ```markdown
  What is your hobby?
  Who do you like best?
  Where do you live?
  ```

* detail.html 에는 하나의 질문에 대해 투표할 수 있도록 답변 항목을 폼으로 보여준다.

  ```
  What is your hobby?
  
  [ ] Reading
  [ ] Soccer
  [ ] Climbing
  ```

* results.html 에는 질문에 따른 투표 결과를 보여준다.

  ```
  * Reading - 3 votes
  * Soccer - 1 vote
  * Climbing - 7 votes
  
  Vote Again?
  ```

* 힌트: DB 테이블을 추출하면 다음과 같이 될 것이다.
  * Question 테이블: 질문을 저장하는 테이블이다.
    * Column: id
      * Type: Integer
      * Restriant: NotNull, PK, AutoIncrement
      * 설명: Primary Key의 역할
    * Column: question_text
      * Type: varchar(200)
      * Restriant: NotNull
      * 설명: 질문 문장
    * Column: pub_date
      * Type: datetime
      * Restriant: NotNull
      * 설명: 질문 생성 시각
  * Choice 테이블: 질문별로 선택용 답변 항목을 저장하는 테이블이다.
    * Column: id
      * Type: integer
      * Restriant: NotNull, PK, AutoIncrement
    * Column: choice_text
      * Type: varchar(200)
      * Restriant: NotNull
    * Column: votes
      * Type: integer
      * Restriant: NotNull
    * Column: question
      * Type: integer
      * Restriant: NotNull, FK(Question.id), Index

## admin 페이지 접속 관련

* superuser 명령을 통해 접속 가능한 슈퍼유저 계정을 만들어야 한다.
* 파이썬 3.7.0 버전과 장고 3.0 이상의 버전을 이용할 경우 admin 페이지 접속 시 서버가 종료되는 버그가 있다.
  * 장고 2.2 버전으로 맞춰주거나, 파이썬 3.7.6 버전 이상을 설치한다.

## 장고의 모델

* models.py는 애플리케이션의 테이블을 선언하는 역할이다.
* 우리는 클래스로 테이블 정의를 변경하는 것이 목표이다.
* sqlite 데이터베이스는 기본 장고 데이터베이스 어댑터이다.
* 모델을 선언할 때 우리는 Django의 내장 클래스인 ```models.Model``` 객체로부터 상속받아 구현한다.
  * 장고의 모델과 관련된 모든 기능이 구현되어 있다.
  * CharField, TextField, DateTimeField 등 데이터를 저장할 수 있는 자료형이 선언되어 있다.
* ```on_delete=models.CASCADE``` 를 보자.
  * cascade는 종속, 직렬, 폭포처럼 흐르다 라는 뜻이 있다.
  * 해당 옵션이 설정된 경우 특정한 객체가 삭제될 때 참조된 모든 객체가 함께 삭제되는 것이다.
  * 즉, 삭제의 여파가 종속되어 직렬로 연결되어 폭포처럼 모두에게 닿는 것이다.
* ```def __str__(self)``` 의 의미
  * ```__str__()``` 메소드는 모델 클래스의 객체의 문자열 표현을 리턴한다.
  * 자바로 치면 ```toString()``` 에 해당하는 역할을 수행한다고 생각해볼 수 있을 것 같다.

```python
# ch3\polls\models.py

from django.db import models

class Question (models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def _str_(self):
        return self.question_text

class Choice (models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def _str_(self):
        return self.choice_text
```

## Admin 사이트에 테이블 반영하기

* models.py 에서 정의한 테이블도 Admin 사이트에 보이도록 등록한다.

```python
from polls.models import Question, Choice
admin.site.register(Question)
```

* models.py 모듈에서 정의한 Question, Choice 클래스를 임포트한다.
* ```admin.site.register()``` 함수를 이용하여 임포트한 클래스를 Admin 사이트에 등록해주면 된다.
* 즉, 테이블을 새로 만들 때에는 models.py와 admin.py 두 개의 파일을 함께 수정해야 한다.

## 데이터베이스 변경사항 반영

* 테이블의 신규 생성, 테이블의 정의 변경 등 데이터베이스에 변경이 필요한 사항이 있으면, 이를 데이터베이스에 실제로 반영해주는 작업을 해야 한다.
* 다음의 명령으로 변경사항을 데이터베이스에 반영한다.

```python
python manage.py makemigrations

# Migrations for 'polls':
#  polls/migrations/0001_initial.py
#   - Create model Question
#   - Create model Choice

python manage.py migrate

# Operations to perform:
#  Apply all migrations: admin, auth, contenttypes, polls, sessions
#  Running migrations:
#  Applying polls.0001_initial... OK
```

* 마이그레이션(migration) 이라는 용어에 대해 학습한다.

  * 장고 1.7 버전부터 추가된 ORM 기능이라고 보아도 무방하다.
  * 테이블 및 필드의 생성, 삭제, 변경 등과 같이 데이터베이스에 대한 변경사항을 알려주는 정보이다.
  * ```makemigrations``` 명령을 실행하면서 ```polls/migrations``` 디렉토리 하위에 마이그레이션 파일이 생성되고, 이 마이그레이션 파일을 이용해 ```migrate``` 명령으로 데이터베이스에 테이블을 만들어준다.
  * 특정 마이그레이션 명령을 마이그레이션 파일을 이용해 롤백할 수도 있다.
  * ```python manage.py sqlmigrate polls 0001``` 명령을 이용하여 장고가 출력하는 SQL 문장을 확인해본다.

  ![views](https://wayhome25.github.io/assets/post-img/django/migration.png)

## View 및 Template 설계하기

* polls 애플리케이션의 요구사항에 따르면 총 3개의 페이지가 필요하다.
* 위의 요구사항을 분석하여 구조를 설계하면 다음과 같다.

![image-20200315145854238](/Users/jypsnewmac/Library/Application Support/typora-user-images/image-20200315145854238.png)

* URL과 뷰는 1:1 관계로 매핑되는데, N:1 관계도 가능하다. 
* 이러한 URL/뷰 매핑을 URLconf라고 하며 urls.py 파일에 저장된다.

### URLconf 코딩하기

* urls.py에 다음과 같이 기계적으로 코딩해주면 된다.

```python
from django.contrib import admin
from django.urls import path
from polls import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', views.index, name='index'),
    path('polls/<int:question_id>', views.detail, name='detail'),
    path('polls/<int:question_id>/results/', views.results, name='results'),
    path('polls/<int:question_id>/vote/', views.vote, name='vote'),
]
```

* path() 함수는 route, view 2개의 필수 인자와 kwargs, name 2개의 선택 인자를 받는다.
  * route: URL 패턴을 표현하는 문자열로, URL 스트링이라고도 부른다.
  * view: URL 스트링이 매칭되면 호출되는 뷰 함수이다. HttpRequest 객체와 URL 스트링에서 추출된 항목이 뷰 함수의 인자로 전달된다.
  * kwargs: URL 스트링에서 추출된 항목 외에 추가적인 인자를 뷰 함수에 전달할 때, 파이썬 딕셔너리 타입으로 인자를 정의한다.
  * name: 각 URL 패턴별로 이름을 붙여준다. 여기서 정해준 이름은 템플릿 파일에서 많이 사용된다.
* 위의 코드 중 일부를 해석해보자.

```python
path('polls', views.index, name='index')
```

* 요청의 URL이 /polls/ 라면 위의 라인이 매칭되고, URL 스트링에서 추출되는 항목이 없으므로 ``` views.index(request)``` 처럼 뷰 함수가 호출된다.
  * request는 HttpRequest 객체이다.

```python
path('polls/<int:question_id>', views.detail, name='detail'),
```

* 요청의 URL이 /polls/3 이라면 위의 라인이 매칭되고, URL 스트링에서 3이 추출되므로 뷰 함수 호출 시 ```views.detail(request, question_id=3)``` 처럼 인자가 대입된다.
* 추가적으로, ```mysite/settings.py``` 파일에 ROOT_URLCONF 항목이 정의되면서 urls.py 를 먼저 분석하기 시작한다.

```python
ROOT_URLCONF = 'mysite.urls'
```

* URLconf를 코딩할 때 앞에서처럼 하나의 urls.py 파이에 작성할 수도 있고, mysite/urls.py와 polls/urls.py 2개의 파일에 작성할 수도 있다. 이는 코드의 재사용을 기본 원칙으로 하는 장고의 장점을 한껏 사용한 것으로 생각해볼 수 있다.

```python
# mysite\urls.py

urlpatterns = [
  path('admin/', admin.site.urls),
  path('polls/', include('polls.urls')) # 해당 URL 요청에 대한 전권을 polls/urls.py에 위임하게 된다.
]
```

```python
# polls\urls.py

app_name = 'polls'
urlpatterns = [
  ... 이하 생략 ...
]
```

## 템플릿 작성

* 템플릿을 먼저 코딩하는 과정을 통해 UI 화면을 생각하면서 로직을 풀어나가도록 한다.

* polls 디렉토리 내부에 templates 디렉토리를 생성한다.

* 장고 템플릿 언어에 익숙해진다.

  * 템플릿 변수는 ```{{variable_name}}```로 구성되어 있다.
  * ```{% if %}``` 와 ```{% for %}``` 태그 등을 활용하여 파이썬의 반복문 문법을 그대로 활용할 수 있다.
  * 템플릿 언어에서 순회가능 객체를 순회하기 위해서는 아래와 같이 for문을 활용한다.

  ```python
  {% for 변수 in 순회가능객체 %}
      {{ 변수 }}
  {% endfor %}
  ```

## View 작성

* index.html을 작성하면서 필요한 변수가 무엇인지 발견했을 것이다.
* 뷰 함수에서 context 변수로 정의해서 템플릿으로 넘겨주면서 위의 템플릿이 비로소 구현될 수 있다.
* 다음과 같이 views.py를 추가한다.

```python
from django.shortcuts import render
from polls.models import Question

def index(request):
    latest_question_list = Question.objects.all().order_by('-pub_date')[:5] # 상위 5개까지만
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

* context 변수에 대해 학습한다.

  * 파이썬의 딕셔너리로 구현된다.
  * ```{myvar1: 101, myvar2: 102}``` 가 있을 때 ```myvar1``` 을 101로, ```myvar2``` 를 102로 변환한다.
  * context는 변수명과 변수값을 매핑하는 데 사용되는 변수이다.

* render 함수에 대해 학습한다.

  ```
  render(request, template_name, context=None, content_type=None, status=None, using=None)
  ```

  * request는 HttpRequest 객체이며, template_name에는 불러오고 싶은 템플릿을 기재해주면 된다.

  * 즉, html 파일을 띄우는 역할을 수행한다고 보면 된다.

  * 템플릿 파일에 context 변수를 적용하여 최종 HTML 텍스트를 만들고, 이를 담아서 HttpResponse 객체를 반환한다.

  * redirect 함수와의 차이점을 비교한다.

    ```
    redirect(to, permanent=False, *args, **kwargs)
    ```

    * ```to``` 에는 어느 URL로 이동할 지 결정하며, 이 때 상대 URL과 절대 URL 모두 가능하다.
    * `urls.py` 에 `name` 을 정의하고 이를 많이 사용한다.
    * 단지 URL로 이동하는 것이기 때문에 render 처럼 context 값을 넘기지는 못한다.

* 장고 프레임워크는 템플릿 디렉토리를 찾기 위하여 프로젝트의 설정 파일인 settings.py에서 경로를 지정해주어야 한다.

```python
TEMPLATES = [
  {
    ...
    'DIRS': [os.path.join(BASE_DIR, 'templates')],
    ...
  }
```

* 위의 코드처럼 지정해주면 템플릿 디렉토리를 검색하여 템플릿 파일을 찾는다.
* TEMPLATES 항목에 정의된 디렉토리를 먼저 찾고, 그 다음에 INSTALLED_APPS 항목에 등록된 각 앱의 templates 디렉토리를 찾는다.

## 폼 템플릿 작성

* 이번 단계의 목표는 3개의 질문 중 하나를 선택했을 때, 질문에 대한 답변 항목을 보여주고 투표하도록 하는 화면을 만드는 것이다.
* detail.html을 생성한 뒤 템플릿 파일에서 새롭게 작업한다.

```HTML
<html>
<title>Django Practice</title>
<h1>Welcome to Django World!</h1>
<h1>
    {{question.question_text}}
</h1>
{% if error_message %}
<p>
    <strong>
            {{error_message}}
        </strong>
</p>
{% endif %}
<form action="{% url 'polls:vote' question.id %" method="post">
    {% csrf_token %} {% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{choice.id}}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br /> {% endfor %}
    <input type="submit" value="Vote " />
</form>

</html>
```

* 주목할 것은 <form action> 부분이다.
  * 일단 라디오 버튼으로 답변 항목을 보여주고 있다. 해당 라디오 버튼을 클릭하면 POST 데이터가 'choice'='3' 형태로 구현되도록 name과 value 속성을 정의하고 있다.
  * forloop.counter 변수는 for 루프를 실행한 횟수를 담고 있는 템플릿 변수이다. <label for> 속성과 <input id> 속성은 값이 같아야 서로 바인딩된다.
  * Vote 버튼을 크릭하면 사용자가 선택한 폼 데이터가 POST 방식으로 polls:vote URL로 전송된다. 전송된 데이터는 vote() 뷰 함수에서 request.POST['choice'] 구문으로 액세스된다.
* Question 객체의 choice_set 속성은 자주 사용되는데, 외래키로 연결된 1:N 관계의 경우 set 속성을 디폴트로 제공한다.
* 즉, question.choice_set.all() 이라고 하면 Question 테이블의 question 레코드에 연결된 Choice 테이블의 레코드 모두를 뜻한다.

## 뷰 함수 detail() 작성하기

* detail() 뷰 함수에서 정의해야 할 컨텍스트 변수는 question 변수 하나면 괜찮다.
* views.py 파일에 아래의 코드를 추가한다.

```python
from django.shortcuts import get_object_or_404, render

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': Question})
```

* 뷰 함수를 정의하는 과정에서 request 객체는 필수 인자이고, 추가적으로 question_id 인자를 더 받는다.
* ```get_object_or_404()``` 단축함수를 사용하고 있다. 이 함수의 첫 번째 인자는 모델 클래스이고, 두 번째 인자부터는 검색 조건을 여러 개 사용할 수 있다.
  * 이 예제에서는 Question 모델 클래스로부터 pk=question_id 검색 조건에 맞는 객체를 조회하고 있다.
  * 조건에 맞는 객체가 없으면 Http404 Exception을 발생시킨다.
* detail() 뷰 함수는 최종적으로 detail.html의 텍스트 데이터를 담은 HttpResponse 객체를 반환한다.

## 뷰 함수 vote() 작성하기

* 사용자가 Vote 버튼을 누르면 vote() 뷰 함수가 호출되고, 수신한 POST 데이터를 처리하는 vote() 뷰 함수를 작성해본다.
* views.py 에 다음 코드를 추가해본다.

```python
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id);
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice']);
    except(KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You did not select a choice.",
        });
    else:
        selected_choice.votes += 1;
        selected_choice.save();
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)));
```

### URL Reverse

* 다음의 [링크](https://github.com/YeongBaeeee/practice/wiki/13-URL-Reverse) 를 참조한다.

* urls.py의 변경 만으로 각 뷰에 대한 URL이 변경되는 유동적인 시스템이다.
* 개발자가 일일이 URL을 계산하지 않아도 되므로 편리하다.
* 다음과 같이 설정할 수 있다.

```python
resolve_url('blog:post_detail', id=10)
```

## 뷰 함수 results() 작성하기

* views.py 에 다음의 코드를 추가한다.

```python
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id);
    return render(request, 'polls/results.html', {'question': question});
```

* results() 뷰 함수는 최종적으로 results.html 템플릿 코드를 렌더링한 결과인 HTML 텍스트 데이터를 담은 HttpResponse 객체를 반환한다.

## 템플릿 파일 results.html 작성하기

* polls\polls\results.html을 생성하고 다음의 코드를 작성한다.

```HTML
<html>
<h1>
    {{question.question_text}}
</h1>
<ul>
    {% for choice in question.choice_set.all %}
    <li>
        {{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}
    </li>
    {{ % endfor % }}
</ul>
<a href="{% url 'polls:detail' question.id %}">Vote again?</a>

</html>
```

* Choice 객체의 의  choice_text를 순서 없는 리스트로 화면에 보여준다(<ul>, <li> 태그 역할)
* 각 텍스트 옆에 투표 카운트(choice.votes)를 숫자로 보여준다.
* votes{{ choice.votes|pluralize }}의 의미는 choice, votes 값에 따라 복수 접미사(s)를 붙여주는 것이다.
* 결과적으로 choice.votes 값에 따라 vote 또는 votes가 표시된다.

### URL 스트링 추출에 대하여

* 뷰 함수와 템플릿 태그 양쪽에서 모두 URL 스트링을 추출할 수 있다.
* 뷰 함수에서는 reverse() 함수를 사용하고, 템플릿에서는 {% url %} 태그를 사용한다.
* 이번 예제에서 보았던 탬플릿 태그와 동일한 URL 스트링을 추출하도록 뷰 함수의 reverse() 함수를 사용해서 표현해보면 다음과 같다.

```python
{% url 'polls:detail' question.id %} // 템플릿에서 사용됨
reverse('polls:detail', args=(question.id,))
```

## Tips

* ```tree``` 명령어를 활용하면 현재 디렉토리의 파일 현황을 트리구조 형태로 보여준다.


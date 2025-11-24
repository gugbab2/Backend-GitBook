# Django, DRF Serializers

#### 참고 링크&#x20;

{% embed url="https://velog.io/@qlgks1/Django-DRF-Serializers-serializer-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-%EC%99%9C-serializer-response%EA%B0%80-%EB%A7%8C%EB%93%A4%EC%96%B4%EC%A7%80%EA%B8%B0-%EA%B9%8C%EC%A7%80" %}

## 1. Serializers 를 왜 사용해야 할까?&#x20;

* Django 는 에초부터 full-stack-framework 이다. FE 를 렌더링 해주기 위해서 template, template-engine 이 있고 우리는 그 MVT(Model-View-Template) pattern 을 따라서 full-stack Django app 을 만들어야 했다.&#x20;
* template-engine 은 java jsp, typeleaf 등과 같은 template-engine 과 크게 다르지 않다.&#x20;
* 이 과정에서 중복되는 header, footer 분리, 그리고 기본적인 보안을 위한 csrf token 등이 있고 또 특정 모델(object) 을 리스트 형태로 또는 단일 모델을 보여줘야 하는 경우가 당연히 필요했다.&#x20;
* Django 의 강력한 ORM 기능을 바탕으로 queryset 을 구성하고 적절한 CRUD 기능이 필요했다. 그래서  View(요청을 받아 응답을 반환하는 호출 가능한 객체) 라는 개념의 컨셉으로 http protocol request 를 method(get, post, patch, put, delete, ...) 에 맞게  대응해 주는 것이 있다.&#x20;
* 대표적으로 아래와 같은 `BaseListView` 가 존재한다.

```python
class BaseListView(MultipleObjectMixin, View):
    """A base view for displaying a list of objects."""

    def get(self, request, *args, **kwargs):
        self.object_list = self.get_queryset()
        allow_empty = self.get_allow_empty()

        if not allow_empty:
            # When pagination is enabled and object_list is a queryset,
            # it's better to do a cheap query than to load the unpaginated
            # queryset in memory.
            if self.get_paginate_by(self.object_list) is not None and hasattr(
                self.object_list, "exists"
            ):
                is_empty = not self.object_list.exists()
            else:
                is_empty = not self.object_list
            if is_empty:
                raise Http404(
                    _("Empty list and “%(class_name)s.allow_empty” is False.")
                    % {
                        "class_name": self.__class__.__name__,
                    }
                )
        context = self.get_context_data()
        return self.render_to_response(context)
```

* 위와 같은 View 를 상속 받아서, 아래와 같이 우리가 원하는 값(attribute) 만 재정의 해주면 아주 쉽게 사용할 수 있다.&#x20;

```python
class ArticleListView(generic.ListView):
    model = Article
    context_object_name: str = "article_list"
    template_name: str = "articleapp/list.html"
    paginate_by: int = 5
```

* 근데 문제가 하나 생긴다. 제일 처음 마주하는 문제는 create action(post request) 을 위해 FE(html) 에서 from tag 로 만들었는데, "특정 값만 받으면 되는 경우" 이다.&#x20;
  * 로그인 이후 게시글 하나 쓸 때 `title`, `content` 는 입력받지만, **작성자(현재 로그인한 사람)와 조회수(초기값 0), 작성일(현재 시간) 같은 데이터는 BE 에서 알아서 채워 넣어야 한다.**
  * 그리고 web 은 전쟁터다. form 값에 대한 1차(FE), 2차(FE) validation 은 필수적이다!
* 이 것을 위해 Django 에서는 Model Form 이 있다.&#x20;
  * 위에서 언급한 과정을 django core 에서 만들어 놓은 좋은 `BaseModelForm` - `ModelForm` class 를 삭속 받아서 어래와 같이 사용한다.&#x20;

```python
class ArticleCreationForm(ModelForm):
    class Meta:
        model = Article
        fields = ["title", "image", "content"]
```

* 굉장히 Django 철학스러운 방법이다. 장고는 많은 부분이 "템플릿 메서드 패턴" 디자인 패턴으로 상위 class 를 만들어 두었다. **하지만 이렇게 full-stack 으로 제공하기 위한 것들이 Django 를 rest-full 한 API 를 만들게 하기 위함으로써 걸림돌이 되었다.**
  * **Django Model Form 은 템플릿 엔진을 위한 사용 방법이다.**&#x20;
* 그럼에도 Django 를 선택하는 이유는 있다.&#x20;
  * 아무래도 빠른 개발이 필요한 상황에 model 에 대한 admin 까지 간편하게 만들어주고,&#x20;
  * rest-full 한 API(DRF 기능을 사용할 경우) 까지 만들 수 있는 최적의 선택은 Django 이었을 것이다.
* 하지만, Django ORM 이랑 결합해서 response 는 json 이라는 data type 이 필요하게 되었고, 기존에 model form 이 가지는 concept 또한 필요하게 되었다. 그렇게 DRF 와 같은 형태의 라이브러리가 필요하게 된다.&#x20;
* 우선 DRF 없이 Django로 rest-full한 api를 만들려면 어떻게 해야 할까?  &#x20;
  * django에는 (django > http) `JsonResponse` 가 이미 존재한다. 하지만 json request는? 어떻게 django ORM으로 casting해서 ORM을 잘 활용할까?
  * 여러 방법들이 있기는 하지만 매우 불편하다;;     &#x20;
* DRF 는 Django base 로 하는 또 다른 web-framework 이다. rest-full concept 에 마지 않은 Django 를 그나마 rest api concept 에 맞춰서 사용할 수 있는 것들을 많이 제공해 준다.&#x20;
  * 그래서 우리는 마치 model-form 을 사용하는 것 처럼 drf 를 사용할 수 있다.&#x20;
  * **full-stack 으로 개발하던 django 관성을 해치치 않고 동시에 rest-full 하게 api 도 만들 수 있으니 최적의 선택이 되는 것이다.**&#x20;

### 단일 Serializers&#x20;

#### Serializer&#x20;

* 가장 흔하게 보게 되고 기본적인 serializers.Serializer 부터 살펴보자. model 예시는 아래 CheckCrn 이라는 model 을 가지고 하겠다.&#x20;

```python
class CheckedCrn(models.Model):
    registration_number = models.CharField(
        max_length=20, unique=True, blank=False, null=False, verbose_name="사업자등록번호")
    is_closed = models.BooleanField(blank=False, null=False, verbose_name="사업자 폐업여부")
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

```

* 이런 필드를 가지는 model 을 get all 하는 api 를 만들기 위해서 serializer 를 만들어보자.&#x20;

```python
class CheckedCrnSerializer(serializers.Serializer):
    registration_number = serializers.CharField(max_length=20)
    is_closed = serializers.BooleanField()
    created_at = serializers.DateTimeField()
    updated_at = serializers.DateTimeField()
```

* 아주 심플하게 구성할 수 있다. 이것을 dataclass, DTO 등과 같이 활용할 수 있다.&#x20;
  * `reggistration_number` 를 그냥 다른 변수 명으로 변경해도 괜찮고(model 형태만 같다면),&#x20;
  * 아니면 또 다른 나만의 필드를 추가할 수 있다. \
    (`fixed_name = serializers.CharField(default="FIX")`) 를 추가해보자. 그러면 response 도 해당 field 가 추가되어서 준다.&#x20;
  * 이렇게 data model 과 같이 자유롭게 활용할 수 있다.&#x20;
* 그리고 Django ORM Model <-> Serializer 가 자유롭다. ORM 이 아니더라도 dict 등을 통해 validation 을 해볼 수 있다.&#x20;
* 그럼 이렇게 만든 serializer로 어떻게 django view에서 활용할 수 있을까?&#x20;
  * django의 view를 통해서 request, response를 json -> model -> ... -> json 시리얼라이징하면 된다.&#x20;
  * 사실 이렇게 직접 할꺼면 drf, Serializer를 활용하는 의미가 없다.&#x20;
  * 그래서 drf 에서는 `GenericAPIView`를 제공하는 것이다. 그 중 `generics.ListAPIView` 를 활용해 아래와 같이 view 를 구성하면 rest-api 구성은 끝난다.

```python
class CheckedCrnListAPIView(generics.ListAPIView):
    """
    - CheckedCrn 모델 GET ALL API
    """
    queryset = CheckedCrn.objects.all().order_by("-id")
    serializer_class = CheckedCrnSerializer
```

* 매우 간단하다. Django 를 사용하는 관성을 헤치지 않고 마치, Django full-stack 개발을 하는 것처럼, model-form 을 활용하는 것 처럼 개발할 수 있는 것이다.&#x20;

#### ModelSerializer&#x20;

* 하지만 일반적인 Serializer 를 쓰다보니까 번거로움이 생긴다. \
  &#xNAN;**"아니 model 에 이미 적혀져 있고, 선언되어 있는 ORM 필드가 있는데 그걸 왜 다시 한번 써야해?!"**
* 사실 위 Serializer 는 ORM 은 위한 것이 아니라, Django model 을 위한 Serializer 는 바로  `ModelSerializer` 이다. 위 `CheckedCrnSerializer` 를 아래와 같이 아주 간단하게 바꿀 수 있다.&#x20;

```python
class CheckedCrnSerializer(serializers.ModelSerializer):
    class Meta:
        model = CheckedCrn
        fields = "__all__"
```

* 아주 간단해졌다. 이게 끝이다! 만약 모든 fields를 response로 주고 싶지 않다면 `fields = "__all__"` 부분을 `exclude = ("id",)` 로 바꿔보자! 이렇게 field control이 간단하다.
* 근데 갑자기 이런 기능이 고려된다. "아 비즈니스로직이랑 상관없이 response 줄 때 그냥 특정 값에 따라 dynamic하게 모두 바꿀 순 없나?"&#x20;
* `이때가 바로 SerializerMethodField` 를 사용할 때다.

#### SerializerMethodField&#x20;

* 위 `CheckedCrnSerializer` 를 그대로 가져와&#x20;
  * `registration_number` 의 길이가 4보다 크면 `is_long` 이라는 필드를 추가해서 "long" 으로 주고 싶다.&#x20;
  * 길이가 4보다 작다면 "short" 으로 주자.&#x20;

```python
class CheckedCrnSerializer(serializers.ModelSerializer):
    is_long = serializers.SerializerMethodField()

    class Meta:
        model = CheckedCrn
        exclude = ("id",)

    def get_is_long(self, instance: CheckedCrn):
        if len(instance.registration_number) > 4:
            return "long"
        else:
            return "short"
```

* 간단하다. 기본 Serializer class 에 `serializers.SerializerMethodField()` 를 추가하고 이름값 답게 `get_[field_name...]` 으로 method 를 만들어서 정의해주자.&#x20;
* 이 방법은 고정 파라미터 값으로 2번째 instance 를 받는데, Serialzer 에 사용되는 model 을 의미한다.&#x20;

### 다중 Serializers, nested Serializers&#x20;

#### 1:N 일 때, 1의 입장에서&#x20;

```python
class Product(models.Model):
    name = models.CharField(
        max_length=20, blank=False, null=False, verbose_name="제품 이름")
    price = models.PositiveIntegerField(blank=False, null=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
class Cart(models.Model):
    uesr = models.ForeignKey(User, on_delete=models.CASCADE, blank=False, null=False)
    product = models.ForeignKey(
        Product, on_delete=models.CASCADE, blank=False, null=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

* 위와 같은 모델이 두 개 있다고 치자.  `Product` 1:N `Cart` 형태의 관계의 model 이다. \
  (참고로 일단 cart 에서 FK 인 Product 에는 `related_name` 을 주지 않았다.)
* 그리고 serializers 를 다음과 같이 구성하고 `ListAPIView` 를 만들어보자.&#x20;

```python
class CartSerializer(serializers.ModelSerializer):
    class Meta:
        model = Cart
        fields = "__all__"


class ProductSerializer(serializers.ModelSerializer):
    cart_set = CartSerializer(many=True, read_only=True)

    class Meta:
        model = Product
        exclude = ("id",)
```

```python
# view
class ProductListAPIView(generics.ListAPIView):
    """
    - Product n Card GET ALL API
    """
    queryset = Product.objects.all().order_by("-id")
    serializer_class = ProductSerializer
```

<figure><img src="../../../../../.gitbook/assets/image (260).png" alt=""><figcaption></figcaption></figure>

* product 입장에서 자신을 참조하고 있는 cart 를 다 가져와서 보여주는 형태가 된다.&#x20;
  * 참고로 앞서 언급한 model 정의에 FK 줄 때, `related_name` 을 주지 않는다면 역참조시에 해당 `model이름_set` 가 default 로 세팅이 된다.&#x20;
  * 고로 웬만하면 `related_name` 을 주는 것이 협업시 혼선을 줄일 수 있다.&#x20;
* 이렇게 N 입장인 cart 도 serializer 를 재사용하여 product 에게 cart\_set 이라는 field 로 넘겨주면 아무런 추가 작업 없이 nested(중첩) 구성이 가능하다.

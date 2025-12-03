# 섹션8. Django REST Framework 을 이용한 API 개발

## DRF Serializer&#x20;

* Python 데이터를 JSON 형식으로 직렬화(serialize) 하거나, 그 반대로 역직렬화(deserialize) 하는 도구&#x20;
* Python Serializer 는 요청(역직렬화), 응답(직렬화) 시 모두 사용된다.
  * 직렬화 : Django 모델 인스턴스를 곧바로 JSON 형태로 변환
  * 역직렬화 : **클라이언트가 보낸 데이터를 빠르게 유효성 검사**하여 사용 가능

<figure><img src="../../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### ShortURLResponseSerializer 정의&#x20;

* 응답에 사용할 필드를 명시적으로 선언&#x20;
* `ModelSerializer` 사용 시, 모델 정의에 맞는 필드를 자동으로 생성

```python
import string
import random

from django.db import models

class ShortURL(models.Model):
    code = models.CharField(max_length=8, unique=True)
    original_url = models.URLField(max_length=200)
    access_count = models.PositiveIntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        app_label = "shortener"
        db_table = "short_url"

    @staticmethod
    def generate_code():
        characters = string.ascii_letters + string.digits
        return "".join(random.choices(characters, k=8))

```

```python
class ShortURLResponseSerializer(serializers.ModelSerializer):
    class Meta:
        model = ShortURL
        fields = '__all__'
```

### ShortURL 목록 조회 : View&#x20;

* `many=True` : `Serializer` 가 여러 개의 객체를 처리하도록 설정

```python
from rest_framework.response import Response
from rest_framework.views import APIView

from shortener.models import ShortURL
from shortener.serializers import ShortURLSerializer

class ShortURLsAPIView(APIView):
    def get(self, request):
        short_urls = ShortURL.objects.all()
        serializer = ShortURLResponseSerializer(short_urls, many=True)
        return Response(data=serializer.data)
```

### ShortURLCreateSerializer 정의&#x20;

* Serializer 를 역할에 따라 분리&#x20;
* 클라이언트로부터 original\_url 만 입력 받고, code 는 서버에서 주입

```python
# shortener/serializerz.py
class ShortURLCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = ShortURL
        fields = ["original_url"]
```

### ShortURL 생성 : View

* 역직렬화 과정에서 데이터 유효성 검사를 위해 `is_valid()` 호출

```python
class ShortURLsAPIView(APIView):
    …
    def post(self, request):
        serializer = ShortURLCreateSerializer(data=request.data)
        if serializer.is_valid():
            # 중복 검사
            while True:
                code = ShortURL.generate_code()
                duplicate = ShortURL.objects.filter(code=code).exists()
                if not duplicate:
                    break

            short_url = serializer.save(code=code)
            return Response(data=ShortURLResponseSerializer(short_url).data, status=status.HTTP_201_CREATED)

```

### ShortURL 삭제 : View&#x20;

* DELETE 요청에 대한 성공 응답은 데이터 없이 204 반환&#x20;

```python
class ShortURLAPIView(APIView):
    def delete(self, request, code):
        short_url = get_object_or_404(ShortURL, code=code)
        short_url.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

## GenericAPIView

### API 개발시 반복적인 패턴

* 기본적인 CRUD API 개발시, 비슷한 형식의 코드가 반복적으로 사용
* 예 : 새로운 데이터 생성&#x20;
  * 1\) 클라이언트가 보낸 데이터 읽기&#x20;
  * 2\) 데이터 유효성 검사&#x20;
  * 3\) 데이터베이스에 데이터 저장&#x20;
  * 4\) 응답 반환&#x20;
* 달라지는 부분&#x20;
  * 어떤 Serializer 를 사용할지, 어떤 모델에 데이터를 저장할지&#x20;

### GenericAPIView&#x20;

* DRF 에서 자주 사용되는 패턴을 처리할 수 있도록 지원하는 Class-based View
  * 데이터 조회, 데이터 필터링, 페이지네이션, 직렬화/역직렬화 등&#x20;
  * 개발자는 API 마다 달라지는 부분에 대해서만 작성한다.&#x20;

```python
# shortener/apis.py
class ShortURLsAPIView(GenericAPIView):
    queryset = ShortURL.objects.all()
    serializer_class = ShortURLResponseSerializer
    
    def get(self, request):
        queryset = self.get_queryset()
        serializer = self.get_serializer(queryset, many=True)
        return Response(data=serializer.data)
```

#### GenericAPIView 주요 메서드&#x20;

1. `get_queryset()`
   1. 사용할 QuerySet 반환(default: `queryset` 속성에 설정된 값)&#x20;
2. `get_object()`
   1. 하나의 객체를 반환(default: `lookup_field` 속성을 사용해 객체 검색)
3. `get_serializer_class()`
   1. 사용할 Serializer Class 반환(default: `serializer_class` 속성에 설정된 값)

#### GenericAPIView + Mixin&#x20;

* `Mixin` : 특정 기능을 다른 클래스에 재사용 가능하게 지원하는 클래스&#x20;
* DRF 는 반복적인 CRUD 기능을 빠르게 구현할 수 있도록 다양한 Mixin 제공

<pre class="language-python"><code class="lang-python"><strong>class ShortURLsAPIView(ListModelMixin, GenericAPIView):
</strong>    queryset = ShortURL.objects.all()
    serializer_class = ShortURLResponseSerializer
    
    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
</code></pre>

* Mixin 을 반드시 사용해야 하는 것은 아니다. 오히려 더 복잡해 질 수 있다. (`CreateModelMixin`)

```python
class ShortURLsAPIView(ListModelMixin, CreateModelMixin, GenericAPIView):
    queryset = ShortURL.objects.all()
    serializer_class = ShortURLResponseSerializer
    
    ...
    
    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

    def perform_create(self, serializer):
        while True:
            code = ShortURL.generate_code()
            if not ShortURL.objects.filter(code=code).exists():
                break
        serializer.save(code=code)

    def create(self, request, *args, **kwargs):
        serializer = ShortURLCreateSerializer(data=request.data)
        if serializer.is_valid():
            self.perform_create(serializer)
            return Response(data=ShortURLResponseSerializer(serializer.instance).data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### Concrete GenericAPIView

* Mixin 이 미리 조합되어 있는 CBV(Class Based View) 도 제공&#x20;

```python
class ShortURLsAPIView(ListCreateAPIView):
    queryset = ShortURL.objects.all()
    serializer_class = ShortURLResponseSerializer
    
    # def get(self, request, *args, **kwargs):
    #    return self.list(request, *args, **kwargs)

    # def post(self, request, *args, **kwargs):
    #    return self.create(request, *args, **kwargs)
    
    def perform_create(self, serializer):
        while True:
            code = ShortURL.generate_code()
            if not ShortURL.objects.filter(code=code).exists():
                break
        serializer.save(code=code)

    def create(self, request, *args, **kwargs):
        serializer = ShortURLCreateSerializer(data=request.data)
        if serializer.is_valid():
            self.perform_create(serializer)
            return Response(data=ShortURLResponseSerializer(serializer.instance).data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### GenericViewSet&#x20;

* GenericAPIView + ViewSet&#x20;
* ViewSet : 여러 API 동작(GET, POST, PUT, DELETE 등) 을 하나의 클래스에 묶어서 쉽게 관리할 수 있게 하는 기능&#x20;

```python
# shortener/apis.py
class ShortURLViewSet(
    ListModelMixin, CreateModelMixin, DestroyModelMixin, GenericViewSet
):
    queryset = ShortURL.objects.all()
    serializer_class = ShortURLResponseSerializer
    lookup_field = "code"

    def perform_create(self, serializer):
        …

    def create(self, request, *args, **kwargs):
        …
```

```python
# shortener/urls.py
router = DefaultRouter()
router.register("api/short-urls", ShortURLViewSet, "short_urls")

urlpatterns = [
    path('', include(router.urls)),
]
```

## DRF View 선택 방법&#x20;

* 개인적으로 너무 높은 추상화는 코드 가독성이 더욱 떨어진다고 생각한다.&#x20;
* 팀과 개인이 사정을 고려해 추상화 수준을 고려해야 한다.

<figure><img src="../../../../../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

# 섹션8. Django REST Framework 을 이용한 API 개발

## DRF Serializer&#x20;

* Python 데이터를 JSON 형식으로 직렬화(serialize) 하거나, 그 반대로 역직렬화(deserialize) 하는 도구&#x20;
* Python Serializer 는 요청(역직렬화), 응답(직렬화) 시 모두 사용된다.&#x20;
  * 직렬화 : Django 모델 인스턴스를 곧바로 JSON 형태로 변환&#x20;
  * 역직렬화 : **클라이언트가 보낸 데이터를 빠르게 유효성 검사**하여 사용 가능

<figure><img src="../../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### ShortURLResponseSerializer 정의&#x20;

* 응답에 사용할 필드를 명시적으로 선언&#x20;
* `ModelSerializer` 사용시 모델 정의에 맞는 필드를 자동으로 생성&#x20;

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


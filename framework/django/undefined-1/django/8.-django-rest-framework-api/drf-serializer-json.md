# \[DRF] Serializer는 도대체 어떻게 데이터를 JSON으로 바꿀까?

#### 참고 링크&#x20;

{% embed url="https://velog.io/@qlgks1/Django-DRF-Serializers-serializer-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0-%EC%99%9C-serializer-response%EA%B0%80-%EB%A7%8C%EB%93%A4%EC%96%B4%EC%A7%80%EA%B8%B0-%EA%B9%8C%EC%A7%80" %}

## 1. 왜 굳이 'Serializer' 를 사용해야 할까?&#x20;

### Django의 태생적 한계 : "나는 HTML 만드는 기계인데?"

Django 는 원래 풀스택 프레임워크로 태어났다. DB 에서 데이터를 꺼내서 HTML(Template) 에 예쁘게 채워 넣어 브라우저에게 주는 것이 주특기였다. 이 역할을 했던 것이 바로 ModelForm(데이터 검증) 과 Template Engine(화면) 이다.&#x20;

**또한 Django 는 전체 설계가 Model 을 중심으로 되어있기 때문에, ModelForm 은 Model 에 정의된 제약들을 통해서 수 많은 검증 로직들을 구현하지 않아도 되었다.**&#x20;

### 시대의 변화 : "이제 JSON 을 주세요!"

하지만 React, Vue 같은 프론트엔드 프레임워크가 등장하면서 서버는 HTML이 아니라 **데이터(JSON)**&#xB9CC; 내려주면 되게 바뀌었다.&#x20;

* 문제 : Django ORM 으로 꺼낸 데이터는 파이썬 객체인데, 이걸 어떻게 JSON 으로 바꾸지?&#x20;
* 해결 : Django 가 원래 쓰던 ModelForm 이랑 비슷하게 만들자. 대신 HTML 말고 JSON 을 만들게 하자.&#x20;

그렇게 탄생한 것이 DRF(Django REST Framework) 이고, 그 핵심 도구가 Serializer(직렬화기) 이다.&#x20;

**Serializer 또한 Django 의 Model 중심 설계의 이점을 가져가 Model 에 정의된 제약들을 통해서 수 많은 검증 로직들을 구현하지 않아도 되었다.**&#x20;

즉, Serializer 는 API 시대를 위한 ModelForm 이라고 이해하면 가장 정확하다.&#x20;

## 2. Serializer 활용하기 : 기초부터 응용까지&#x20;

### 기본 : Serializer&#x20;

가장 날것의 형태이다. DB 모델과 상관 없이 내가 원하는 필드를 직접 정의한다.&#x20;

```python
class CheckedCrnSerializer(serializers.Serializer):
    registration_number = serializers.CharField(max_length=20)
    is_closed = serializers.BooleanField()
```

마치 DTO 나 딕셔너리처럼 데이터를 검증하고 반환하는 역할을 한다.&#x20;

### 핵심 : ModelSerializer

어차피 DB 모델(models.py) 에 필드 다 정의해 놨는데, 왜 또 써야해? 이런 불만을 잠재우기 위해 등장했다.&#x20;

```python
class CheckedCrnSerializer(serializers.ModelSerializer):
    class Meta:
        model = CheckedCrn
        fields = "__all__"  # 모델에 있는 거 다 가져와!
```

Meta 클래스에 모델만 연결해주면 필드 정의를 자동으로 해준다.&#x20;

### 마법의 필드 : SerializerMethodField

DB 에는 없지만, API 줄 때만 계산해서 넣어주고 싶은 값이 있다면?&#x20;

* 예를 들어, 사업자 번호 길이에 따라 "long" 이나 "short" 라는 값을 추가하고 싶다면 이렇게 한다.&#x20;

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

## 3. 관계 다루기 : Nested Serializer

### 1:N 관계 (Product 와 Cart)&#x20;

상품(Product) 정보를 줄 때, 이 상품이 담긴 장바구니(Cart) 정보도 같이 주고 싶다면? Serializer 안에 또 다른 Serializer를 넣으면 된다. (Nested)

* 1:N 관계에서 1의 입장으로 설계해야 한다.&#x20;

```python
class ProductSerializer(serializers.ModelSerializer):
    # Product 안에 Cart 정보를 리스트로 포함
    cart_set = CartSerializer(many=True, read_only=True)
```

### 주의할 점 : "읽기는 쉽지만 쓰기는 어렵다"

Nested Serializer는 **조회(Read)**&#xD560; 때는 정말 편하지만, **생성(Create)이나 수정(Update)**&#xC744; 하려 하면 복잡해진다.&#x20;

* 부모를 만들면서 자식까지 한 번에 저장(`create` 오버라이딩)해야 하는데, 로직이 상당히 까다로진다.&#x20;
* 팁: Serializer에 너무 복잡한 생성 로직을 넣기보다는, View 계층에서 처리하는 것이 정신 건강에 이롭다.

## 4. 심화 : Serializer 의 내부 동작 원리&#x20;

우리가 무심코 쓰는 `Response(serializer.data)` 한 줄 뒤에서는 엄청난 일들이 벌어지고 있다.&#x20;

### 흐름 : Django 에서 DRF 까지&#x20;

1. 요청 도착 : 사용자의 요청이 들어오면 Django 의 View 가 받는다.&#x20;
2. DRF 개입 : DRF 는 `APIView` (또는 `GenericAPIView`) 라는 녀석을 통해 Django 의 기본 View 를 확장했다.&#x20;
3. Mixin 의 조립 : "목록 조회(List)", "생성(Create)" 같은 기능들은 `Mixin`이라는 조각들로 나뉘어 있고, 필요에 따라 조립해서 사용한다.&#x20;

### 결정적 순간 : .data 가 호출될 때&#x20;

Serializer가 생성되고 `serializer.data`를 호출하는 순간, 진짜 변환 작업이 시작됩니다.

1. 생성자(`__init__`): Serializer에 데이터(`instance`)와 설정(`context`)이 담깁니다.
2. **`to_representation` 메서드: 여기가 핵심 공장입니다!**
   * 필드들을 하나씩 순회하며 데이터를 꺼냅니다 (`get_attribute`).
   * 데이터를 JSON에 맞는 형태(문자열, 숫자 등)로 변환합니다.
   * 최종적으로 파이썬의 딕셔너리(Dict) 형태로 만들어 반환합니다.
3. Response: 이 딕셔너리를 `Response` 객체가 받아서 최종 JSON으로 감싸 클라이언트에게 던져줍니다.

## 5. 결론&#x20;

DRF 는 Django 의 "빠른 생산성" 과 "관리자 페이지(Admin) 등 기능" 이라는 장점을 유지하면서도, REST API 를 만들 수 있도록 해주는 고마운 도구이다.&#x20;

물론 내부 구조(Mixin, GenericView, Serializer flow) 를 파고들면 꽤 복잡하게 추상화되어 있지만, 이 구조를 이해하면 단순히 기능 구현을 넘어 "프레임워크가 의도한 코드" 대로 코딩을 할 수 있게 된다.&#x20;

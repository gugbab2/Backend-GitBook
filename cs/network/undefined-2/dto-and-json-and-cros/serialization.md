# 직렬화(Serialization)

### Serialization

> [직렬화](https://ko.wikipedia.org/wiki/%EC%A7%81%EB%A0%AC%ED%99%94)

> [마샬링(컴퓨터 과학)](https://ko.wikipedia.org/wiki/%EB%A7%88%EC%83%AC%EB%A7%81\_\(%EC%BB%B4%ED%93%A8%ED%84%B0\_%EA%B3%BC%ED%95%99\))

객체를 그 자체로 DB에 저장하거나 네트워크로 전송하는 건 불가능. 객체를 복구할 수 있도록 데이터화 하는 게 필요함. 바이너리라면 Byte Stream, 텍스트라면 기계가 파싱 할 수 있고 사람도 읽을 수 있는 형태를 사용. XML, **JSON**, YAML 같은 형식이 인기.\
\-> System.out.println(boy);\
\-> boy 라는 객체를 찍어내었을 때, 내부 정보가 나오는 것이 아닌 주소가 찍힌다.\
\-> 객체의 데이터를 사용하기 위한 직렬화가 필요하다!(역질렬화를 할 수 있도록 직렬화를 해야한다)\
\-> 객체 != 데이터 => 다만 사실상 같기 때문에, equals 메서드를 오버라이딩 하여서 사용하기도 한다.

**직렬화**와 **마샬링**은 거의 같지만, Java에선 마샬링을 특수하게 다룸.

* 직렬화(Serialization) : 역직렬화(Deserialization)를 통해 객체 또는 데이터(DTO)의 복사본을 만들 수 있음.
* 마샬링(Marshalling) : 직렬화와 같거나, 원격 객체(본래 객체를 컨트롤 할 수 있는 리모콘 객체)로 복원할 수 있음.

### JSON (JavaScript Object Notation)

> [JSON](https://en.wikipedia.org/wiki/JSON)

> [JSON 개요](https://www.json.org/json-ko.html)

> [JSON으로 작업하기](https://developer.mozilla.org/ko/docs/Learn/JavaScript/Objects/JSON)

JSON 은 사람이 읽기 쉽고, 기계도 해석 또는 생성하기 쉽다.

보안 문제만 없다면 JavaScript에서 그대로 사용하는 것도 가능하지만, 대부분 JSON.parse(역직렬화)와 JSON.stringify(직렬화)로 안전하게 사용한다.

JavaScript의 object는 기본적으로 key-value 쌍이다\
(심지어 Array도 제한된 key-value + length 관리에 불과함).

Java는 Map이 이와 유사하지만, 스키마(메타데이터의 집합) 관리(객체 내부 정보를 어느정도 가이드 해준다) 및 타입 안전성(타입의 제한을 둔다)을 위해 DTO를 활용한다.

* 생성: DTO (Java 세계, 객체) → 변환기 → JSON 문자열(네트워크)
* 해석: JSON 문자열(네트워크) → 변환기 → DTO (Java 세계, 객체)

Java에선 [Jackson](https://github.com/FasterXML/jackson)이란 도구가 유명하고, Spring Boot에서 Web 의존성을 추가하면 바로 사용할 수 있다.\
(즉, 우리는 딱히 아무 것도 안 해도 된다)

변환할 때 getter 사용. @JsonProperty로 속성 이름(key) 지정 가능. 다른 객체(DTO)를 포함하고 있어도 됨.\
\-> 특별한 경우가 아니라면, getter 이름을 변경해 사용하도록 하자.

```java
public class PostDTO {
    private String id;
    private String title;
    private String content;

    public PostDTO(String id) {
    }

    public PostDTO(String id, String title, String content) {
        this.id = id;
        this.title = title;
        this.content = content;
    }

    @JsonProperty("ID")
    public String getId() {
        return id;
    }
    @JsonProperty("TITLE")
    public String getTitle() {
        return title;
    }
    @JsonProperty("CONTENT")
    public String getContent() {
        return content;
    }
}

```

### JSON 스키마로서의 DTO (class)

DTO는 여러 곳에서 사용될 수 있고, 그 의미는 계속 확대됨. 예를 들어 Tier, 즉 Remote 통신이 아닌 상황인 Layer 사이나 내부 객체를 감춘 공개 인터페이스를 만들 때도 DTO를 활용. “데이터 전송”이기만 하면 딱히 틀리지 않다.

우리가 여기서 쓰는 건 JSON 스키마로서의 DTO. 이게 DTO의 전부라고 생각하지 말고, DTO를 쓰는 다양한 상황을 상상해 보자.\
\-> DTO 변환은 Tier, 애플리케이션 레이어 , 넓은 범위에서 데이터 전송을 한다면 DTO 를 사용할 수 있다.\
\-> 너무 많은 DTO 사용은 매우 비효율 적일 수 있다.

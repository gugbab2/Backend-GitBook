# CROS

F/E 애플리케이션에서 B/E REST API를 사용하는 \
(실제) 상황 사용자가 접근하는 URL: [http://localhost:3000/](http://localhost:3000/) (바뀔 예정)

### Same-Origin Policy

> [CORS](https://github.com/ahastudio/til/blob/main/http/20201205-cors.md)

> [Same-origin policy](https://developer.mozilla.org/ko/docs/Web/Security/Same-origin\_policy)

웹 브라우저가 처리하는 보안 정책(서버에서는 이미 처리 끝내고 결과를 준 상태).

Back-end, 즉 얻으려는 리소스의 출처(호스트)가 Front-end, 즉 현재 페이지(B/E 입장에서는 요청하는 쪽)와 다르면 접근할 수 없게 하는 보안 정책. **출처에는 포트까지 포함된다는 점에 주의**.

**요청 메시지**

```
GET /posts HTTP/1.1
Host: <http://localhost:8080> → Back-end (REST API)
Origin: <http://localhost:3000> → Front-end
(...자세한 헤더는 생략…)
```

### JSONP

> [JSONP](https://ko.wikipedia.org/wiki/JSONP)

\<script> 태그는 동일 출처를 따지지 않는다는 점을 이용. 서버에서 JSON을 직접 전달하는 게 아니라, 실행되는 자바스크립트 코드를 전달하는 방식.

```html
<script>
	window.success = (data) => {
		// 얻은 데이터를 처리하는 코드
		console.log(data);
	}
</script>

<script src="<http://server/posts?callback=success>"></script>
```

서버에서는 JSON이 아니라 JavaScript 코드를 생성.

```jsx
“callback” query parameter로 들어온 콜백 함수명([
	{ id: '1', title: '제목', content: '내용' }
]);
```

이제는 그냥 CORS 쓰자.

#### CORS (Cross-Origin Resource Sharing)

> [CORS](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)

> [Access-Control-Allow-Origin](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Access-Control-Allow-Origin)

Back-end, 즉 REST API의 응답 헤더에 “Access-Control-Allow-Origin” 속성을 포함시키면 됨. 서버 쪽에서 “여기(F/E)에서 요청한 거라면 괜찮아요”라고 알려주는 방식. 요청 헤더의 “Origin” 속성을 참고할 것.

* 웹 페이지 (F/E): [https://ahastudio.com](https://ahastudio.com)
* API 서버 (B/E): [https://api.ahastudio.com](https://api.ahastudio.com)

**응답 메시지**

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: <https://ahastudio.com>
(...자세한 헤더는 생략…)
(빈 줄)
[
	{
		"id": "123",
		"title": "재밌는 이야기",
		"content": "는 다음 기회에…"
	}
]
```

### Spring Web MVC에서 CORS

#### HttpServletResponse

> [HttpServletResponse](https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpServletResponse.html)

```java
@GetMapping
public List<PostDto> list(
	HttpServletResponse response
) {
	response.addHeader("Access-Control-Allow-Origin", "<http://localhost:3000>");
```

Origin과 무관하게 모든 요청을 허용할 거면 그냥 “\*”로 잡아주면 된다.

```java
response.addHeader("Access-Control-Allow-Origin", "*");
```

#### @CrossOrigin

> [CrossOrigin](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html)

Spring Web에선 @CrossOrigin 애너테이션을 써주면 된다. 클래스, 메서드 모두 지정 가능.

```java
@CrossOrigin("<http://localhost:3000>")
```

모든 요청을 허용할 거라면 마찬가지로 “\*”로 잡아주면 된다.

```java
@CrossOrigin("*")
```

아무 것도 안 써도 동일함.

```java
@CrossOrigin
```

#### WebMvcConfigurer

WebMvcConfigurer 인터페이스에 대한 Spring Bean으로 환경 설정.

```java
@Bean
	public WebMvcConfigurer webMvcConfigurer() {
		return new WebMvcConfigurer() {
		
		@Override
		public void addCorsMappings(CorsRegistry registry) {
			registry.addMapping("/**")
							.allowedMethods("GET", "POST", "PATCH", "DELETE", "OPTIONS")
							.allowedOrigins("<http://localhost:3000>");
		}
	};
}
```

마찬가지로 “\*”을 쓰거나 allowedOrigins 메서드를 따로 써주지 않으면 모든 요청을 허용할 수 있다.

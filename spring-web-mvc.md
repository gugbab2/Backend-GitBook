# Spring Web MVC 구현

### @RequestMapping -> 스프링 어노테이션으로 요청 URL 과 매핑 시켜준다.

### 아래 어노테이션은 각각의 요청 메서드와 매핑 시켜준다.&#x20;

\-> 해당 어노테이션을 타고 들어가면 @RequestMapping 이 들어있다.

* ### @GetMapping
* ### @PostMapping
* ### @DeleteMapping

### @PathVariable -> URL 에서 {}안에 들어와 있는 값을 캐스팅 하여 메서드의 변수로 사용

### @RequestBody -> RequestBody 값을 캐스팅하여 메서드의 변수로 사용

### @ExceptionHandler -> 지정한 Exception 을 캐스팅 하여, 에러 메시지를 설정할 수 있다.

### @ResponseStatus -> 응답 코드를 설정할 수 있다.

### \[아래 예제는 Collection Pattern, 스프링 어노테이션을 활용하여 간단한 REST API 구성]

```java
@RestController
//@RequestMapping("/posts")     // 일관된 메서드를 하나로 묶어줄 수 있다.
public class PostController {

    @GetMapping("/posts")
    public String list() {
        return "게시물 목록\n";
    }

    @GetMapping("/posts/{id}")
    public String detail(@PathVariable String id) {
        if (id.equals("404")) {
            throw new PostNotFound();
        }
        return "게시물 상세 " + id;
    }

    @PostMapping("/posts")
    @ResponseStatus(HttpStatus.CREATED)
    public String create(@RequestBody String body) {
        return "게시물 생성: " + body;
    }

    @PatchMapping("/posts/{id}")
    public String update(@PathVariable("id") String id, @RequestBody String body) {
        return "게시물 수정: " + id + " with " + body;
    }

    @DeleteMapping("/posts/{id}")
    public String delete(@PathVariable("id") String id) {
        return "게시물 삭제: " + id;
    }

    @ExceptionHandler(PostNotFound.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public String postNotFound() {
        return "게시물을 찾을 수 없습니다\n";
    }
}
```


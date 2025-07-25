# 검증1 - Validation1

#### 현재 학습의 방향이 API 서버를 만드는 방향으로 가고 있기 때문에, 뷰 템플릿에 관련된 코드 설명은 없을 수 있다.&#x20;

## 1. 검증 요구사항&#x20;

상품 관리 시스템에 새로운 요구사항이 추가되었다.&#x20;

#### 요구 사항 : 검증 로직 추가&#x20;

* 타입 검증
  * 가격, 수량에 문자가 들어가면 검증 오류 처리&#x20;
* 필드 검증
  * 상품명: 필수, 공백X
  * 가격: 1000원 이상, 1백만원 이하
  * 수량: 최대 9999
* 특정 필드의 범위를 넘어서는 검증
  * 가격 \* 수량의 합은 10,000원 이상

**컨트롤러의 중요한 역할중 하나는 HTTP 요청이 정상인지 검증하는 것이다.** 그리고 정상 로직보다 이런 검증 로직을 잘 개발하는 것이 어쩌면 더 어려울 수 있다.

> 참고 - 클라이언트 검증, 서버 검증&#x20;
>
> * 클라이언트 검증은 조작할 수 있으므로 보안에 취약하다.&#x20;
> * 서버만으로 검증하면, 즉각적인 고객 사용성이 부족해진다.&#x20;
> * **둘을 적절히 섞어서 사용하되, 최종적으로 서버 검증은 필수**&#x20;
> * API 방식을 사용하면 API 스펙을 잘 정의해서 검증 오류를 API 응답 결과에 잘 남겨주어야 함&#x20;

## 2. 검증 직접 처리 - 소개&#x20;

#### ~~서버 사이드 랜더링 기준임을 참고하자 (HTTP API 서버 기준이 아니다)~~&#x20;

#### 상품 저장 성공&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-11 12.59.49.png" alt=""><figcaption></figcaption></figure>

사용자가 상품 등록 폼에서 정상 범위의 데이터를 입력하면, 서버에서는 검증 로직이 통과하고, 상품을 저장하고, 상품 상세 화면으로 redirect 한다.&#x20;

#### 상품 저장 실패&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-11 13.00.43.png" alt=""><figcaption></figcaption></figure>

고객이 상품 등록 폼에서 상품을 입력하지 않거나, 가격, 수량 등이 너무 작거나 커서 검증 범위를 넘어서면 서버 금증 로직이 실패해야 한다. 이렇게 검증에 실패한 경우 고객에게 다시 상품 등록 폼을 보여주고, 어떤 값을 잘못 입력했는지 친절하게 알려주어야 한다.&#x20;

## 3. 검증 직접 처리 - 개발&#x20;

### 상품 등록 검증&#x20;

먼저 상품 등록 검증 코드를 작성해보자.&#x20;

#### ValidationItemControllerV1 - addItem() 수정&#x20;

```java
package hello.itemservice.web.validation;

import hello.itemservice.domain.item.Item;
import hello.itemservice.domain.item.ItemRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
@RequestMapping("/validation/v1/items")
@RequiredArgsConstructor
public class ValidationItemControllerV1 {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "validation/v1/items";
    }

    @GetMapping("/{itemId}")
    public String item(@PathVariable long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);
        return "validation/v1/item";
    }

    @GetMapping("/add")
    public String addForm(Model model) {
        model.addAttribute("item", new Item());
        return "validation/v1/addForm";
    }

    @PostMapping("/add")
    public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {

        // 검증 오류 결과 보관
        Map<String, String> errors = new HashMap<>();

        // 검증 로직
        if (!StringUtils.hasText(item.getItemName())) {
            errors.put("itemName", "상품 이름은 필수입니다.");
        }
        if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
            errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
        }
        if(item.getQuantity() == null || item.getQuantity() >= 9999) {
            errors.put("quantity", "수량은 최대 9,999까지 허용합니다.");
        }

        // 특정 필드가 아닌 복합 룰 검증
        if(item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10_000) {
                errors.put("globalError", "가격 * 수량의 합은 10,000 이상이어야 합니다. 현재 값 = " + resultPrice);
            }
        }

        //검증에 실패하면 다시 입력 폼으로
        if (!errors.isEmpty()) {
            model.addAttribute("errors", errors);
            return "validation/v1/addForm";
        }

        // 성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v1/items/{itemId}";
    }

    @GetMapping("/{itemId}/edit")
    public String editForm(@PathVariable Long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);
        return "validation/v1/editForm";
    }

    @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
        itemRepository.update(itemId, item);
        return "redirect:/validation/v1/items/{itemId}";
    }

}
```

#### 검증 오류 보관&#x20;

`Map<String, String> errors = new HashMap<>();`

만약 검증시 오류가 발생하면 어떤 검증에서 오류가 발생했는지 정보를 담아둔다.&#x20;

#### 검증 로직&#x20;

검증시 오류가 발생하면 `errors` 에 담아둔다. 이때 어떤 필드에서 오류가 발생했는지 구분하기 위해 오류가 발생한 필드명을 `key`  로 사용한다. 이후 뷰에서 이 데이터를 사용해서 고객에게 친절한 오류 메시지를 출력할 수 있다.

```java
// 검증 로직
if (!StringUtils.hasText(item.getItemName())) {
    errors.put("itemName", "상품 이름은 필수입니다.");
}
if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
    errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
}
if(item.getQuantity() == null || item.getQuantity() >= 9999) {
    errors.put("quantity", "수량은 최대 9,999까지 허용합니다.");
}
```

#### 특정 필드의 범위를 넘어서는 검증 로직&#x20;

```java
// 특정 필드가 아닌 복합 룰 검증
if(item.getPrice() != null && item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if(resultPrice < 10_000) {
        errors.put("globalError", "가격 * 수량의 합은 10,000 이상이어야 합니다. 현재 값 = " + resultPrice);
    }
}
```

특정 필드를 넘어서는 오류를 처리해야 할 수도 있다. 이떄는 필드 이름을 넣을 수 없으므로 `globalError` 라는 `key` 를 사용한다.&#x20;

#### 검증에 실패하면 다시 입력 폼으로&#x20;

```java
//검증에 실패하면 다시 입력 폼으로
if (!errors.isEmpty()) {
    model.addAttribute("errors", errors);
    return "validation/v1/addForm";
}
```

만약 검증에서 오류 메시지가 하나라도 있으면 오류 메시지를 출력하기 위해 `model` 에 `errors` 를 담고, 입력 폼이 있는 뷰 템플릿으로 보낸다.

#### addForm.html&#x20;

~~뷰 템플릿이 학습의 목적이 아니기에 패스..~~&#x20;

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }

        .field-error {
            border-color: #dc3545;
            color: #dc3545;
        }
    </style>
</head>
<body>

<div class="container">

    <div class="py-5 text-center">
        <h2 th:text="#{page.addItem}">상품 등록</h2>
    </div>

    <form action="item.html" th:action th:object="${item}" method="post">

        <div th:if="${errors?.containsKey('globalError')}">
            <p class="field-error" th:text="${errors['globalError']}">전체 오류 메시지</p>
        </div>

        <div>
            <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
            <input type="text" id="itemName" th:field="*{itemName}" th:class="${errors?.containsKey('itemName')} ? 'form-control field-error' : 'form-control'" class="form-control" placeholder="이름을 입력하세요">
            <div class="field-error" th:if="${errors?.containsKey('itemName')}" th:text="${errors['itemName']}"> 상품명 오류
            </div>
        </div>
        <div>
            <label for="price" th:text="#{label.item.price}">가격</label>
            <input type="text" id="price" th:field="*{price}" th:class="${errors?.containsKey('price')} ? 'form-control field-error' : 'form-control'" class="form-control" placeholder="가격을 입력하세요">
            <div class="field-error" th:if="${errors?.containsKey('price')}" th:text="${errors['price']}">가격 오류
            </div>
        </div>
        <div>
            <label for="quantity" th:text="#{label.item.quantity}">수량</label>
            <input type="text" id="quantity" th:field="*{quantity}" th:class="${errors?.containsKey('quantity')} ? 'form-control field-error' : 'form-control'" class="form-control" placeholder="수량을 입력하세요">
            <div class="field-error" th:if="${errors?.containsKey('quantity')}" th:text="${errors['quantity']}">수량 오류
            </div>
        </div>

        <hr class="my-4">

        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit" th:text="#{button.save}">상품 등록</button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'"
                        th:onclick="|location.href='@{/validation/v2/items}'|"
                        type="button" th:text="#{button.cancel}">취소</button>
            </div>
        </div>

    </form>

</div> <!-- /container -->
</body>
</html>
```

#### 정리&#x20;

* 만약 검증 오류가 발생하면 입력 폼을 다시 보여준다.&#x20;
* 검증 오류들을 고객에게 친절하게 안내해서 다시 입력할 수 있게 한다.&#x20;
* 검증 오류가 발생해도 고객이 입력한 데이터가 유지된다.&#x20;

#### 남은 문제점&#x20;

* 뷰 템플릿에서 중복 처리가 많다. 뭔가 비슷하다.
*   타입 오류 처리가 안된다. `Item` 의 `price`, `quantity` 같은 숫자 필드는 타입이 `Integer` 이므로 문자 타입

    으로 설정하는 것이 불가능하다. 숫자 타입에 문자가 들어오면 오류가 발생한다. 그런데 이러한 오류는 스프링

    MVC에서 컨트롤러에 진입하기도 전에 예외가 발생하기 때문에, 컨트롤러가 호출되지도 않고, 400 예외가 발생

    하면서 오류 페이지를 띄워준다.
*   `Item`의 `price` 에 문자를 입력하는 것 처럼 타입 오류가 발생해도 고객이 입력한 문자를 화면에 남겨야 한다.

    만약 컨트롤러가 호출된다고 가정해도 `Item` 의 `price` 는 `Integer` 이므로 문자를 보관할 수가 없다. 결국 문

    자는 바인딩이 불가능하므로 고객이 입력한 문자가 사라지게 되고, 고객은 본인이 어떤 내용을 입력해서 오류가 발생했는지 이해하기 어렵다.
* 결국 고객이 입력한 값도 어딘가에 별도로 관리가 되어야 한다.

지금부터 스프링이 제공하는 검증 방법을 하나씩 알아보자.&#x20;

## 4. BindingResult1&#x20;

지금부터 스프링이 제공하는 검증 오류 처리 방법을 알아보자. 여기서 핵심은 `BindingResult` 이다. 우선 코드로 확인해보자.&#x20;

#### ValiationItemControllerV2 - addItemV1&#x20;

```java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    // 검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
    }
    if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
        bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
    }
    if(item.getQuantity() == null || item.getQuantity() >= 9999) {
        bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999까지 허용합니다."));
    }

    // 특정 필드가 아닌 복합 룰 검증
    if(item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if(resultPrice < 10_000) {
            bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000 이상이어야 합니다. 현재 값 = " + resultPrice));
        }
    }

    //검증에 실패하면 다시 입력 폼으로
    if (bindingResult.hasErrors()) {
        return "/validation/v2/addForm";
    }

    // 성공 로직
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```

#### 주의&#x20;

`BindingResult bindingResult` 파라미터의 위치는 `@ModelAttribute Item item` 다음에 와야 한다.

#### 필드 오류 - FieldError&#x20;

```java
if (!StringUtils.hasText(item.getItemName())) {
    bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
}
```

#### FieldError 생성자 요약&#x20;

```java
public FieldError(String objectName, String field, String defaultMessage) {}
```

필드에 오류가 있으면 `FieldError` 객체를 생성해서 `bindingResult` 에 담아두면 된다.

* `objectName` : `@ModelAttribute` 이름
* `field` : 오류가 발생한 필드 이름
* `defaultMessage` : 오류 기본 메시지

#### 글로벌 오류 - ObjectError

```java
bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000 이상이어야 합니다. 현재 값 = " + resultPrice));
```

#### ObjectError 생성자 요약

```java
public ObjectError(String objectName, String defaultMessage) {}
```

특정 필드를 넘어서는 오류가 있으면 `ObjectError` 객체를 생성해서 `bindingResult` 에 담아두면 된다.

* `objectName` : `@ModelAttribute` 의 이름
* `defaultMessage` : 오류 기본 메시지

#### validation/v2/addForm.html 수정

~~뷰 템플릿이 학습의 목적이 아니기에 패스..~~&#x20;

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }

        .field-error {
            border-color: #dc3545;
            color: #dc3545;
        }
    </style>
</head>
<body>

<div class="container">

    <div class="py-5 text-center">
        <h2 th:text="#{page.addItem}">상품 등록</h2>
    </div>

    <form action="item.html" th:action th:object="${item}" method="post">
        <div th:if="${#fields.hasGlobalErrors()}">
            <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">글로벌 오류 메시지</p>
        </div>
        <div>
            <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
            <input type="text" id="itemName" th:field="*{itemName}" th:errorclass="field-error" class="form-control" placeholder="이름을 입력하세요">
            <div class="field-error" th:errors="*{itemName}">
                상품명 오류
            </div>
        </div>
        <div>
            <label for="price" th:text="#{label.item.price}">가격</label>
            <input type="text" id="price" th:field="*{price}" th:errorclass="field-error" class="form-control" placeholder="가격을 입력하세요">
            <div class="field-error" th:errors="*{price}">
                가격 오류
            </div>
        </div>
        <div>
            <label for="quantity" th:text="#{label.item.quantity}">수량</label>
            <input type="text" id="quantity" th:field="*{quantity}" th:errorclass="field-error" class="form-control" placeholder="수량을 입력하세요">
            <div class="field-error" th:errors="*{quantity}">
                수량 오류
            </div>
        </div>

        <hr class="my-4">

        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit" th:text="#{button.save}">상품 등록</button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'"
                        th:onclick="|location.href='@{/validation/v2/items}'|"
                        type="button" th:text="#{button.cancel}">취소</button>
            </div>
        </div>

    </form>

</div> <!-- /container -->
</body>
</html>
```

## 5. BindingResult2&#x20;

* 스프링이 제공하는 검증 오류를 보관하는 객체이다. 검증 오류가 발생하면 여기에 보관하면 된다.
* `BindingResult` 가 있으면 `@ModelAttribute` 에 데이터 바인딩 시 오류가 발생해도 컨트롤러가 호출된다!

**예) @ModelAttribute에 바인딩 시 타입 오류가 발생하면?**

* `BindingResult` 가 없으면 -> 400 오류가 발생하면서 컨트롤러가 호출되지 않고, 오류 페이지로 이동한다.
* `BindingResult` 가 있으면 -> 오류 정보(`FieldError` )를 `BindingResult` 에 담아서 컨트롤러를 정상 호출한다.

**예) @RequestBody에 바인딩 시 타입 오류가 발생하면?**&#x20;

* `BindingResult` 가 없으면 -> `@RequestBody`를 통해 객체로 변환하는 `HttpMessageConverter` 단계에서 타입 오류가 발생하면, `ItemSaveForm`과 같은 객체를 만들지 못하기 때문에, **컨트롤러 자체가 호출되지 않고 그 전에 `HttpMessageNotReadableException`과 같은 예외가 발생한다.**&#x20;
* `BindingResult` 가 있으면 -> `HttpMessageConverter` 단계에서 타입 오류가 발생하면, **`@RequestBody`의 `BindingResult` 유무와 관계없이 동일하게 `HttpMessageNotReadableException`과 같은 예외가 발생하고 컨트롤러는 호출되지 않는다.**

#### **BindingResult에 검증 오류를 적용하는 3가지 방법**

*   `@ModelAttribute`의 객체에 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 `FieldError` 생성해서 `BindingResult` 에 넣어준다.

    (`@RequestBody` 의 경우 컨트롤러를 호출하기 이전에 튕겨버린다)
* 컨트롤러 내부에서 개발자가 직접 넣어준다.
* `Validator` 사용 이것은 뒤에서 설명

#### BindingResult 와 Errors&#x20;

* `org.springframework.validation.Errors`
* `org.springframework.validation.BindingResult`&#x20;

`BindingResult` 는 인터페이스이고, `Errors` 인터페이스를 상속받고 있다.\
실제 넘어오는 구현체는 `BeanPropertyBindingResult` 라는 것인데, 둘다 구현하고 있으므로 `BindingResult` \
대신에 `Errors` 를 사용해도 된다.&#x20;

`Errors` 인터페이스는 단순한 오류 저장과 조회 기능을 제공한다.\
`BindingResult` 는 여기에 더해서 추가적인 기능들을 제공한다. `addError()` 도 `BindingResult` 가 제공하므로 여기서는 `BindingResult` 를 사용하자. 주로 관례상 `BindingResult` 를 많이 사용한다.

#### 정리&#x20;

`BindingResult` , `FieldError` , `ObjectError` 를 사용해서 오류 메시지를 처리하는 방법을 알아보았다.\
그런데 오류가 발생하는 경우 고객이 입력한 내용이 모두 사라진다. 이 문제를 해결해보자.

## 6. FieldError, ObjectError

#### 목표&#x20;

* 사용자 입력 오류 메시지가 화면에 남도록 하자.
  * 예) 가격을 1000원 미만으로 설정시 입력한 값이 남아있어야 한다.
* `FieldError`, `ObjectError`에 대해서 더 자세히 알아보자.

#### ValidationItemControllerV2 - addItemV2

```java
@PostMapping("/add")
public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    // 검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수입니다."));
    }
    if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
        bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, null, null, "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
    }
    if(item.getQuantity() == null || item.getQuantity() >= 9999) {
        bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, null, null, "수량은 최대 9,999까지 허용합니다."));
    }

    // 특정 필드가 아닌 복합 룰 검증
    if(item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if(resultPrice < 10_000) {
            bindingResult.addError(new ObjectError("item", null, null, "가격 * 수량의 합은 10,000 이상이어야 합니다. 현재 값 = " + resultPrice));
        }
    }

    //검증에 실패하면 다시 입력 폼으로
    if (bindingResult.hasErrors()) {
        return "/validation/v2/addForm";
    }

    // 성공 로직
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```

#### FieldError 생성자

```java
public FieldError(String objectName, String field, String defaultMessage);
public FieldError(String objectName, String field, @Nullable Object
rejectedValue, boolean bindingFailure, @Nullable String[] codes, 
@Nullable Object[] arguments, @Nullable String defaultMessage)
```

파라미터 목록&#x20;

* `objectName` : 오류가 발생한 객체 이름
* `field` : 오류 필드
* `rejectedValue` : 사용자가 입력한 값(거절된 값)
* `bindingFailure` : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
* `codes` : 메시지 코드
* `arguments` : 메시지에서 사용하는 인자
* `defaultMessage` : 기본 오류 메시지

#### ObjectError 생성자&#x20;

`ObjectError` 도 유사하게 두 가지 생성자를 제공한다.&#x20;

#### 오류 발생시 사용자 입력 값 유지&#x20;

사용자의 입력 데이터가 컨트롤러의 `@ModelAttribute` 에 바인딩되는 시점에 오류가 발생하면 모델 객체에 사용자 입력 값을 유지하기 어렵다. 예를 들어, 가격에 숫자가 아닌 문자가 입력된다면 가격은 `Integer` 타입이므로 문자를 보관할 수 있는 방법이 없다. 그래서 오류가 발생한 경우 사용자 입력 값을 보관하는 별도의 방법이 필요하다. 그리고 이렇게 보관한 사용자 입력 값을 검증 오류 발생시 화면에 다시 출력하면 된다.

**`FieldError` 는 오류 발생시 사용자 입력 값을 저장하는 기능을 제공한다.**

여기서 `rejectedValue` 가 바로 오류 발생시 사용자 입력 값을 저장하는 필드다.\
`bindingFailure` 는 타입 오류 같은 바인딩이 실패했는지 여부를 적어주면 된다. 여기서는 바인딩이 실패한 것은 아니기 때문에 `false` 를 사용한다.

#### 스프링의 바인딩 오류 처리&#x20;

타입 오류로 바인딩에 실패하면 스프링은 `FieldError` 를 생성하면서 사용자가 입력한 값을 넣어둔다. 그리고 해당 오류를 `BindingResult`에 담아서 컨트롤러를 호출한다. 따라서 타입 오류 같은 바인딩 실패시에도 사용자의 오류 메시지를 정상 출력할 수 있다.

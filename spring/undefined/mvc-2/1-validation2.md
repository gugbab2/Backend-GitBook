# 검증1 - Validation2

## 7. 오류 코드와 메시지 처리1&#x20;

오류 메시지를 체계적으로 다뤄보자.&#x20;

#### FieldError 생성자&#x20;

```java
public FieldError(String objectName, String field, String defaultMessage);
public FieldError(String objectName, String field, @Nullable Object
rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable
Object[] arguments, @Nullable String defaultMessage)
```

파라미터 목록&#x20;

* `objectName` : 오류가 발생한 객체 이름
* `field` : 오류 필드
* `rejectedValue` : 사용자가 입력한 값(거절된 값)
* `bindingFailure` : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
* `codes` : 메시지 코드
* `arguments` : 메시지에서 사용하는 인자
* `defaultMessage` : 기본 오류 메시지

`FieldError` , `ObjectError` 의 생성자는 `codes` , `arguments` 를 제공한다. 이것은 오류 발생시 오류 코드로 메시지를 찾기 위해 사용된다.&#x20;

#### errors 메시지 파일 생성&#x20;

먼저 스프링 부트가 해당 메시지 파일을 인식할 수 있게 다음 설정을 추가한다.

`application.properties`

```properties
spring.messages.basename=errors
```

`errors` 메시지 파일을 생성하자.&#x20;

`src/main/resources/errors.properties`

```properties
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
```

이제 `errors` 에 등록한 메시지를 사용하도록 코드를 변경하자.&#x20;

**ValidationItemControllerV2 - addItemV3() 추가**

```java
@PostMapping("/add")
public String addItemV3(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    // 검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, new String[]{"required.item.itemName"}, null, null));
    }
    if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
        bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"}, new Object[]{1_000, 1_000_000}, null));
    }
    if(item.getQuantity() == null || item.getQuantity() >= 9999) {
        bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, new String[]{"max.item.quantity"}, new Object[]{9_999}, null));
    }

    // 특정 필드가 아닌 복합 룰 검증
    if(item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if(resultPrice < 10_000) {
            bindingResult.addError(new ObjectError("item", new String[]{"totalPriceMin"}, new Object[]{10_000, resultPrice}, null));
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

```java
bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, 
                       new String[]{"required.item.itemName"}, null, null));
```

* `codes` : `required.item.itemName` 를 사용해서 메시지 코드를 지정한다. 메시지 코드는 하나가 아니라 배열로 여러 값을 전달할 수 있는데, 순서대로 매칭해서 처음 매칭되는 메시지가 사용된다.&#x20;
* `arguments` : `Object[]{1000, 1000000}` 를 사용해서 코드의 `{0}`, `{1}` 로 치환할 값을 전달한다.

## 8. 오류 코드와 메시지 처리2&#x20;

목표&#x20;

* `FieldError` , `ObjectError` 는 다루기 너무 번거롭다.
* 오류 코드도 좀 더 자동화 할 수 있지 않을까? 예) `item.itemName` 처럼?

컨트롤러에서 `BindingResult` 는 검증해야 할 객체인 `target` 바로 다음에 온다. 따라서 `BindingResult` 는 이미 본인이 검증해야 할 객체인 `target` 을 알고 있다.

다음을 컨트롤러에서 실행해보자.

```java
log.info("objectName={}", bindingResult.getObjectName());
log.info("target={}", bindingResult.getTarget());
```

#### 출력 결과&#x20;

```
objectName=item //@ModelAttribute name
target=Item(id=null, itemName=상품, price=100, quantity=1234)
```

### rejectValue(), reject()&#x20;

`BindingResult` 가 제공하는 `rejectValue()`, `reject()` 를 사용하면 `FieldError`, `ObjectError` 를 직접 생성하지 않고, 깔끔하게 검증 오류를 다룰 수 있다.&#x20;

`rejectValue()` , `reject()` 를 사용해서 기존 코드를 단순화해보자.

**ValidationItemControllerV2 - addItemV4() 추가**

```java
@PostMapping("/add")
public String addItemV4(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    log.info("Object={}", bindingResult.getObjectName());
    log.info("Target={}", bindingResult.getTarget());

    // 검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.rejectValue("itemName", "require", new Object[]{item.getItemName()}, null);
    }
    if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
        bindingResult.rejectValue("price", "range", new Object[]{1_000, 1_000_000}, null);
    }
    if(item.getQuantity() == null || item.getQuantity() >= 9999) {
        bindingResult.rejectValue("quantity", "max", new Object[]{9_999}, null);
    }

    // 특정 필  드가 아닌 복합 룰 검증
    if(item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if(resultPrice < 10_000) {
            bindingResult.reject("totalPriceMin", new Object[]{10_000, resultPrice}, null);
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

#### rejectValue()&#x20;

```java
void rejectValue(@Nullable String field, String errorCode,
    @Nullable Object[] errorArgs, @Nullable String defaultMessage);
```

* `field` : 오류 필드명
*   `errorCode` : 오류 코드(이 오류 코드는 메시지에 등록된 코드가 아니다. 뒤에서 설명할 messageResolver를

    위한 오류 코드이다.)
* `errorArgs` : 오류 메시지에서 `{0}` 을 치환하기 위한 값
* `defaultMessage` : 오류 메시지를 찾을 수 없을 때 사용하는 기본 메시지

```java
bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null)
```

앞에서 `BindingResult` 는 어떤 객체를 대상으로 검증하는지 target을 이미 알고 있다고 했다. 따라서 target(item)에대한 정보는 없어도 된다. 오류 필드명은 동일하게 `price` 를 사용했다.

**축약된 오류 코드**

`FieldError()` 를 직접 다룰 때는 오류 코드를 `range.item.price` 와 같이 모두 입력했다. 그런데`rejectValue()` 를 사용하고 부터는 오류 코드를 `range` 로 간단하게 입력했다. 그래도 오류 메시지를 잘 찾아서 출력한다. 무언가 규칙이 있는 것 처럼 보인다. 이 부분을 이해하려면 `MessageCodesResolver` 를 이해해야 한다. 왜 이런식으로 오류 코드를 구성하는지 바로 다음에 자세히 알아보자.

## 9. 오류 코드와 메시지 처리3&#x20;

오류 코드를 만들 때 다음과 같이 자세히 만들 수도 있고, \
`required.item.itemName` : 상품 이름은 필수 입니다.\
`range.item.price` : 상품의 가격 범위 오류 입니다.

또는 다음과 같이 단순하게 만들 수도 있다. \
`required` : 필수 값 입니다.\
`range` : 범위 오류 입니다.

단순하게 만들면 범용성이 좋아서 여러곳에서 사용할 수 있지만, 메시지를 세밀하게 작성하기 어렵다. 반대로 너무 자세하게 만들면 범용성이 떨어진다. 가장 좋은 방법은 범용성으로 사용하다가, 세밀하게 작성되어야 하는 경우에 세밀한 내용이 적용되도록 메시지에 단계를 두는 방법이다.&#x20;

예를 들어서 `required` 라고 오류 코드를 사용한다고 가정해보자.\
다음과 같이 `required` 라는 메시지만 있으면 이 메시지를 선택해서 사용하는 것이다.

```
required: 필수 값 입니다.
```

그런데 오류 메시지에 `required.item.itemName` 와 같이 객체명과 필드명을 조합한 세밀한 메시지 코드가 있으면 이 메시지를 높은 우선순위로 사용하는 것이다.&#x20;

```
#Level1
required.item.itemName: 상품 이름은 필수 입니다.

#Level2
required: 필수 값 입니다.
```

물론 이렇게 객체명과 필드명을 조합한 메시지가 있는지 우선 확인하고, 없으면 좀 더 범용적인 메시지를 선택하도록 추가 개발을 해야겠지만, 범용성 있게 잘 개발해두면, 메시지의 추가 만으로 매우 편리하게 오류 메시지를 관리할 수 있을 것이다.

**스프링은 `MessageCodesResolver` 라는 것으로 이러한 기능을 지원한다.**&#x20;

## 10. 오류 코드와 메시지 처리4&#x20;

우선 테스트 코드로 `MessageCodesResolver` 를 알아보자.&#x20;

`MessageCodesResolverTest`

```java
package hello.itemservice.validation;

import org.junit.jupiter.api.Test;
import org.springframework.validation.DefaultMessageCodesResolver;
import org.springframework.validation.MessageCodesResolver;

import static org.assertj.core.api.Assertions.*;

public class MessageCodesResolverTest {

    MessageCodesResolver codesResolver = new DefaultMessageCodesResolver();

    @Test
    void messageCodesResolverObject() {
        String[] messageCodes = codesResolver.resolveMessageCodes("required", "item");
        for (String messageCode : messageCodes) {
            System.out.println("messageCode = " + messageCode);
        }

        assertThat(messageCodes).containsExactly(
                "required.item",
                "required"
        );
    }

    @Test
    void messageCodeResolverField() {
        String[] messageCodes = codesResolver.resolveMessageCodes("required", "item", "itemName", String.class);
        for (String messageCode : messageCodes) {
            System.out.println("messageCode = " + messageCode);
        }

        assertThat(messageCodes).containsExactly(
                "required.item.itemName", // 객체명.필드명
                "required.itemName",      // 필드명
                "required.java.lang.String", // 타입명
                "required" // 기본 코드
        );
    }
}
```

#### MessageCodesResolver

* 검증 오류 코드로 메시지 코드들을 생성한다.&#x20;
* `MessageCodesResolver` 인터페이스이고 `DefaultMessageCodesResolver` 는 기본 구현체이다.
* 주로 다음과 함께 사용 `ObjectError`, `FieldError`&#x20;

#### **DefaultMessageCodesResolver의 기본 메시지 생성 규칙**

**객체 오류**&#x20;

```
객체 오류의 경우 다음 순서로 2가지 생성
1.: code + "." + object name
2.: code

예) 오류 코드: required, object name: item
1.: required.item
2.: required
```

**필드 오류**&#x20;

```
필드 오류의 경우 다음 순서로 4가지 메시지 코드 생성
1.: code + "." + object name + "." + field
2.: code + "." + field
3.: code + "." + field type
4.: code

예) 오류 코드: typeMismatch, object name "user", field "age", field type: int
1. "typeMismatch.user.age"
2. "typeMismatch.age"
3. "typeMismatch.int"
4. "typeMismatch"
```

#### 동작 방식&#x20;

*   `rejectValue()`, `reject()` 는 내부에서 `MessageCodesResolver` 를 사용한다. 여기에서 메시지 코드들

    을 생성한다.
*   `FieldError`, `ObjectError` 의 생성자를 보면, 오류 코드를 하나가 아니라 여러 오류 코드를 가질 수 있다.

    &#x20;`MessageCodesResolver` 를 통해서 생성된 순서대로 오류 코드를 보관한다.
*   이 부분을 `BindingResult` 의 로그를 통해서 확인해보자.

    `codes [range.item.price, range.price, range.java.lang.Integer, range]`

#### **FieldError** `rejectValue("itemName", "required")`

다음 4가지 오류 코드를 자동으로 생성&#x20;

* `required.item.itemName`&#x20;
* `required.itemName`
* `required.java.lang.String`
* `required`

#### ObjectError `reject("totalPriceMin")`

다음 2가지 오류 코드를 자동으로 생성&#x20;

* `totalPriceMin.item`&#x20;
* `totalPriceMin`

## 11. 오류 코드와 메시지 처리5&#x20;

### 오류 코드 관리 전략&#x20;

#### 핵심은 구체적인 것에서! 덜 구체적인 것으로!&#x20;

`MessageCodesResolver` 는 `required.item.itemName` 처럼 구체적인 것을 먼저 만들어주고, `required` 처럼 덜 구체적인 것을 가장 나중에 만든다.&#x20;

이렇게 하면 앞서 말한 것 처럼 메시지와 관련된 공통 전략을 편리하게 도입할 수 있다.&#x20;

#### 왜 이렇게 복잡하게 사용하는가?&#x20;

모든 오류 코드에 대해서 메시지를 각각 다 정의하면 개발자 입장에서 관리하기 너무 힘들다.\
크게 중요하지 않은 메시지는 범용성 있는 `requried` 같은 메시지로 끝내고, 정말 중요한 메시지는 꼭 필요할 때 구체적으로 적어서 사용하는 방식이 더 효과적이다.

#### 우선 다음처럼 만들어보자.

`errors.properties`

```
#required.item.itemName=상품 이름은 필수입니다.
#range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
#max.item.quantity=수량은 최대 {0} 까지 허용합니다.
#totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}

#==ObjectError==
#Level1
totalPriceMin.item=상품의 가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}

#Level2 - 생략
totalPriceMin=전체 가격은 {0}원 이상이어야 합니다. 현재 값 = {1}


#==FieldError==
#Level1
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.

#Level2 - 생략

#Level3
required.java.lang.String = 필수 문자입니다.
required.java.lang.Integer = 필수 숫자입니다.
min.java.lang.String = {0} 이상의 문자를 입력해주세요.
min.java.lang.Integer = {0} 이상의 숫자를 입력해주세요.
range.java.lang.String = {0} ~ {1} 까지의 문자를 입력해주세요.
range.java.lang.Integer = {0} ~ {1} 까지의 숫자를 입력
max.java.lang.Integer = {0} 까지의 숫자를 허용합니다.

#Level4
required = 필수 값 입니다.
min= {0} 이상이어야 합니다.
range= {0} ~ {1} 범위를 허용합니다.
max= {0} 까지 허용합니다.

#추가
typeMismatch.java.lang.Integer=숫자를 입력해주세요.
typeMismatch=타입 오류입니다.
```

&#x20;`itemName` 의 경우 `required` 검증 오류 메시지가 발생하면 다음 코드 순서대로 메시지가 생성된다.

1. `required.item.itemName`
2. `required.itemName`&#x20;
3. `required.java.lang.String`&#x20;
4. `required`

그리고 이렇게 생성된 메시지 코드를 기반으로 순서대로 `MessageSource` 에서 메시지에서 찾는다.

구체적인 것에서 덜 구체적인 순서대로 찾는다. 메시지에 1번이 없으면 2번을 찾고, 2번이 없으면 3번을 찾는다.\
이렇게 되면 만약에 크게 중요하지 않은 오류 메시지는 기존에 정의된 것을 그냥 **재활용** 하면 된다!

#### 정리&#x20;

1. `rejectValue()` 호출
2. `MessageCodesResolver` 를 사용해서 검증 오류 코드로 메시지 코드들을 생성
3. `new FieldError()` 를 생성하면서 메시지 코드들을 보관
4. `th:erros` 에서 메시지 코드들로 메시지를 순서대로 메시지에서 찾고, 노출

## 12. 오류 코드와 메시지 처리6&#x20;

### 스프링이 직접 만든 오류 메시지 처리&#x20;

검증 오류 코드는 다음과 같이 2가지로 나눌 수 있다.&#x20;

* 개발자가 직접 설정한 오류 코드 `rejectValue()` 를 직접 호출
* 스프링이 직접 검증 오류에 추가한 경우(주로 타입 정보가 맞지 않음)

지금까지 학습한 메시지 코드 전략의 강점을 지금부터 확인해보자.

#### `price` 필드에 문자 "A"를 입력해보자.

로그를 확인해보면 `BindingResult` 에 `FieldError` 가 담겨있고, 다음과 같은 메시지 코드들이 생성된 것을 확인 할 수 있다. `codes[typeMismatch.item.price,typeMismatch.price,typeMismatch.java.lang.Integer,typeMismatch]`

다음과 같이 4가지 메시지 코드가 입력되어 있다.

* `typeMismatch.item.price`&#x20;
* `typeMismatch.price`&#x20;
* `typeMismatch.java.lang.Integer`&#x20;
* `typeMismatch`&#x20;

그렇다. 스프링은 타입 오류가 발생하면 `typeMismatch` 라는 오류 코드를 사용한다. 이 오류 코드가 `MessageCodesResolver` 를 통하면서 4가지 메시지 코드가 생성된 것이다.

`error.properties` 에 다음 내용을 추가하자

```
#추가
typeMismatch.java.lang.Integer=숫자를 입력해주세요.
typeMismatch=타입 오류입니다.
```

#### **정리**

메시지 코드 생성 전략은 그냥 만들어진 것이 아니다. Bean Validation 을 학습하면 그 진가를 확인할 수 있을 것이다.&#x20;

## 13. Validator 분리1&#x20;

목표&#x20;

* 복잡한 검증 로직을 분리하자.

컨트롤러에서 검증 로직이 차지하는 부분은 매우 크다. 이런 경우 별도의 클래스로 역할을 분리하는 것이 좋다. 그리고 이렇게 분리한 검증 로직을 재사용 할 수도 있다.

#### `ItemValidator` 를 만들자.

```java
package hello.itemservice.web.validation;

import hello.itemservice.domain.item.Item;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.validation.Errors;
import org.springframework.validation.Validator;

public class ItemValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Item item = (Item) target;

        // 검증 로직
        if (!StringUtils.hasText(item.getItemName())) {
            errors.rejectValue("itemName", "required", new Object[]{item.getItemName()}, null);
        }
        if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
            errors.rejectValue("price", "range", new Object[]{1_000, 1_000_000}, null);
        }
        if(item.getQuantity() == null || item.getQuantity() >= 9999) {
            errors.rejectValue("quantity", "max", new Object[]{9_999}, null);
        }

        // 특정 필  드가 아닌 복합 룰 검증
        if(item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10_000) {
                errors.reject("totalPriceMin", new Object[]{10_000, resultPrice}, null);
            }
        }
    }
}
```

스프링은 검증을 체계적으로 제공하기 위해 다음 인터페이스를 제공한다.

```java
public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}
```

* `supports() {}` : 해당 검증기를 지원하는 여부 확인(뒤에서 설명)
* `validate(Object target, Errors errors)` : 검증 대상 객체와 `BindingResult`

#### ItemValidator 직접 호출하기

**ValidationItemControllerV2 - addItemV5()**

```java
@PostMapping("/add")
public String addItemV5(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    itemValidator.validate(item, bindingResult);

    //검증에 실패하면 다시 입력 폼으로
    if (bindingResult.hasErrors()) {
        log.info("errors={}", bindingResult);
        return "/validation/v2/addForm";
    }

    // 성공 로직
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```

지금 컨트롤러 내 검증 코드는 굳이 스프링 인터페이스를 상속받지 않고 사용해도 되는 구조이다.&#x20;

## 14. Validator 분리2&#x20;

스프링이 `Validator` 인터페이스를 별도로 제공하는 이유는 체계적으로 검증 기능을 도입하기 위해서다. 그런데 앞에서는 검증기를 직접 불러서 사용했고, 이렇게 사용해도 된다. 그런데 `Validator` 인터페이스를 사용해서 검증기를 만들면 스프링의 추가적인 도움을 받을 수 있다.

#### @Component 추가&#x20;

```java
package hello.itemservice.web.validation;

import hello.itemservice.domain.item.Item;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.validation.Errors;
import org.springframework.validation.Validator;

@Component
public class ItemValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Item item = (Item) target;

        // 검증 로직
        if (!StringUtils.hasText(item.getItemName())) {
            errors.rejectValue("itemName", "required", new Object[]{item.getItemName()}, null);
        }
        if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1_000_000) {
            errors.rejectValue("price", "range", new Object[]{1_000, 1_000_000}, null);
        }
        if(item.getQuantity() == null || item.getQuantity() >= 9999) {
            errors.rejectValue("quantity", "max", new Object[]{9_999}, null);
        }

        // 특정 필  드가 아닌 복합 룰 검증
        if(item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10_000) {
                errors.reject("totalPriceMin", new Object[]{10_000, resultPrice}, null);
            }
        }
    }
}
```

#### WebDataBinder 를 통해서 사용하기&#x20;

`WebDataBinder` 는 스프링의 파라미터 바인딩의 역할을 해주고 검증 기능도 내부에 포함한다.

**ValidationItemControllerV2에 다음 코드를 추가하자**

```java
@InitBinder
public void init(WebDataBinder binder) {
    // ItemValidator를 Item 클래스에 대한 검증기로 등록
    binder.addValidators(itemValidator);
}
```

이렇게 `WebDataBinder` 에 검증기를 추가하면 해당 컨트롤러에서는 검증기를 자동으로 적용할 수 있다.\
`@InitBinder` ->  해당 컨트롤러에만 영향을 준다. 글로벌 설정은 별도로 해야한다.\
(요청마다 `WebDataBinder` 객체를 생성한다.)

#### @Validated 적용

**ValidationItemControllerV2 - addItemV6()**

```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    //검증에 실패하면 다시 입력 폼으로
    if (bindingResult.hasErrors()) {
        log.info("errors={}", bindingResult);
        return "/validation/v2/addForm";
    }

    // 성공 로직
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```

validator를 직접 호출하는 부분이 사라지고, 대신에 검증 대상 앞에 `@Validated` 가 붙었다.

#### 동작 방식&#x20;

`@Validated`는 검증기를 실행하라는 애노테이션이다.\
이 애노테이션이 붙으면 앞서 `WebDataBinder` 에 등록한 검증기를 찾아서 실행한다. 그런데 여러 검증기를 등록한다면 그 중에 어떤 검증기가 실행되어야 할지 구분이 필요하다. 이때 `supports()` 가 사용된다. 여기서는 `supports(Item.class)` 호출되고, 결과가 `true` 이므로 `ItemValidator` 의 `validate()`가 호출된다.

```java
@Component
public class ItemValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors) {...}
}
```

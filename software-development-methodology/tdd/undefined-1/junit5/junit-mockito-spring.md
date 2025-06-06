# Junit 과 Mockito 기반의 Spring 단위 테스트 코드 작성법

> 참고 링크&#x20;
>
> [https://mangkyu.tistory.com/145](https://mangkyu.tistory.com/145)

## 1. Mockito 소개 및 사용법&#x20;

### Mockito 란?&#x20;

Mockito란 Java 오픈소스 테스트 프레임워크입니다. Mockito를 사용하면 실제 객체를 모방한 가짜 객체, `Mock` 객체 생성이 가능해집니다. 개발자는 이 Mock 객체를 통해 테스트를 보다 간단하고 통일성있게 구현할 수 있습니다.

일반적으로 Spring 으로 웹 애플리케이션을 개발하면, 여러 객체들 간의 의존성이 생긴다. 이러한 의존성은 단위 테스트 작성을 어렵게 하는데, 이를 해결하기 위해서는 가짜 객체를 주입시켜주는 Mockito 라이브러리를 사용할 수 있다. Mockito 를 활용하면 가짜 객체에 원하는 결과를 Stub 하여 단위 테스트를 진행할 수 있다. 물론 프레임워크 도구가 필요없다면 사용하지 않는 것이 가장 좋다.&#x20;

### Mockito 사용법

#### 1. Mock 객체 의존성 주입&#x20;

Mockito 에서 **가짜 객체의 의존성 주입**을 위해서 크게 3가지 어노테이션이 사용된다.&#x20;

* `@Mock` : 가짜 객체를 만들어 반환해주는 어노테이션&#x20;
* `@Spy` : Stub 하지 않은 메서드들은 원본 메서드 그대로 사용하는 어노테이션&#x20;
* `@InjectMocks` : `@Mock` 또는 `@Spy` 로 생성된 가짜 객체를 자동으로 주입시켜주는 어노테이션&#x20;

예를 들어 `UserController` 에 대한 단위 테스트를 작성하고자 할 때, `UserService` 를 사용하고 있다면, `@Mock` 어노테이션을 통해 가짜 `UserService` 를 만들고 `@InjectMocks` 를 통해 `UserController` 에 이를 주입시킬 수 있다.&#x20;

#### 2. Stub 로 결과 처리&#x20;

앞서 설명하였듯, **의존성이 있는 객체는 가짜 객체를 주입하여 어떤 결과를 반환하라고 정해진 답변을 준비시켜야 한다.**\
Mockito 에서는 다음과 같은 stub 메서드를 제공한다.&#x20;

* `doReturn()` : 가짜 객체가 특정한 값을 반환해야 하는 경우&#x20;
* `doNothing()` : 가짜 객체가 아무 것도 반환하지 않는 경우&#x20;
* `doThrow()` : 가짜 객체가 예외를 발생시키는 경우&#x20;

#### 3. Mockito 와 Junit 의 결합&#x20;

Mockito 도 테스팅 프레임워크이기 때문에 Junit 과 결합되기 위해서는 별도의 작업이 필요하다.&#x20;

기존의 JUnit4 에서 Mockito 를 활용하기 위해서는 클래스 어노테이션으로 `@RunWith(MockitoJuintRunner.class)` 를 붙여주어야 연동이 가능했다. 하지만 SpringBoot 2.2.0 부터 공식적으로 Junit5 를 지원함에 따라 이제부터는 `@ExtendWith(MockitoExtenstion.class)` 를 사용해야 결합이 가능하다.&#x20;

## 2. Spring 컨트롤러 단위 테스트 작성 예시&#x20;

### 사용자 회원가입, 목록 조회 API&#x20;

```java
@RestController
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping("/users/signUp")
    public ResponseEntity<UserResponse> signUp(@RequestBody SignUpRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(userService.signUp(request));
    }

    @GetMapping("/users")
    public ResponseEntity<List<UserResponse>> findAll() {
        return ResponseEntity.ok(userService.findAll());
    }
}
```

### 단위 테스트 작성 준비&#x20;

앞서 설명하였듯 Junit5 와 Mockito를 연동하기 위해서는 `@ExtendWith(MockitoExtension.class)` 를 사용해야 한다. 이를 클래스 어노테이션으로 붙여 다음과 같은 테스트 클래스를 작성할 수 없다.&#x20;

```java
@ExtendWith(MockitoExtension.class) 
class UserControllerTest {

}
```

이제 의존성 주입을 해주어야 한다. 먼저 테스트 대상인 `UserController` 에는 가짜 객체 주입을 위한 `@InjectMocks` 를 붙여주어야 한다. 그리고 `UserService` 에는 가짜 객체 생성을 위해 `@Mock` 어노테이션을 붙여주면 된다.&#x20;

```java
@ExtendWith(MockitoExtension.class) 
class UserControllerTest {

    @InjectMocks 
    private UserController userController;
    
    @Mock 
    private UserService userService; 
}
```

컨트롤러를 테스트하기 위해서는 HTTP 호출이 필요하다. 일반적인 방법으로는 HTTP 호출이 불가능하므로 스프링에서는 이를 위한 `MockMVC` 를 제공하고 있다. `MockMVC` 는 다음과 같이 생성할 수 있다.&#x20;

```java
@ExtendWith(MockitoExtension.class) 
class UserControllerTest {
    
    @InjectMocks 
    private UserController userController; 
    
    @Mock 
    private UserService userService; 
    
    private MockMvc mockMvc;
    
    @BeforeEach
    public void init() {
        mockMvc = MockMvcBuilders.standaloneSetup(userController).build(); 
    }
    
}
```

그러면 이제 UserController 를 테스트하기 위한 준비가 끝났으므로, 다음의 케이스들에 대해서 테스트 코드를 작성해주도록 하자.&#x20;

1. 회원가입 성공&#x20;
2. 사용자 목록 조회&#x20;

### 1. 회원가입 성공 테스트

우성 회원가입 요청을 보내기 위해서는 `SignUpRequest` 객체 1개와 `UserService` 의 `signUp` 에 대한 Stub 이 필요하다. 이러한 준비 작업을 해주면 given 단계에 다음과 같은 테스트 코드가 작성된다.&#x20;

```java
@DisplayNamae("회원 가입 성공")
@Test 
void setUpSuccess() throws Exception { 
    // given 
    SignUpRequest request = signUpRequest();
    UserResponse response = userResponse(); 
    
    doReturn(response).when(userService)
        .signUp(any(SignUpRequest.class));
}

private SignUpRequest signUpRequest() {
    return SignUpRequest.builder()
        .email("test@test.com")
        .pw("test")
        .build();
}

private UserResponse userResponse() {
    return UserResponse.builder()
        .email("test@test.test")
        .pw("test")
        .role(UserRole.ROLE_USER)
        .build();
}
```

HTTP 요청을 보내면 Spring 내부에서 `MessageConverter` 를 사용해 Json String 을 객체로 반환한다. 그런데 이것은 Spring 내부에서 진행되므로, 우리가 API 로 전달되는 파라미터인 `SignUpRequest` 를 조작할 수 없다.&#x20;

그래서 `SignUpRequest` 클래스 타입이라면 어떠한 객체도 처리할 수 있도록 `any()` 가 적용되었다. **`any()` 를 사용할 때에는 특정 클래스의 타입을 지정해주는 것이 좋다.**&#x20;

그 다음 `when` 단계에서는 `mockMVC` 에 데이터와 함께 POST 요청을 보내야 한다. 요청 정보는 `mockMvc` 의 `perform` 에서 작성 가능한데, 요청 정보에는 `MockMvcRequestBuilders` 가 사용되며 요청 메소드 종류, 내용, 파라미터 등을 설정할 수 있다.&#x20;

보내는 데이터는 객체가 아닌 문자열이어야 하므로 별도의 변환이 필요하므로 `Gson` 을 사용해 변환하였다.&#x20;

```java
@DisplayName("회원 가입 성공")
@Test
void signUpSuccess() throws Exception {
    // given
    SignUpRequest request = signUpRequest();
    UserResponse response = userResponse();
    
    doReturn(response).when(userService)
        .signUp(any(SignUpRequest.class));

    // when
    ResultActions resultActions = mockMvc.perform(
        MockMvcRequestBuilders.post("/users/signUp")
                .contentType(MediaType.APPLICATION_JSON)
                .content(new Gson().toJson(request))
    );

}
```

마지막으로 호출된 결과를 검증하는 then 단계에서는 회원가입 API 호출 결과로 200 Response 와 응답 결과를 검증해야 한다. 응답 검증 시에는 `jsonPath` 를 이용해 해당 `json` 값이 존재하는지 확인하면 된다.

```java
@DisplayName("회원 가입 성공")
@Test
void signUpSuccess() throws Exception {
    // given
    SignUpRequest request = signUpRequest();
    UserResponse response = userResponse();
    
    doReturn(response).when(userService)
        .signUp(any(SignUpRequest.class));

    // when
    ResultActions resultActions = mockMvc.perform(
        MockMvcRequestBuilders.post("/users/signUp")
                .contentType(MediaType.APPLICATION_JSON)
                .content(new Gson().toJson(request))
    );

    // then
    MvcResult mvcResult = resultActions.andExpect(status().isOk())
        .andExpect(jsonPath("email", response.getEmail()).exists())
        .andExpect(jsonPath("pw", response.getPw()).exists())
        .andExpect(jsonPath("role", response.getRole()).exists())
}
```

### 2. 사용자 목록 조회 테스트&#x20;

사용자 목록 조회 given 단계에서는 `UserService` `findAll` 에 대한 Stub 이 필요하다.&#x20;

그리고 when 단계에서는 호출하는 HTTP 메서드를 GET 으로 , URL 을 "/users/list" 로 작성해주어야 한다. 마지막으로 then 단계에서는 HTTP Status 가 OK 이며, 주어진 데이터가 올바른지를 검증해야 하는데, 이번에는 Json 응답을 객체로 변환하려 확인하자.

```java
@DisplayName("사용자 목록 조회")
@Test
void getUserList() throws Exception {
    // given
    doReturn(userList()).when(userService)
        .findAll();

    // when
    ResultActions resultActions = mockMvc.perform(
        MockMvcRequestBuilders.get("/user/list")
    );

    // then
    MvcResult mvcResult = resultActions.andExpect(status().isOk()).andReturn();
    
    UserListResponseDTO response = new Gson().fromJson(mvcResult.getResponse().getContentAsString(), UserListResponseDTO.class);
    assertThat(response.getUserList().size()).isEqualTo(5);
}

private List<UserResponse> userList() {
    List<UserResponse> userList = new ArrayList<>();
    for (int i = 0; i < 5; i++) {
        userList.add(new UserResponse("test@test.test", "test", UserRole.ROLE_USER));
    }
    return userList;
}
```

### @WebMvcTest&#x20;

위와 같이 `MockMvc` 를 생성하는 등의 작업은 번거롭다. 다행히도 SpringBoot 는 컨트롤러 테스트를 위한 `@WebMvcTest` 어노테이션을 제공하고 있다. 이를 이용하면 `MockMvc` 객체가 자동으로 생성될 뿐 아니라, ControllerAdvice 나, Filter, Interceptor 등 웹 계층 테스트에 필요한 요소들을 모두 빈으로 등록해 스프링 컨텍스트 환경을 구성한다. `@WebMvcTest` 는 스프링부트가 제공하는 테스트 환경이므로 `@Mock` 과 `@Spy` 대신 각각 `@MockBean` 과 `@SpyBean` 을 사용해주어야 한다.&#x20;

```java
@WebMVcTest(UserController.class)
class UserControllerTest {

    @MockBean
    private UserService userService;

    @Autowired
    private MockMvc mockMvc;

    // 테스트 작성

}
```

#### 주의할 점&#x20;

스프링은 내부적으로 스프링 컨텍스트를 캐싱해두고 동일한 테스트 환경이라면 재사용한다. 그런데 특정 컨트롤러만을 빈으로 만들고 @MockBran 과 @SpyBean 으로 빈을 모킹하는 @WebMvcTest 는 캐싱의 효과를 거의 얻지 못하고 새로운 컨텍스트의 생성을 필요로 한다.&#x20;

그러므로 빠른 테스트를 원한다면 직접 MockMvc 를 생성했던 처음의 방법을 사용하는 것이 좋을 수 있다.&#x20;

## 3. Spring 서비스 계층 단위 테스트 작성 예시&#x20;

### 사용자 회원가입/목록 조회 비지니스 로직&#x20;

사용자 회원가입과 목록 조회를 위해서는 다음과 같은 비지니스 로직 레이어(Service Layer) 가 필요하다.

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserServiceImpl {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder passwordEncoder;

    @Transactional
    public UserResponse signUp(final SignUpRequest request) {
        final User user = User.builder()
                .email(request.getEmail())
                .pw(passwordEncoder.encode(request.getPw()))
                .role(UserRole.ROLE_USER)
                .build();

        return UserResponse.of(userRepository.save(user));
    }

    public List<User> findAll() {
        return userRepository.findAll().stream()
            .map(UserResponse::of)
            .collect(Collectors.toList()));
    }
}
```

### 단위 테스트 작성 준비&#x20;

앞서 설명하였듯 `@ExtendWith(MockitoExtension.class)` 와 가짜 객체 주입을 사용해 다음과 같은 테스트 클래스를 작성할 수 있다.&#x20;

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @InjectMocks
    private UserService userService;

    @Mock
    private UserRepository userRepository;

    @Spy
    private BCryptPasswordEncoder passwordEncoder;

}
```

이번에는 `BCryptPasswordEncoder` 에 `@Spy` 를 사용하였다. 앞서 설명하였듯 Spy 는 Stub 하지 않은 메서드는 실제 메서드로 동작하게 하는데, 위의 예제에서 실제로 사용자 비밀번호를 암호화해야하므로, `@Spy` 를 사용하였다. 이번에도 테스트 코드를 작성해보자.&#x20;

1. 회원가입 성공&#x20;
2. 사용자 목록 조회&#x20;

### 1. 회원가입 성공 테스트&#x20;

```java
@DisplayName("회원 가입")
@Test
void signUp() {
    // given
    BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
    SignUpRequest request = signUpRequest();
    String encryptedPw = encoder.encode(request.getPw());

    doReturn(new User(request.getEmail(), encryptedPw, UserRole.ROLE_USER)).when(userRepository)
        .save(any(User.class));
        
    // when
    UserResponse user = userService.signUp(request);

    // then
    assertThat(user.getEmail()).isEqualTo(request.getEmail());
    assertThat(encoder.matches(signUpDTO.getPw(), user.getPw())).isTrue();

    // verify
    verify(userRepository, times(1)).save(any(User.class));
    verify(passwordEncoder, times(1)).encode(any(String.class));
}
```

이번에는 추가적으로 mockito 의 `verify()` 를 사용해보았다. **`verity` 는 Mockito 라이브러리를 통해 만들어진 가짜 객체의 특정 메서드가 호출된 횟수를 검증할 수 있다.**&#x20;

위에서는 `passwordEncoder` 의 `encode` 메서드와 `userRepository` 의 `save` 메서드가 각각 1번만 호출되었는지를 검증하기 위해서 사용했다.&#x20;

### 2. 사용자 목록 조회 테스트&#x20;

```java
@DisplayName("사용자 목록 조회")
@Test
void findAll() {
    // given
    doReturn(userList()).when(userRepository)
        .findAll();

    // when
    final List<UserResponse> userList = userService.findAll();

    // then
    assertThat(userList.size()).isEqualTo(5);
}

private List<User> userList() {
    List<User> userList = new ArrayList<>();
    for (int i = 0; i < 5; i++) {
        userList.add(new User("test@test.test", "test", UserRole.ROLE_USER));
    }
    return userList;
}
```

## 4. Spring 레포지토리 계층 단위 테스트 작성 예시&#x20;

### 사용자 추가/목록 조회 코드&#x20;

사용자 회원가입과 목록 조회를 위한 JPA 레포지토리 인터페이스는 다음과 같이 구현되어 있다.&#x20;

```java
public interface UserRepository extends JpaRepository <User, Long> {

}
```

이번에도 역시 다음과 같은 기능들에 대한 테스트 코드를 작성해보도록 하자

1. 사용자 추가&#x20;
2. 사용자 목록 조회&#x20;

### @DataJpaTest 어노테이션&#x20;

스프링 부트는 JPA 레포지토리를 손쉽게 테스트 할 수 있는 `@DataJpaTest` 어노테이션을 제공하고 있다.

`@DataJpaTest` 를 사용하면 기본적으로 인메모리 데이터베이스인 H2 를 기반으로 테스트용 테이터베이스를 구축하며, 테스트가 끝나면 트랜잭션을 롤백 해준다. 레포지토리 계층은 실제 DB 와 통신없이 단순 모킹하는 것은 의미가 없으므로 직접 데이터베이스와 통신하는 `@DataJpaTest` 를 사용하도록 하자.&#x20;

### 1. 사용자 추가 테스트&#x20;

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @DisplayName("사용자 추가")
    @Test
    void addUser() {
        // given
        User user = user();
        
        // when
        User savedUser = userRepository.save(user);

        // then
        assertThat(savedUser.getEmail()).isEqualTo(user.getEmail());
        assertThat(savedUser.getPw()).isEqualTo(user.getPw());
        assertThat(savedUser.getRole()).isEqualTo(user.getRole());
    }

    private User user() {
        return User.builder()
                .email("email")
                .pw("pw")
                .role(UserRole.ROLE_USER).build();
    }
}
```

### 2. 사용자 목록 테스트&#x20;

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @DisplayName("사용자 목록 조회")
    @Test
    void addUser() {
        // given
        userRepository.save(user());
        
        // when
        List<User> userList = userRepository.findAll();

        // then        
        assertThat(userList.size()).isEqualTo(1);
    }
}
```

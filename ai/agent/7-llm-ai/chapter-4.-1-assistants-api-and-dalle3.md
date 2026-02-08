# chapter 4. 에이전트1 : 자동화된 사무 구현 Assistants API & DALLE3 모델을 이용한 프레젠테이션 제작

## 4.1 OpenAI 의 도우미란 무엇인가?&#x20;

OpenAI 의 assistant 는 GPT 모델을 바탕으로 한 언어 이해와 생성을 위한 플랫폼이다. 이 플랫폼은 정보를 제공하고 질문에 답하며 텍스트를 생성하고 특정 작업을 수행함으로 우리의 일상 업무를 도울 수 있다.&#x20;

여기까지 들으면 OpenAI assistant 가 에이전트와 비슷하다는 느낌이 들 수 있지만, assistant 는 유연하고 다양한 기능을 갖추도록 설계되어 간단한 일상  대화부터 복잡한 기술 문제 해결까지 다양한 상황에 적용될 수 있다.&#x20;

주요 특징은 다음과 같다.&#x20;

* 자연어 이해 : 자연어 입력을 이해하고 처리하여 사용자의 의도와 요구를 파악할 수 있다.&#x20;
* 풍부한 텍스트 생성 : 사용자 지시에 따라 일관되고 관련성이 높은 유용한 텍스트 응답을 생성할 수 있다.&#x20;
* 적응성과 맞춤 설정 : 특정 응용 시나리오와 요구에 맞춰 개인화된 서비스를 제공하도록 맞춤 설정을 할 수 있다.&#x20;
* 상호작용성 : 문맥을 이해해 일관된 대화를 이러가면서 대화 기록을 기억하여 더 깊고 연속적인 상호작용 경험을 제공한다.&#x20;
* 쉬운 통합 : 웹사이트, 애플리케이션, 등등 과 쉽게 통합할 수 있다.&#x20;

이러한 특징 덕분에 도우미는 고객 서비스 자동화, 개인 비서, 교육, 콘텐츠 생성, 프로그래밍 지원과 같이 매우 광범위한 분야에서 다양하게 응용할 수 있다. 더군다나 도우미는 계속 학습하고 적응하는 과정을 반복하므로서, 점점 더 효율적이고 유연하게 지능적으로 발전하고 있다.&#x20;

***

## 4.3 Assistants API 의 간단한 예제&#x20;

~~2026.08.26 deprecated 예정~~\
-> Response API 에서 해당 기능 지원

{% embed url="https://platform.openai.com/docs/api-reference/responses" %}

Assistants API 는 자신의 프로그램 내에서 인공지능 도우미를 구축할 수 있는 도구에 해당한다. 도우미에게 명령을 내리면 모델이나 도구뿐만 아니라 문서와 같은 외부 지식을 활용하여 우리의 명령에 맞는 응답을 생성해준다.&#x20;

2024년 8월 기준 Assistants API 는 코드 해석기, 검색, 함수 호출이라는 세 가지 유형의 도구를 지원한다. Assistants API 는 여러 함수를 한 번에 호출하는 기능을 제공하는데, **개발자는 이를 통해 대화 흐름과 상황 정보를 직접 관리할 필요 없이 OpenAI 서비스가 이를 대신하게 할 수 있다.**&#x20;

Assistants API 를 호출하는 절차는 다음과 같다.&#x20;

1. **assistants** 를 생성해 지침을 정의하고 모델을 선택한다. 이때 코드 해석기, 검색, 함수 호출 등의 도구를 활성화할 수 있다.&#x20;
2. 사용자가 대화를 시작하면서 **대화 흐름(thread)** 을 생성한다.&#x20;
3. 사용자가 질문을 던지면 **해당 메시지(message)** 를 흐름에 추가한다.&#x20;
4. thread 에서 assistant 를 실행해서 응답을 발생시킨다. 이 과정에서 관련 **도구**가 자동으로 호출된다.&#x20;

#### Assistant 생성하기&#x20;

{% embed url="https://platform.openai.com/docs/api-reference/assistants/createAssistant" %}

Assistant 를 생성할 때는 다음과 같은 매개변수를 설정할 필요가 있다.&#x20;

* name : assistant 이름을 지정&#x20;
* instructions : assistant 가 어떻게 행동하고 응답해야 하는지를 알려주는 지침으로, '당신은 특정 분야의 전문가 입니다' 와 같은 방식으로 지정한다.&#x20;
* tools : Assistants API 는 OpenAI 에서 구축하고 관리하고 있는 도구인 code\_interpreter, file\_search 를 지원한다.&#x20;
* model : 모델 선택&#x20;
* functions : 사용자 정의 함수를 도구로 추가할 수 있다. 이에 대해서는 뒤에서 자세히 다루자.&#x20;

아래와 같이 create API 를 호출하면 대시보드 Assistants 메뉴에서 생성된 것이 확인 가능하다.&#x20;

* 여기서 알아둘 것은 생성된 Assistant 마다 고유의 ID 가 부여되므로, 다음부터는 이 ID 를 이용해 앞에서 생성했던 Assistant 를 반복 호출할 수 있다는 것이다. 그렇지 않으면 동일한 기능을 가지지만 서로 다른 ID 가 부여된 여러개의 불필요한 Assistant 가 생성된다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2026-02-08 21.14.16.png" alt=""><figcaption></figcaption></figure>

#### 대화 흐름 생성하기&#x20;

{% embed url="https://platform.openai.com/docs/api-reference/threads/createThread" %}

Assistant 를 생성했다면 이제 대화 흐름을 생성할 차례이다.&#x20;

아래와 같이 대화 흐름을 생성하면 Assistant 와 마찬가지로 ID 가 리턴되고 이 ID 를 통해서 컨텍스트를 유지하며 대화를 할 수 있다.&#x20;

* 여기서 알아둘 것은 도우미 생성과 마찬가지로 대화 흐름 생성 코드를 반복적으로 실행하면 안된다는 것이다. 그렇지 않으면 마찬가지로 동일하지만 서로 다른 ID 가 부여된 여러 개의 불필요한 대화 흐름이 생성된다.&#x20;
* OpenAI API 에서 대화 흐름과 Assistant 는 연속적인 대화와 작업 처리를 위해 서로 연관되어 있기는 하지만, 기술적으로는 서로 독립적인 구성 요소에 해당된다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2026-02-08 21.21.01.png" alt=""><figcaption></figcaption></figure>

#### 메시지 추가하기&#x20;

{% embed url="https://platform.openai.com/docs/api-reference/messages/createMessage" %}

#### Assistant 실행하기&#x20;

{% embed url="https://platform.openai.com/docs/api-reference/runs/createRun" %}

Assistant 실행 후 실행 세션의 상태는 기본적으로 대기중(queued) 상태이고, 시스템 자원이 할당되면 곧바로 진행 중(in\_progress) 상태가 된 다음, 완료(completed) 상태가 된다.&#x20;

* 그 외 상태는 공식 문서 살펴보기...

`runs.retrieve` 메서드를 지속적으로 호출하여 해당 세션의 상태가 완료되었는지 지속적으로 판단해야 한다.&#x20;

#### 응답 표시하기&#x20;

{% embed url="https://platform.openai.com/docs/api-reference/messages/listMessages" %}

세션 상태가 완료되었다면, `List messages` 를 조회해 응답 결과를 확인할 수 있다.&#x20;

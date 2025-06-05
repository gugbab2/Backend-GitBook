# 스트림 API3 - 컬렉터

## 컬렉터1&#x20;

스트이 중간 연산을 거쳐 최종 연산으로써 데이터를 처리할 때, 그 결과물이 필요한 경우가 많다. 대표적으로 "리스트 나 맵 같은 자료구조에 담고 싶다"거나 "동계 테이터를 내고 싶다" 는 식의 요구가 있을 때 이 최종 연산에 `Collectors` 를 활용한다.&#x20;

collect 연산(예: `stream.collect(...)`)은 **반환값**을 만들어내는 최종 연산이다. `collect(Collector<? super T, A, R> collector)` 형태를 주로 사용하고, `Collectors` 클래스 안에 준비된 여러 메서드를 통해서 다양한 수집 방식을 적용할 수 있다.

**참고**: 필요한 대부분의 기능이 `Collectors` 에 이미 구현되어 있기 때문에, `Collector` 인터페이스를 직접 구현하는 것보다는 `Collectors` 의 사용법을 익히는 것이 중요하다.

### Collectors 의 주요 기능 표 정리&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-06-05 11.43.26.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-06-05 11.43.35.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-06-05 11.43.43.png" alt=""><figcaption></figcaption></figure>

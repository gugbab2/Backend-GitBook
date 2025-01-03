# Checksum (검사합)

## Checksum

* 데이터 오류 여부 확인 방법으로 널리 사용된다.&#x20;
* 보안성이 없다!\
  &#xNAN;**-> 중간에 해킹되어 복사본의 값을 조작하고, 원본의 검사합까지 조작한다면 서버 입장에서는 데이터가 조작된 줄 모른다! (이러한 문제점을 해결하기 위해서 Hash 가 등장)**
* 검사합의 생성 과정은 다음과 같다.&#x20;
  * 일정 자리수를 정하고 범위를 넘는 자리 올림은 버려서 자릿수를 유지한다.&#x20;
* 검사합의 사용 과정은 다음과 같다.&#x20;
  * **클라이언트와 서버가 데이터를 송수신 하는 과정에서 원본과 복사본이 존재한다.**&#x20;
  * **클라이언트에서 원본을 서버로 보낼 때 검사합을 같이 보낸다.**&#x20;
  * **서버에서는 복사본과 복사본의 검사합을 만들고, 원본의 검사합과 비교한다.**&#x20;
  * **검사합이 같다면 문제가 없지만, 검사합이 다르다면 데이터의 문제가 있다고 판단한다.**&#x20;

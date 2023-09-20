# Chapter 1. 리팩터링 리팩터링하기

## 1.1 리팩터링이란 무엇인가?
* 기능을 변경하지 않고 코드의 가독성과 유지보수가 쉽도록 코드를 변경하는 것이다.
* 리팩터링을 해야하는 이유
  * 경제적인 이유 - 코드베이스의 가독성을 높이면 새로운 기능을 구현하기 위한 시간을 확보할 수 있다.
  * 유지보수 - 유지보수가 용이해지면 버그가 줄어들고 수정이 쉬워진다.
  * 좋은 코드베이스는 생각하기 편하다.

## 1.2 스킬: 무엇을 리팩터링할 것인가?
* 리팩터링할 대상을 결정하는 데 쉽게 인식하고 적용할 수 있는 규칙이 있다. 이 규칙들은 앞으로 알아갈 것이다.
* 때로는 규칙이 너무 엄격해서 스멜이 없는 코드를 수정해야 할 수 있다.
* 드물지만 규칙을 지키고도 스멜을 풍기는 코드가 될 수 있다.

## 1.3 문화: 리팩터링은 언제 할까?
* 리팩터링은 정기적으로 수행하는 것이 효과적이고 비용이 적게 들기 때문에 가능하면 일상 업무에 통합하는 것이 좋다.

### 1.3.1 레거시 시스템에서의 리팩터링
* 우선 변경하기 쉽게 만든 후 변경하라.
* 새로운 것을 구현할 때마다 새 코드를 쉽게 추가할 수 있게 리팩터링을 먼저 한다.

### 1.3.2 언제 리팩터링을 하지 말아야 할까?
* 리팩터링은 정기적으로 하지 않을 경우 시간이 많이 걸릴 수 있다.
* 리팩터링이 필요하지 않는 경우
  * 한 번 실행하고 삭제할 코드. (익스트림 프로그래밍 커뮤니티에서 스파이크로 알려진 것이다.)
  * 폐기되기 전 유지보수 모드에 있는 코드
  * 임베디드 시스템이나 게임의 고급 물리엔진과 같이 엄격한 성능 요구사항이 있는 코드
* 위 세 가지 외에는 리팩터링에 투자하자.

## 1.4 도구: (안전한) 리팩터링 방법
* 자동화된 테스트를 수행하면 안심하고 빠르게 진행할 수 있다.
* 리팩터링 할 때 도움이 되는 도구
  * 리팩터링 패턴
  * 버전 관리 (Git)
  * 컴파일러
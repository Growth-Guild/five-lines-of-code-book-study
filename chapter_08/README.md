# Chapter 8. 주석 자제하기
## 8.1 오래된 주석 제거
* 오래된 주석은 작성됐을 당시에는 의도된 것이지만 코드베이스와 동기화되지 않아 완전히 잘못되었거나 오해의 소지가 있는 주석도 포함되어 있을 수 있다.
* 이런 주석은 시간을 절약해주지도 않고 읽는 데 시간이 걸리므로 삭제해야 한다.

## 8.2 주석 처리된 코드 제거
* 간혹 일부 코드를 주석 처리하고 테스트할 때가 있다. 코드를 주석 처리하고 어떤 일이 일어나는지 확인하는 것은 빠르고 쉽다.
* 그러나 테스트 후에 주석 처리된 코드를 모두 삭제해야 한다.
* 코드는 버전 관리를 통해 관리되기 때문에 삭제 후에도 쉽게 복구할 수 있다.

## 8.3 불필요한 주석 제거
* 코드가 주석만큼 읽기 쉬울 때 해당 주석은 불필요한 주석이기 때문에 제거한다.
* 코드를 읽을 때 무시하는 주석도 포함된다. 주석을 읽지 않으면 가치가 없기 때문이다.

## 8.4 메서드의 이름으로 주석 대신하기
* 일부 주석은 기능보다는 코드를 문서화한다.
  * 이런 경우는 주석과 동일한 이름을 가진 메서드로 블록을 간단히 추출할 수 있다.

### 8.4.1 계획을 위한 주석 사용
* 이러한 유형의 주석은 작업에 대한 계획을 세우고 작업을 세분화할 때 자주 사용된다.
```typescript
// 데이터를 불러옴
// 필요 사항을 검사
//   변형
// ELSE
//   제출 
```

## 8.5 불변 속성을 문서화한 주석 유지
* 로컬이 아닌 불변속성을 문서화한 주석은 버그가 발생할 확률이 높다.
  * 이를 판단하는 방법은 이 주석으로 누군가 버그를 만드는 것을 막을 수 있는지 자문해 보는 것이다.
* TODO나 probably, fixme 또는 임시 방편으로 서드파티 소프트웨어 사용 같은 항목을 나타내는 데 사용되는 주석의 키워드는 불변속성이다.
  * 이는 코드의 불변속성이 아니라, 프로세스의 불변속성이다.
  * 이것들은 코드가 아닌 티켓 시스템에 있어야 하는 것이 좋다.

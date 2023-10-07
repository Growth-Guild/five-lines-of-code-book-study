# Chapter 4. 타입 코드 처리하기
- - -

## 4.1 간단한 if 문 리팩터링

### 4.1.1 규칙: if 문에서 else를 사용하지 말 것

#### 정의
* 프로그램에서 이해하지 못하는 타입(형)인지를 검사하지 않는 한 if 문에서 else를 사용하지 말아야 한다.

#### 설명
* if-else를 사용하면 코드에서 결정이 내려지는 지점을 고정하게 된다.
* if-else는 하드코딩된 결정으로 볼 수 있다.
  * 코드에서 하드코딩된 상수가 좋지 않은 것처럼 하드코딩된 결정도 좋지 않다.
  * 어떤 결정을 하드코딩하지 않는 방법은 if 구문을 else 구문과 함께 사용하지 않는 것이다.
* 애플리케이션 외부에서 입력을 받는 프로그램의 경계에서 발생하는 경우에는 문제가 되지 않는다. (사용자 입력 또는 데이터베이스에서 값을 읽어오기 등)
  * 이 경우에 가장 먼저 할 일은 외부의 데이터 타입을 내부에서 제어 가능한 데이터 타입으로 매핑하는 것이다.
* 독립된 if 문은 검사(check)로 간주하고, if-else 문은 의사결정(decision)으로 간주한다.

#### 스멜
* 이 규칙은 스멜로 인식되는 이른 바인딩(early binding)과 관련이 있다.
* if-else 같은 의사결정 동작은 컴파일 시 처리되어 애플리케이션에 고정되며 재컴파일 없이는 수정할 수 없고, 반대는 실행되는 순간에 동작이 결정되는 늦은 바인딩(late binding)이다.
* 이른 바인딩은 if 문을 수정해야 변경할 수 있기 때문에 추가에 의한 변경을 방해한다.

### 4.1.3 리팩터링 패턴: 클래스로 타입 코드 대체
#### 설명
* 열거형을 인터페이스로 변환하고 열거형의 값들은 클래스가 된다.
  * 이렇게 하면 각 값에 속성을 추가하고 해당 특정 값과 관련된 기능을 특성에 맞게 만들 수 있다.
* 열거형에 새 값을 추가하는 것은 수많은 파일에 걸쳐서 해당 열거형과 연결된 로직들을 확인해야 한다.
* 인터페이스를 구현한 새로운 클래스를 추가하는 것은 해당 클래스에 메서드의 구현이 필요할 뿐, 새로운 클래스를 사용하기 전까지는 다른 코드를 수정하지 않아도 된다.

#### 절차
1. 임시 이름을 가진 새로운 인터페이스를 도입한다. 인터페이스에는 열거형의 각 값에 대한 메서드가 있어야 한다.
2. 열거형의 각 값에 해당하는 클래스를 만든다. 클래스에 해당하는 메서드를 제외한 인터페이스의 모든 메서드는 false를 반환해야 한다.
3. 열거형의 이름을 다른 이름으로 바꾼다.
4. 타입을 이전 이름에서 임시 이름으로 변경하고 일치성 검사를 새로운 메서드로 대체한다.
5. 남아있는 열거형 값에 대한 참조 대신 새로운 클래스를 인스턴스화하여 교체한다.
6. 오류가 더 이상 없으면 인터페이스의 이름을 모든 위치에서 영구적인 것으로 바꾼다.

#### 예제
```java
// 규칙 적용 전
enum TrafficLight {
    RED, YELLOW, GREEN
}

class Main {
    private static final TrafficLight[] CYCLE = {TrafficLight.RED, TrafficLight.YELLOW, TrafficLight.GREEN};
    
    public void updateCarForLight(TrafficLight current) {
        if (current == TrafficLight.RED) {
            car.stop();
        } else {
            car.drive();
        }
    }
}
```
1. 임시 이름으로 새로운 인터페이스를 도입한다.
```java
interface TrafficLight2 {
    boolean isRed();
    boolean isYellow();
    boolean isGreen();
}
```
2. 각 열거형 값에 대한 클래스를 만든다.
```java
class Red implements TrafficLight2 {
    public void isRed() { return true; }
    public void isYellow() { return false; }
    public void isGreen() { return false; }
}
class Yellow implements TrafficLight2 { /* ... */ }
class Green implements TrafficLight2 { /* ... */ }
```
3. 열거형의 이름을 다른 이름으로 바꾼다.
```java
enum RawTrafficLight {
    RED, YELLOW, GREEN
}
```
4. 타입을 이전 이름에서 임시 이름으로 변경하고 일치 여부 검사를 새로운 메서드로 대체한다.
5. 열거형 값에 대한 나머지 참조를 새 클래스로 인스턴스화해서 교체한다.
```java
class Main {
    private static final TrafficLight[] CYCLE = { new Red(), new Green(), new Yellow() };
    
    public void updateCarForLight(TrafficLight2 current) {
        if (current.isRed()) {
            car.stop();
        } else {
            car.drive();
        }
    }
}
```
6. 마지막으로 더 이상 오류가 없으면 인터페이스 이름을 모든 위치에서 영구적인 이름으로 바꾼다.
- - -
* 모든 값에 대한 메서드를 갖는 것도 스멜이기 때문에 이것은 하나의 스멜을 다른 스멜로 대체한 것이다.
  * 하지만 열거형 값들은 밀접하게 연결되어 있는 반면, 메서드는 하나씩 처리할 수 있다.

### 4.1.5 리팩터링 패턴: 클래스로의 코드 이관

#### 설명
* 기능을 클래스로 옮기기 때문에 if 구문이 제거되고 기능이 데이터에 더 가까이 이동한다.
* 특정 값과 연결된 기능이 값에 해당하는 클래스로 이동하기 때문에 이는 불변속성을 지역화하는 데 도움이 된다.

#### 절차
1. 원래 함수를 복사하여 모든 클래스로 붙여 넣는다.
2. 메서드 선언 부분의 메서드 이름과 매개변수 리스트를 인터페이스에 복사하고 원래 메서드와 약간 다른 이름을 지정한다.
3. 모든 클래스에서 새로운 메서드를 점검한다.
4. 원래 함수의 본문을 새로운 메서드에 대한 호출로 바꾼다.

#### 예제
```java
// 초기 코드
class Red implements TrafficLight {
    public void isRed() { return true; }
    public void isYellow() { return false; }
    public void isGreen() { return false; }
}
class Yellow implements TrafficLight { /* ... */ }
class Green implements TrafficLight { /* ... */ }

class Main {
  private static final TrafficLight[] CYCLE = { new Red(), new Green(), new Yellow() };

  public void updateCarForLight(TrafficLight current) {
    if (current.isRed()) {
      car.stop();
    } else {
      car.drive();
    }
  }
}
```

1. 인터페이스에 새로운 메서드를 만든다.
```java
interface TrafficLight {
    // ...
  void updateCar();
}
```
2. 소스 함수를 복사해서 모든 클래스에 붙여 넣는다.
```java
class Red implements TrafficLight {
  public void isRed() { return true; }
  public void isYellow() { return false; }
  public void isGreen() { return false; }
  public void updateCar() {
      if (this.isRed()) {
          car.stop;
      } else {
          car.drive();
      }
  }
}
class Yellow implements TrafficLight { /* ... */ }
class Green implements TrafficLight { /* ... */ }
```
3. 모든 클래스에서 새로운 메서드를 점검한다.
```java
class Red implements TrafficLight {
  public void isRed() { return true; }
  public void isYellow() { return false; }
  public void isGreen() { return false; }
  public void updateCar() { car.stop(); }
}
class Yellow implements TrafficLight { /* ... */ }
class Green implements TrafficLight { /* ... */ }
```
4. 원래 함수의 본문을 새로운 메서드에 대한 호출로 바꾼다.
```java
class Main {
  private static final TrafficLight[] CYCLE = { new Red(), new Green(), new Yellow() };

  public void updateCarForLight(TrafficLight current) {
    current.updateCar();
  }
}
```
* is로 시작하는 메서드가 남아 있는 것은 스멜이었는데, 이 시점에서는 is로 시작하는 메서드가 필요하지 않게 되었다.

### 4.1.7 리팩터링 패턴: 메서드의 인라인화
#### 설명
* 이 패턴은 프로그램에서 더 이상 가독성에 도움이 되지 않는 메서드를 제거한다.
* 그 방법은 메서드에서 이를 호출하는 모든 곳으로 코드를 옮기는 것이다.
* 메서드 인라인화 리팩터링 패턴은 모든 호출 측을 수정하여 원래의 메서드를 제거한다.
* 고려사항으로는 메서드가 인라인화하기에 너무 복잡하지 않은지 여부이다.

#### 절차
1. 메서드 이름을 임시로 변경한다. (함수를 사용하는 모든 곳에서 컴파일 오류가 발생한다.)
2. 메서드의 본문을 복사하고 매개변수를 기억해 둔다.
3. 컴파일러가 오류를 발생시킨 모든 곳의 호출을 복사된 코드로 교체하고 인자를 매개변수에 매핑한다.
4. 오류 없이 컴파일되면 원래의 메서드가 더 이상 사용되지 않는 것이므로 원래 메서드를 삭제한다.

#### 예제
```java
// 초기 코드
class AccountService {
    public void deposit(String to, Long amount) {
        Long accountId = databse.find(to);
        database.updateOne(accountId, amount);
    }
    
    public void transfer(String from, String to, Long amount) {
        deposit(from, -amount);
        deposit(to, amount);
    }
}
```
1. 메서드의 이름을 임시로 변경한다.
```java
class AccountService {
    public void deposit2(String to, Long amount) {
        Long accountId = databse.find(to);
        database.updateOne(accountId, amount);
    }
    // ...
}
```
2. 메서드 본문을 복사하고 매개변수를 기억해 둔다.
3. 컴파일러에서 오류가 발생하는 모든 곳에서 메서드 호출을 복사된 코드로 변경하고 인자를 매개변수에 매핑한다.
```java
class AccountService {
    // ...
    
    public void transfer(String from, String to, Long amount) {
      Long fromAccountId = databse.find(from);
      database.updateOne(fromAccountId, -amount);

      Long toAccountId = databse.find(to);
      database.updateOne(toAccountId, amount);
    }
}
```
4. 컴파일러 오류가 없으면 원래 메서드가 더 이상 사용되지 않는다는 것을 의미하므로, 원래 메서드를 삭제한다.

* 위 예시는 코드 복제가 좋은 생각인지 아닌지의 논란의 여지가 있고, 캡슐화를 통해서 다른 해결책을 사용할 수 있다.

### 4.2.2 리팩터링 패턴: 메서드 전문화
#### 설명
* 프로그래머들은 일반화하고 재사용 하려는 본능적인 욕구가 있지만 그렇게 하면 책임이 흐려지고 다양한 위치에서 코드를 호출할 수 있기 때문에 문제가 될 수 있다.
* 이 리팩터링 패턴은 이 효과를 반전 시킨다. 전문화된 메서드는 더 적은 위치에서 호출되어 필요성이 없어지면 더 빨리 제거할 수 있게 된다.

#### 절차
1. 전문화하려는 메서드를 복제한다.
2. 메서드 중 하나의 이름을 새로 사용할 메서드의 이르으로 변경하고 전문화하려는 매개변수를 제거(또는 교체)한다.
3. 매개변수 제거에 따라 메서드를 수정해서 오류가 없도록 한다.
4. 이전의 호출을 새로운 것을 사용하도록 변경한다.

#### 예제
```java
// 초기 코드
class Chess {
    public boolean canMove(Tile start, Tile end, int dx, int dy) {
        return dx * abs(start.x - end.x) == dy * abs(start.y - end.y)
                || dy * abs(start.x - end.x) == dx * abs(start.y - end.y);
    }
    
    // ...
}
```
1. 전문화하려는 메서드를 복제한다.
```java
class Chess {
    public boolean canMove(Tile start, Tile end, int dx, int dy) {
        return dx * abs(start.x - end.x) == dy * abs(start.y - end.y)
                || dy * abs(start.x - end.x) == dx * abs(start.y - end.y);
    }

    public boolean canMove(Tile start, Tile end, int dx, int dy) {
      return dx * abs(start.x - end.x) == dy * abs(start.y - end.y)
              || dy * abs(start.x - end.x) == dx * abs(start.y - end.y);
    }
    
    // ...
}
```

2. 메서드 중 하나를 새로운 이름으로 바꾸고 전문화할 매개변수를 제거(또는 교체)한다.
```java
class Chess {
    public boolean rookMove(Tile start, Tile end, int dx, int dy) {
        return 1 * abs(start.x - end.x) == 0
                || 0 == 1 * abs(start.y - end.y);
    }
    
    // ...
}
```
3. 오류가 없도록 메서드를 수정한다.
```java
class Chess {
    public boolean rookMove(Tile start, Tile end, int dx, int dy) {
        return 1 * abs(start.x - end.x) == 0 * abs(start.y - end.y)
                || 0 * abs(start.x - end.x) == 1 * abs(start.y - end.y);
    }
    
    // ...
}
```
4. 이전의 호출을 새로운 것으로 변경한다.

### 4.2.4 규칙: switch를 사용하지 말 것
#### 정의
* default 케이스가 없고 모든 case에 반환 값이 있는 경우가 아니라면 switch를 사용하지 않는 것이 좋다.

#### 설명
* switch는 각각 버그로 이어지는 두 가지 편의성을 허용한다는 문제가 있다.
  * 첫 번째는 case를 분석할 때 모든 값에 대한 처리를 실행할 필요가 없도록 default 키워드가 지원되는데, 이는 default로 중복 없이 여러 값을 지정할 수 있게 된다.
    * switch를 사용할 경우 무엇을 처리할지와 무엇을 처리하지 않을지는 이제 불변속성인데, 기본값이 지정된 다른 경우와 마찬가지로 새로운 값을 추가할 때 이러한 불변속성이 여전히 유효한지 컴파일러를 통해 판단할 수 없게 된다.
    * 컴파일러 입장에서는 새로 추가한 값의 처리를 잊어버린 것인지, 아니면 default에 지정하고자 한 것인지를 구분할 방법이 없다.
  * 두 번째는 break 키워드를 만나기 전까지 케이스를 연속해서 실행하는 폴스루(fail-through) 로직이라는 점이다.
    * break 키워드를 쓰는 것을 누락하고 알아채지 못하기 쉽다.
* 첫 번째 문제는 기능을 default에 두지 않는 것으로 해결할 수 있다.
* 두 번째 문제는 return을 지정해서 폴스루 문제를 해결한다.

#### 스멜
* switch는 컨텍스트를 처리하는 방법에 초점을 맞춘다.
* 반대로 클래스에 기능을 밀어 넣을 때는 데이터가 상황을 처리하는 방법에 초점을 맞춘다.

### 4.3.1 인터페이스 대신 추상 클래스를 사용할 수는 없을까?
* 사용할 수 있고, 그렇게 함으로써 코드의 중복을 피할 수 있다.
* 인터페이스는 도입한 각각의 새로운 클래스에 대해 개발자는 능동적으로 처리해야 하기 때문에 잘못해서 속성을 잊어버리거나 해서는 안 되는 오버라이드를 방지할 수 있다.

### 4.3.2 규칙: 인터페이스에서만 상속받을 것
#### 정의
* 상속은 오직 인터페이스를 통해서만 받아야 한다.

#### 설명
* 이 규칙은 클래스나 추상 클래스가 아닌 인터페이스에서만 상속을 해야 한다고 말한다.
* 추상 클래스를 사용하는 가장 일반적인 이유는 일부 메서드에 기본 구현을 제공하고 다른 메서드는 추상화하기 위함이다.
  * 중복을 줄이고 코드의 줄을 줄일 수 있지만 단점이 더 많다.
* 코드의 공유는 커플링을 유발한다.
* 여러 클래스에서 코드를 공유해야 하는 경우, 해당 코드를 다른 공유 클래스에 넣을 수 있다. (전략 패턴)

### 4.3.3 클래스에 있는 코드의 중복은 다 무엇일까?
* 코드 복제는 변경이 필요할 때 수정 내용을 프로그램 전체에 반영하는 방식으로 변경해야 하기 때문에 유지보수에 좋지 않다.

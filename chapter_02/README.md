# Chapter 2. 리팩터링 깊게 들여다보기

## 2.1 가독성 및 유지보수성 향상
* 리팩터링이란 기능의 동작을 변경하지 않고 더 나은 코드를 만드는 것이다.

### 2.1.1 코드 개선
* 가독성은 의도를 전달하기 위한 코드의 성질이다.
* 버그를 고치거나 기능을 추가하기 위해 일부 기능을 변경해야 할 때마다 새 코드를 어디에 놓을지 후보 위치를 조사하는 것으로 시작한다.
  * 유지보수성은 얼마나 많은 후보를 조사해야 하는지를 나타내는 표현이다.
  * 조사 단계에서 시간이 오래 걸린다는 것은 코드 유지보수성이 나쁘다는 징후이며 개선을 위해 노력해야 한다.
* 어떤 시스템에서 한군데서 수정한 결과 관련이 없는 다른 곳에서 문제가 발생하는 경우가 있는데, 그런 시스템은 취약하다고 말한다.
  * 이 취약성의 근원은 일반적으로 전역상태다. 여기서 전역은 우리가 고려한 범위를 벗어난 것을 의미한다.
  * 함께 변하는 것은 함께 있어야 한다.

### 2.1.2 코드가 하는 일을 바꾸지 않고 유지보수하기
* 값을 입력하면 리팩터링 전과 후에 동일한 결과를 얻어야 한다.
* 리팩터링 중에 코드가 느려져도 거의 신경쓰지 않는 이유
  * 대부분의 시스템에서 성능은 가독성과 유지보수성보다 가치가 떨어진다.
  * 성능이 중요한 경우 프로파일링 도구나 성능 전문가의 지도를 받아 리팩터링과 다른 단계에서 처리해야 한다.
* 리팩터링을 할 때는 블랙박스의 경계를 고려해야 한다.
  * 더 많은 코드를 포함할 수록 더 많은 것을 바꿀 수 있다. 이는 협력 시에 중요한데, 누군가가 우리가 리팩터링하는 코드를 변경하면 컨플릭트가 발생하기 때문이다.
* 리팩터링의 세가지 핵심
  1. 의도를 전달함으로써 가독성 향상
  2. 불변속성의 범위 제한을 통한 유지보수성 향상
  3. 범위 밖의 코드에 영향을 주지 않고 1항과 2항을 수행

## 2.2 속도, 유연성 및 안정성 확보
### 2.2.1 상속보다는 컴포지션 사용
* 대부분의 리팩터링 패턴과 규칙은 구체적으로 객체 컴포지션을 돕기 위한 것들이다.
```java
// 상속 이용
interface Bird {
    boolean hasBeak();
    boolean canFly();
}
class CommonBird implements Bird {
    @Override
    public boolean hasBeak() {
        return true;
    }
    @Override
    public boolean canFly() {
        return true;
    }
}
class Penguin implements CommonBird {
    boolean canFly() {
        return false;
    }
}
```
```java
// 컴포지션 이용
interface Bird {
  boolean hasBeak();
  boolean canFly();
}
class CommonBird implements Bird {
  @Override
  public boolean hasBeak() {
    return true;
  }
  @Override
  public boolean canFly() {
    return true;
  }
}
class Penguin implements Bird {
    private Bird bird = new CommonBird();
    @Override
    public boolean hasBeak() {
        return bird.hasBeak();
    }
    @Override
    public boolean canFly() {
        return false;
    }
}
```
* 위 예시에서 canSwim()이라는 새로운 메서드를 Bird에 추가하게 되면..
  * 컴포지션을 이용한 팽귄이 새로운 메서드인 canSwim()을 구현하지 않았기 때문에 컴파일 오류가 발생하여 이를 인지하고 수정할 수 있다.
  * 상속을 이용한 팽귄은 이미 CommonBird에서 canSwim()을 구현할 것이므로 팽귄의 canSwim()에 대해 인지하지 못할 수 있다.
* 컴포지션을 중심으로 만들어진 시스템을 사용하면 다른 방식보다 더 깔끔하게 코드를 결합하고 재사용 할 수 있다.

### 2.2.2 수정이 아니라 추가로 코드를 변경
* 컴포지션의 가장 큰 장점은 추가로 변경이 가능하다는 것이다.
  * 기존 기능에 영향을 주지 않고 기능을 추가하거나 변경할 수 있음을 의미한다. (어떤 경우는 기존 코드를 변경하지 않고도 가능하다.)
  * 이러한 속성을 개방-폐쇄(open-closed) 원칙이라고 한다.

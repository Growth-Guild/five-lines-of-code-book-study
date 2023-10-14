# Chapter 5. 유사한 코드 융합하기

### 5.1.1 리팩터링 패턴: 유사 클래스 통합
#### 설명
* 일련의 상수 메서드를 공통으로 가진 두 개 이상의 클래스에서 이 일련의 상수 메서드가 클래스에 따라 다른 값을 반환할 때마다 이 리팩터링 패턴을 사용해서 클래스를 통합할 수 있다.
  * 일련의 상수 메서드 집합을 기준(basis)라고 한다.
  * 일련의 상수 메서드가 두 개일 때 두 개의 접점을 가진 기준이라고 한다.
  * X개의 클래스를 통합하려면 최대 X-1 개의 접점을 가진 기준이 필요하다.
  * 클래스의 수가 적어진다는 것은 일반적으로 더 많은 구조를 발견했다는 것을 의미하므로 클래스를 통합하는 것은 좋은 것이다.
    
#### 절차
1. 모든 비기준 메서드를 동일하게 만든다. 
   * 각 메서드 버전 본문의 기존 코드 주위에 if (true) {} 를 추가한다.
   * true를 모든 기본 메서드를 호출하여 그 결과를 상수 값과 비교하는 표현식으로 바꾼다.
   * 각 버전의 본문을 복사하고 else와 함께 다른 모든 버전에 붙여 넣는다.
2. 이제 기준 메서드만 다르므로 기준 메서드에 각 메서드에 대한 필드를 도입하고 생성자에서 상수를 할당하는 것으로 시작한다.
3. 상수 대신 도입한 필드를 반환하도록 메서드를 변경한다.
4. 문제가 없는지 확인하기 위해 컴파일한다.
5. 각 클래스에 대해 한 번에 하나의 필드씩 다음을 수행한다.
   * 필드의 기본값을 매개변수로 지정하게 한다.
   * 컴파일 오류를 살펴보고 기본값을 인자로 전달한다.
6. 모든 클래스가 동일하면 통합한 클래스 중 하나를 제외한 모두를 삭제하고, 삭제하지 않은 클래스로 바꾸어 모든 컴파일러 오류를 수정한다.

#### 예제
```java
// 초기코드
class TrafficLight {
    public TrafficColor nextColor(TrafficColor t) {
        if (t.color() == "red") return new Green();
        else if (t.color() == "green") return new Yello();
        else if (t.color() == "yellow") return new Red();
    }
}
interface TrafficColor {
    String color();
    void check(Car car);
}
class Red implements TrafficColor {
    public String color() {
        return "red";
    }
    public void check(Car car) {
        car.stop();
    }
}
class Yellow implements TrafficColor {
    public String color() {
        return "yello";
    }
    public void check(Car car) {
        car.stop();
    }
}
class Green implements TrafficColor {
    public String color() {
        return "green";
    }
    public void check(Car car) {
        car.drive();
    }
}
```

1. 기준 메서드는 color인데 클래스마다 다른 상수를 반환하므로 check 메서드를 동일하게 만들어야 한다.
* 각 버전의 check 본문의 기준 코드 주위에 if (true) {} 를 추가한다.
```java
class Red implements TrafficColor {
    public void check(Car car) {
        if (true) {
            car.stop();
        }
    }
}
// ...
```
* true를 기준 메서드를 호출해서 그 결과를 상수 값과 비교하는 표현식으로 바꾼다.
```java
class Red implements TrafficColor {
    public String color() {
        return "red";
    }
    public void check(Car car) {
        if (this.color() == "red") {
            car.stop();
        }
    }
}
// ...
```
* 각 버전의 본문을 복사하여 다른 모든 버전의 본문에 else와 함께 붙여 넣는다.
```java
class Red implements TrafficColor {
    public void check(Car car) {
        if (this.color() == "red") {
            car.stop();
        } else if (this.color() == "yellow") {
            car.stop();
        } else if (this.color() == "green") {
            car.drive();
        }
    }
}
```
2. color 메서드에 대한 필드를 도입하고 생성자에서 상수를 할당한다.
```java
class Red implements TrafficColor {
    private String color;
    public Red() {
        this.color = "red";
    }
    // ...
}
```
3. 상수 대신 새로 도입한 필드를 반환하도록 메서드를 변경한다.
```java
class Red implements TrafficColor {
    // ...
    public string color() {
        return this.color;
    }
}
```
4. 문제가 없는지 확인하기 위해 컴파일한다.
5. 각 클래스에 대해 한 필드씩 다음 절차를 수행한다.
* 필드의 기본값을 매개변수로 만든다.
```java
class Red implements TrafficColor {
    private String color;
    public Red(String color) {
        this.color = color;
    }
    // ...
}
```
* 컴파일러 오류를 살펴보면서 기본값을 인자로 전달한다.
```java
class TrafficLight {
    public TrafficColor nextColor(TrafficColor t) {
        if (t.color() == "red") return new Green();
        else if (t.color() == "green") return new Yello();
        else if (t.color() == "yellow") return new Red("red");
    }
}
```
6. 모든 클래스가 동일하게 되면 통합 클래스 중 하나를 제외한 다른 모두를 삭제하고, 모든 컴파일 오류를 삭제하지 않은 클래스로 변경해서 수정한다.
```java
class TrafficLight {
    public TrafficColor nextColor(TrafficColor t) {
        if (t.color() == "red") return new Red("green");
        else if (t.color() == "green") return new Red("yellow");
        else if (t.color() == "yellow") return new Red("red");
    }
}
```
* 이 시점에서 인터페이스가 더이상 필요하지 않으며, Red 클래스의 이름을 변경해야 한다.
* else를 가진 if 문을 제거하는 작업도 진행해야 한다.

### 5.2.1 리팩터링 패턴: if 문 결합
#### 설명
* 내용이 동일한 연속적인 if 문을 결합해서 중복을 제거한다.

#### 절차
1. 본문이 실제로 동일한지 확인한다.
2. 본문이 동일하다면 if 문을 하나로 합친다.
3. 표현식이 단순하면 불필요한 괄호를 제거하거나 편집기에서 이를 수행하도록 설정할 수 있다.

```java
// 변경 전
class Foo {
    public void pay() {
        if (expression1()) {
            somthing();
        } else if (expression2()) {
            something();
        }
    }
}
```
```java
// 변경 후
class Foo {
    public void pay() {
        if (expression1() || expression2()) {
            somthing();
        }
    }
}
```

### 5.4.3 규칙: 구현체가 하나뿐인 인터페이스를 만들지 말 것
#### 정의
* 구현체가 하나뿐인 인터페이스를 사용하지 말 것.

#### 설명
* 구현체가 하나뿐인 인터페이스를 사용하지 말아야 하는 이유는 아래와 같다.
  * 구현 클래스가 하나밖에 없는 인터페이스는 가독성에 도움이 되지 않는다. 인터페이스는 변형을 전제로 하는데, 아무것도 없다면 오히려 가독성을 방해할 수 있다.
  * 구현 클래스를 수정하려는 경우 인터페이스를 수정해야 해서 오버헤드를 발생시킨다. 이 규칙은 메서드 전문화와 유사한데, 구현 클래스가 하나만 있는 인터페이스는 도움이 되지 않는 일반화의 한 형태이기 때문이다.
* 아무런 구현체가 없는 인터페이스를 갖는 것이 더 합리적인 경우가 있다.
  * 가장 일반적으로 comparator와 같은 항목에 대해 익명 클래스를 사용하거나 익명의 내부 클래스를 통해 더 엄격한 캡슐화를 수행하려는 경우 유용하다.

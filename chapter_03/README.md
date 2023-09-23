# Chapter 3. 긴 코드 조각내기
* DRY(Don't Repeat Yourself), KISS(Keep It Simple, Stupid) 지침을 따르더라도 코드가 쉽게 지저분해지고 혼란스러워질 수 있다.
  * 메서드가 여러 가지 다른 일을 수행하는 경우
  * 낮은 수준의 원시 연산을 사용하는 경우
  * 주석과 적절한 메서드와 변수명 같이 사람이 읽을 수 있는 텍스트가 부족한 경우

## 3.1 첫 번째 규칙: 왜 다섯 줄인가?
* 이 책에서 제시하는 궁극적인 목표는 어떤 메서드도 5줄 이상을 가질 수 없다는 것이다.

### 3.1.1 규칙: 다섯 줄 제한
* 메서드는 {와 }를 제외하고 5줄 이상이 되어서는 안된다.
* 문장이라고도 하는 코드 한 줄은 하나의 if, for, while 또는 세미콜론으로 끝나는 모든 것을 말한다.
* 즉, 할당, 메서드 호출, return 같은 것이다. 공백과 중괄호는 제외한다.
* 특정 수치로 줄 수를 제한하는 것보다 제한이 있다는 것 자체가 중요하다.
  * 저자의 경험상 기본 데이터 구조를 탐색하는 데 필요한 값으로 제한을 설정하는 것이 효과적이다.

#### 스멜
* 메서드가 길다는 것 자체가 스멜이다. (길다는 것은 얼마나 길어야 길다고 판단할 수 있을까?)
  * 메서드는 한 가지 작업만 해야 한다는 것을 기준으로 길다 잡아볼 수 있을 것 같다.

#### 의도
* 시간에 지남에 따라 더 많은 기능이 추가되면서 메서드가 커지는 경향이 있다. 그로 인해 코드는 점점 더 이해하기 어려워진다.
* 각 메서드의 이름으로 코드의 의도를 전달할 수 있기 때문에 각각 5줄의 코드가 있는 4개의 메서드가 20줄인 하나의 메서드보다 훨씬 빠르고 이해하기 쉽다.

## 3.2 함수 분해를 위한 리팩터링 패턴 소개
* 다섯 줄 제한 규칙은 이해하기 쉽지만 항상 지킬 수 있는 것은 아니다.
* 코드를 이해하기 위한 첫 번째 단계는 항상 함수명을 고려하는 것이다.
  * 각 줄을 이해하려다 보면 시간이 많이 걸리고 비생산적인 수렁에 빠질 위험이 있다. 그러므로 코드의 형태를 살펴보는 것으로 시작하는 것이 좋다.
* 동일한 작업을 하는 데 필요한 줄의 그룹을 식별한다.
  * 그룹을 명확히 하기 위해 그룹이라고 생각하는 곳에 빈 줄을 추가한다. 때로는 그룹과 관련된 것을 기억하는 데 도움이 되는 주석을 추가한다. (주석은 일시적인 것이다.)

### 3.2.1 리팩터링 패턴: 메서드 추출
#### 설명
* 메서드 추출은 한 메서드의 일부를 취해서 자체 메서드로 추출한다.
* if의 일부 분기만 return 문을 가지고 있을 경우 메서드를 추출하는 데 방해가 뒬 수 있으므로 메서드의 끝에서 시작해 위로 작업해가는 것이 좋다.
    * 이는 return 문을 가진 조건을 메서드의 앞쪽에 배치하게 해서 결과적으로 모든 분기에서 return 할 수 있게 된다.
#### 절차
1. 추출할 줄의 주변을 빈 줄로 표시하는데, 주석으로 표시할 수도 있다.
2. 원하는 이름으로 새로운 빈 메서드를 만든다.
3. 그룹의 맨 위에서 새로운 메서드를 호출한다.
4. 그룹의 모든 줄을 선택해서 잘라내어 새로운 메서드의 본문에 붙여 넣는다.
5. 컴파일한다.
6. 매개변수를 도입하여 호출하는 쪽의 오류를 발생시킨다.
7. 이러한 매개변수 중 하나(p)를 반환 값으로 할당해야 할 경우
    * 새로운 메서드의 마지막에 return p;를 추가한다.
    * 새로운 메서드를 호출하는 쪽에서 p = newMethod(...)와 같이 반환 값을 할당한다.
8. 컴파일한다.
9. 호출 시 인자를 전달해서 오류를 잡는다.
10. 사용하지 않는 빈 줄과 주석을 제거한다.

```java
class Game {
    private int[][] map = new int[10][10];
    private Field field = new Field();
    private Player player = new Player();
    void draw() {
        // 맵 그리기
        for (int i = 0; i < map.length; i++) {
            for (int j = 0; j < map[i].length; j++) {
                if (map[i][j] == 1) {
                    field.fillStyle = "#ccffcc";
                } else if (map[i][j] == 2) {
                    field.fillStyle = "#999999";
                } else if (map[i][j] == 3) {
                    field.fillStyle = "#0000cc";
                }
            }
        }
        field.render();
        
        // 플레이어 그리기 
        player.fillStyle = "#ff0000";
        player.render();
    }
}
```
1. 새로운 빈 메서드 drawMap을 만든다.
2. 주석이 있는 곳에서 drawMap을 호출한다.
3. 식별된 그룹의 모든 줄을 선택한 다음 잘라내여 drawMap의 본문을 붙여 넣는다.

```java
class Game {
    private int[][] map = new int[10][10];
    private Field field = new Field();
    private Player player = new Player();
    public void draw() {
        drawMap();
        drawPlayer();
    }
    
    public void drawMap() {
        for (int i = 0; i < map.length; i++) {
            for (int j = 0; j < map[i].length; j++) {
                if (map[i][j] == 1) {
                    field.fillStyle = "#ccffcc";
                } else if (map[i][j] == 2) {
                    field.fillStyle = "#999999";
                } else if (map[i][j] == 3) {
                    field.fillStyle = "#0000cc";
                }
            }
        }
        field.render();
    }
    
    public void drawPlayer() {
        player.fillStyle = "#ff0000";
        player.render();
    }
}
```
* 위 코드는 메서드 추출이라고 부르는 표준 패턴을 통한 결과물이다.
* 이 방법은 코드 줄만 이동하기 때문에 오류가 발생할 위험이 최소화된다.
* 매개변수를 잊었을 경우에는 컴파일러를 통해 알 수 있다.

## 3.3 추상화 수준을 맞추기 위한 함수 분해
### 3.3.1 규칙: 호출 또는 전달, 한 가지만 할 것
#### 정의
* 함수 내에서는 객체에 있는 메서드를 호출하거나 객체를 인자로 전달할 수 있지만 둘을 섞어 사용해서는 안 된다.

#### 설명
* 더 많은 메서드를 도입하고 여러 가지를 매개변수로 전달하기 시작하면 결국 책임이 고르지 않게 될 수 있다.
* 추상화 정도가 높은 수준의 작업과 낮은 수준의 작업이 공존하게 되면 가독성이 떨어질 수 있다.
  * 동일한 수준의 추상화를 유지하라.
```java
// 높은 수준의 추상화 sum(numbers)와 낮은 수준의 numbers.length를 모두 사용하고 있다.
class Math {
    public double average(int[] numbers) {
        return sum(numbers) / numbers.length;
    }
}
```
```java
// 배열의 길이를 찾는 것을 추상화하여 동일한 추상화 수준을 유지하도록 하고 있다.
class Math {
    public double average(int[] numbers) {
        return sum(numbers) / size(numbers);
    }
}
```

#### 스멜
* '함수의 내용은 동일한 추상화 수준에 있어야 한다'는 말은 그 자체가 스멜일 정도로 강력하다.

#### 의도
* 메서드에서 몇 가지 세부적인 부분을 추출해서 추상화를 도입할 때 이 규칙은 연관된 다른 세부적인 부분도 추출하게 된다. (메서드 내부의 추상화 수준이 항상 동일하게 유지된다.)

## 3.4 좋은 함수 이름의 속성
* 좋은 함수 이름이 가져야할 속성
  * 함수의 의도를 설명해야 함.
  * 함수가 하는 모든 것을 담아야 함.
  * 도메인에서 일하는 사람이 이해할 수 있어야 함.

## 3.5 너무 많은 일을 하는 함수 분리하기
### 3.5.1 규칙: if문은 함수의 시작에만 배치
#### 정의
* if 문이 있는 경우 해당 if문은 함수의 첫 번째 항목이어야 한다.

#### 설명
* 무언가를 확인한다는 것은 한 가지 일이다. 따라서 함수에 if가 있는 경우 함수의 첫 번째 항목이어야 한다.
  * 또한 그 이후에 아무것도 해서는 안된다는 의미에서 유일한 것이어야 한다.
* if 문이 메서드가 하는 유일한 일이어야 한다는 말은 곧 그 본문을 추출할 필요가 없으며, 또한 else 문과 분리해서는 안 된다는 말이다.
  * 본문과 else는 모두 코드 구조의 일부이며 이 구조에 의존해서 작업하면 코드를 이해할 필요가 없다.
  * 동작과 구조는 밀접하게 연결되어 있으며 리팩터링할 때 동작을 변경해서는 안 되므로 구조도 변경해서는 안 된다.

#### 스멜
* 이 규칙은 함수가 한 가지 이상의 작업을 수행하는 스멜을 막기 위함이다.

#### 의도
* if 문이 하나의 작업이기 떄문에 이를 분리할 때 이어지는 else if는 if 문과 분리할 수 없는 원자 단위로 본다.

```java
class Game {
    private int[][] map = new int[10][10];
    private Field field = new Field();
    private Player player = new Player();
    public void draw() {
        drawMap();
        drawPlayer();
    }
    
    public void drawMap() {
        for (int i = 0; i < map.length; i++) {
            for (int j = 0; j < map[i].length; j++) {
                updateStyle(j, i);
            }
        }
        field.render();
    }
    
    private void updateStyle(int x, int y) {
        if (map[y][x] == 1) {
            field.fillStyle = "#ccffcc";
        } else if (map[y][x] == 2) {
            field.fillStyle = "#999999";
        } else if (map[y][x] == 3) {
            field.fillStyle = "#0000cc";
        }
    }
    
    public void drawPlayer() {
        player.fillStyle = "#ff0000";
        player.render();
    }
}
```
* updateStyle()를 살펴보면 if 문을 메서드 추출을 통해 한 가지 일만 하도록 했지만 다섯 줄 제한 규칙을 준수하지 못했다.
  * if 문은 이미 한 줄이고 이어진 else if 를 분리할 수 없기 때문에 메서드 추출을 적용할 수 없다.
  * 이 또한 이후에 살펴볼 장에서 해결책이 있다.

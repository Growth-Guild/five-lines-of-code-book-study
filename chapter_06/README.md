# Chapter 6. 데이터 보호
- - -

## 6.1 getter 없이 캡슐화하기
### 6.1.3 리팩터링 패턴: getter와 setter 제거하기
#### 설명
* 기능을 데이터에 더 가깝게 이동하여 getter와 setter를 제거할 수 있다.
* 필드를 비공개로 하는 것의 가장 큰 장점은 푸시 기반의 아키텍처를 장려한다는 것이다.
  * 푸시 기반 아키텍처에서는 가능한 한 데이터에 가깝게 연산을 이관하지만, 풀 기반의 아키텍처에서는 데이터를 가져와 중앙에서 연산을 수행한다.
  * 풀 기반의 아키텍처에서는 기능을 수행하는 메서드 없이 수많은 수동적인 데이터 클래스와 여기저기서 데이터를 혼합해서 모든 작업을 수행하는 소수의 관리자 클래스로 이어진다.
  * 관리자 클래스의 문제는 데이터 클래스와 관리자 사이에 밀 결합을 가져온다는 점이다.

#### 절차
1. getter 또는 setter가 사용되는 모든 곳에서 오류가 발생하도록 비공개로 설정한다.
2. 클래스로의 코드 이관으로 오류를 수정한다.
3. getter 또는 setter는 클래스로의 코드 이관의 일부로 인라인화 되므로, 삭제하여 다른 사람이 사용하지 않게 한다.

#### 예제

```java
// 초기 코드
class Website {
    private String url;

    public Website(String url) {
        this.url = url;
    }

    public String getUrl() {
        return url;
    }
}
class User {
    private String username;

    public User(String username) {
        this.username = username;
    }

    public String getUsername() {
        return username;
    }
}
class BlogPost {
    private User author;
    private String id;
    public BlogPost(User author, String id) {
        this.author = author;
        this.id = id;
    }
    public String getId() {
        return id;
    }
    public User getAuthor() {
        return author;
    }
}
class Main {
    public String generatePostLink(Website website, BlogPost blogPost) {
        return website.getUrl() + blogPost.getAuthor().getUsername() + blogPost.getId();
    }
}
```

1. getter가 사용되는 모든 곳에서 오류가 발생하도록 getter를 비공개로 바꾼다.
```java
class BlogPost {
    private User author;
    private String id;
    public BlogPost(User author, String id) {
        this.author = author;
        this.id = id;
    }
    public String getId() {
        return id;
    }
    private User getAuthor() {
        return author;
    }
}
// ...
```
2. 클래스로의 코드 이관으로 오류를 수정한다.
```java
class Main {
    public String generatePostLink(Website website, BlogPost blogPost) {
        return website.getUrl() + blogPost.getAuthorName() + blogPost.getId();
    }
}
class BlogPost {
  private User author;
  private String id;
  public BlogPost(User author, String id) {
    this.author = author;
    this.id = id;
  }
  public String getId() {
    return id;
  }
  private User getAuthor() {
    return author;
  }
  public String getAuthorName() {
      return this.author.getUsername();
  }
}
```
3. getter는 클래스로의 코드 이관의 일부로 인라인화 되었으므로 삭제하여 다른 사람이 사용하지 않게 한다.

### 6.2.3 리팩터링 패턴: 데이터 캡슐화
#### 설명
* 변수와 메서드를 캡슐화해서 접근할 수 있는 지점을 제한하고 구조를 명확하게 만들 수 있다.
* 메서드를 캡슐화하면 이름을 단순하게 만들고 응집력을 더 명확하게 하는 데 도움이 된다.
* 자연스럽게 더 많은 수의 클래스를 만들게 되는 데, 클래스를 만드는 데 너무 소극적일 필요 없다.
* 여러 위치에서 클래스의 데이터에 직접 접근할수록 유지보수가 어려워지므로, 캡슐화하여 범위를 제한해야한다.

#### 절차
1. 클래스를 만든다.
2. 변수를 새로운 클래스로 이동하고 private로 바꾼다. 변수의 이름을 단순한 것으로 정하고 변수에 대한 getter와 setter를 만든다.
3. 변수가 더 이상 전역 범위에 없기 때문에 컴파일러 오류를 발생시켜 모든 참조를 찾을 수 있게 한다.

#### 예제
```java
// 초기 코드
class Main {
  public static int counter = 0;
  public static void main(String[] args) {
      for (int i = 0; i < 20; i++) {
          incrementCounter();
          System.out.println(counter);
      }
  }
  public static void incrementCounter() {
      counter++;
  }
}
```

1. 클래스를 만든다.
```java
class Counter{}
```

2. 변수를 새로운 클래스로 이동하고 private로 변경한다. 변수의 이름을 단순화하고 변수에 대한 getter와 setter를 만든다.
```java
class Counter {
    private int counter = 0;
    public int getCounter() {
        return counter;
    }
    public void setCounter(int counter) {
        this.counter = counter;
    }
}
```
3. counter가 더 이상 전역범위에 없기 때문에 컴파일러는 참조하는 모든 지점에서 오류를 발생시킨다. 다음 다섯 단계에 따라 오류를 수정한다.
* 새로운 클래스의 인스턴스에 적합한 변수명을 counter로 선택한다.
* 인스턴스의 getter 또는 setter로 데이터에 접근하는 방식을 변경한다.
```java
class Main {
    public static void main(String[] args) {
      for (int i = 0; i < 20; i++) {
            incrementCounter();
            System.out.println(counter.getCounter());
        }
    }
    public static void incrementCounter() {
        counter.setCounter(counter.getCounter() + 1);
    }
}
```
* 2개 이상의 다른 메서드에 오류가 있는 경우 이전의 변수명을 가진 매개변수를 첫 번째 매개변수로 추가하고 동일한 변수를 첫 번째 인자로 호출하는 측에 넣는다.
```java
class Main {
    public static void main(String[] args) {
      for (int i = 0; i < 20; i++) {
            incrementCounter(counter);
            System.out.println(counter.getCounter());
        }
    }
    public static void incrementCounter(Counter counter) {
        counter.setCounter(counter.getCounter() + 1);
    }
}
```
* 한 메서드에서만 오류가 발생할 때까지 반복한다.
```java
class Main {
    public static void main(String[] args) {
      Counter counter = new Counter();
      for (int i = 0; i < 20; i++) {
            incrementCounter(counter);
            System.out.println(counter.getCounter());
        }
    }
    public static void incrementCounter(Counter counter) {
        counter.setCounter(counter.getCounter() + 1);
    }
}
```

### 6.4.1 리팩터링 패턴: 순서 강제화
#### 설명
* 무언가가 다른 것보다 먼저 호출되어야 할 때, 그것을 순서 불변속성(sequence invariant)라고 한다.
* 객체지향 언어에서는 생성자가 항상 객체의 메서드보다 먼저 호출되어야 하는데, 이를 이용해서 작업이 특정 순서로 발생하게 할 수 있다.
* 이 패턴을 수행한 후에는 순서가 적용되기 때문에 실행 순서에 대한 불변속성은 제거된다.

#### 절차
1. 마지막으로 실행되어야 하는 메서드에 데이터 캡슐화를 적용한다.
2. 생성자가 첫 번째 메서드를 호출하도록 한다.
3. 두 메서드의 인자가 연결되어 있으면 이러한 인자를 필드로 만들고 메서드를 제거한다.

#### 예제
```typescript
// 초기 코드
function deposit(
    to: string,
    amount: number
) {
  let accountId = database.find(to);
  database.updateOne(
      accountId,
      { $inc: { amount } }
  );
}
```

1. 마지막으로 실행되어야 하는 메서드에 데이터 캡슐화를 적용한다
```typescript
class Transfer {
    deposit(to: string, amount: number) {
        let accountId = database.find(to);
        database.updateOne(
            accountId,
            { $inc: { amount } }
        );
    }
}
```

2. 생성자에서 첫 번째 메서드를 호출하도록 한다.
```typescript
class Transfer {
  constructor(from: string, amount: number) {
      this.deposit(from, -amount);
  }

  deposit(to: string, amount: number) {
    let accountId = database.find(to);
    database.updateOne(
      accountId, 
      { $inc: { amount } }
    );
  }
}
```

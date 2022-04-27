## 동작 파라미터화 코드 전달하기
동작 파라미터화: 아직 어떻게 실행할 것인지 결정하지 않은 코드 블록

### 2.1 변화하는 요구사항에 대응하기
#### 첫번째 시도 : 녹색사과 필터링

```java
enum Color { RED, GREEN }
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
	    if (GREEN.equals(apple.getColor()) { result.add(apple); }
	}
	return result;
}
```
문제점: 다양한 색 필터링 변화에 적절하게 대응할 수 없다.
해결책: 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.

#### 두번째 시도: 색을 파라미터화
방법: 메서드에 파라미터를 추가하면 된다.
```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory) {
        if (apple.getColor().equals(color)) { result.add(apple); }
    }
    return result;
}
```
그런 다음, 색깔별로 메서드에 파라미터를 변경해서 구현했다.
```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```
하지만, 여기서 이런 요구사항이 발생한다면 어떻게 될까?
"색 이외에도 가벼운 사과와 무거운 사과로 구분할 수 있다면 좋겠네요!"
그러면 filterApplesByWeight라는 메서드를 하나 만들면 된다. 아래와 같이.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory) {
        if (apple.getWeight() > weight) { result.add(apple); }
    }
    return result;
}
```
문제점: 중복되는 코드가 많다. 소프트웨어 공학의 DRY (같은 것을 반복하지 말 것) 원칙을 어기게 된다.

#### 세번째 시도 : 가능한 모든 속성으로 필터링
```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory) {
        if ((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) { result.add(apple); }
    }
    return result;
}
```
```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```
문제점: true, false 의미하는 바 파악 어려움, 요구사항에 유연한 대처 불가능.

### 2.2 동작 파라미터화
```java
public interface ApplePredicate { boolean test (Apple apple); }
```
```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}
```
```java
public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}
```
문제점: filterApples에서 ApplePredicate 객체를 받아서 애플의 조건을 검사하도록 메서드를 고쳐야 한다.
동작 파라미터화 : 메서드가 다양한 동작(전략)을 받아서 내부적으로 다양한 동작을 수행할 수 있다.
```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory) {
        if (p.test(apple)) { result.add(apple); }
    }
    return result;
}
```
장점 : 코드, 동작 전달 가능, 가독성 좋아졌고 사용하기 편리함, 요구사항 변경 시 ApplePredicate를 적절하게 구현하는 클래스를 만들면 된다. 즉 filterApples의 동작을 파라미터화 한 것이다.
한 메서드가 다른 동작을 수행하도록 재활용할 수 있다.

문제점: 여러 클래스를 구현해서 인스턴스화하는 과정이 조금 거추장스럽게 느껴질 수 있다.

### 2.3 복잡한 과정 간소화
#### 다섯번째 시도 : 익명클래스 사용
익명 클래스 : 자바의 지역 클래스(local class)와 비슷한 개념이다.
```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
})
```
문제점: 공간차지가 많다. 많은 프로그래머가 익명 클래스 사용에 익숙하지 않다. 

#### 여섯번째 시도 : 람다 표현식 사용
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

#### 일곱번째 시도 : 리스트 형식으로 추상화
```java
public interface Predicate<T> { boolean test(T t); }

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for (T e: list) {
        if (p.test(e)) {
            result.add(e);
        }
    }
    return result;
}
```

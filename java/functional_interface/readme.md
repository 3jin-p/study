Funtional Interface
---

### 함수형 프로그래밍 
함수형 프로그래밍은 같은 입력에 대해 항상 같은 출력을 반환하는 순수 함수를 조합하요 공유 상태, 변경 가능한 데이터 및 부작용을 피하는 기본 원칙에 따라 소프트웨어를 구성하는 프로그래밍 패러다임.
간결한 코드와 부수효과를 낳지 않는 코드에 목적이 있음.  

### Lamda

#### Lamda?
이전에는 어떠한 행위를 수행하려면 무조건 클래스를 생성하여야 했었지만 람다가 생긴 이후 이 패러다임을 깰 수 있게됬다.   
Java8 부터 지원되는 **익명함수** 로 메서드의 매개변수로 전달되거나, 변수에 저장될 수 있는 함수이다.  
가독성을 증대시키고, 불필요한 코드를 줄일 수 있다.

#### 표현식
``` java
() -> {}
() -> 1 , () -> { return 1; }
() -> { if (is) { return 1; } else { return 2; } }

(int x) -> x + 1
(x) -> x + 1
x -> x + 1

...
```

### Functional Interface

#### Functional Interface 란?
단일 추상 메서드만이 포함된 인터페이스를 가르킵니다. 그로인해 함수형 인터페이스는 단 하나의 기능만 표상.  

#### @FunctionalInterface
@FunctionalInterface 어노테이션은 명시적으로 함수형 인터페이스임을 표시.  
해당 어노테이션이 선언되어 있는 인터페이스가 다른 인터페이스를 확장하면, 컴파일타임에서 찾아낼 수 있음.

#### 대표적인 Functional Interface

#### Consumer\<T>
입력된 T 타입에 대해 작업을 수행하고 싶지만 반환값이 없을때 사용.

- 구현
``` java
 @FunctionalInterface
  public interface Consumer<T> {
    void accept(T t);
     
     // 다수의 Consumer 을 체이닝 해서 수행 순서를 결정하는 메서드.
     default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
  }
```

- example
``` java
public static void main(String[] args) {
    Consumer<String> consumer = s -> System.out.println(s.toUpperCase());
    Consumer<String> consumer2 = s -> System.out.println(s + "!!");
    
    consumer.accept("hello world"); // -> HELLO WORLD
    consumer.accept("hello java"); // -> HELLO JAVA
    consumer.andThen(consumer2).accept("hellow world") // -> HELLO WORLD!!
}
```  

#### Function<T, R> 
입력된 T 타입에 대해 작업을 수행하고, R 타입을 반환하고 싶을때 사용.
- 구현
``` java
@FunctionalInterface
  public interface Function<T, R> {
    R apply(T t);
    
    // 매개변수로 받은 before 함수를 수행한 후 자신을 실행시킵니다.
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    
    // 자신을 실행 시킨 후 매개 변수로 받은 after 함수를 실행시킵니다. 
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    
    // 자신의 값을 그대로 리턴
    static <T> Function<T, T> identity() {
      return t -> t;
    }
  }
```
- example
``` java
public static void main(String[] args) {
    Function<Integer, String> intToStr = i -> String.valueOf(i);
    Function<String, Integer> strToInt = s -> Integer.parseInt(s);
    
    intToStr.apply(1); // -> "1"
    intToStr.compose(strToInt).apply("1"); // -> 1
    intToStr.andThen(strToInt).apply(1); // -> "1"
}
```

#### Predicate\<T>
T 타입을 받아 특정 조건에 부합하는지 boolean 을 반환하고 싶을때 사용.

- 구현
``` java
@FunctionalInterface
public interface Predicate<T> {

  boolean test(T t);
  
  // 두개의 Predicate 룰 && 연산
  default Predicate<T> and(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) && other.test(t);
  }
  
  // 자신의 연산을 ! 연산
  default Predicate<T> negate() {
    return (t) -> !test(t);
  }
   
  // 두개의 Predicate 를 or 연산
  default Predicate<T> or(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) || other.test(t);
  }

  static <T> Predicate<T> isEqual(Object targetRef) {
    return (null == targetRef)
      ? Objects::isNull
        : object -> targetRef.equals(object);
  }
}
```

- example

``` java
public static void main(String[] args) {
    Predicate<Integer> lessThanTen = i -> i < 10
    Predicate<Integer> greaterThanFive = i -> i > 5
    
    lessThanTen.and(greaterThanFive).test(6) // -> true
    lessThanTen.and(greaterThanFive).test(3) // -> false
    
    lessThanTen.or(greaterThanFive).test(6) // -> true
    lessThanTen.or(greaterThanFive).test(3) // -> true
    
    lessThanTen.negate().test(9) // -> false
}
```

#### Supplier\<T>
매개 변수 없이 특정 타입을 반환해야할때 사용.

- 구현
``` java
@FunctionalInterface
public interface Supplier<T> {
  T get();
}
```

- example
``` java
public static void main(String[] args) {
    // 주사위 번호 하나 뽑기 (1~6)
    Supplier<Integer> dice = () -> (int) (Math.random() * 6) + 1; 
}
// Optional 의 orElseThrow
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
    if (value != null) {
        return value;
    } else {
        throw exceptionSupplier.get();
    }
}
```


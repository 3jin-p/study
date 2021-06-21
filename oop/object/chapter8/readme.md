OBJECT - Chater 8 의존성 관리하기
--

### 변경과의존성

어떤 객제가 협력하기 위해 다른 객체를 필요로 할 때 두 객체 사이에 의존성이 존재하게 된다.  
의존성은 실행 시점과 구현 시점에 서로 다른 의미를 가진다.

- 실행 시점: 의존하는 객체가 정상적으로 동작하기 위해서는 실행 시에 의존 대상 객체가 반드시 존재해야 한다.
- 구현 시점: 의존 대상 객체가 변경될 경우 의존하는 객체도 함께 변경된다.

의존성을 가진다는것은 의존의 대상이 되는 객체들의 변경사항이 의존하는 객체에도 영향을 미친다는 뜻이다.

### 의존성 전이

의존성은 전이될 수 있다.

A -> B B -> C 의 의존관계가 있으면 A는 C와도 간접적으로 의존관계가 있을 수 있다.

의존성은 함께 변경될 수 있는 가능성을 의미하기 때문에 모든 경우에 의존성이 전이되는 것은 아나다.  
의존성이 실제로 전이될지 여부는 변경의 방향과 캡슐화의 정도에 따라 달라진다.   
즉 캡슐화란 의존성의 전이를 막는것이기도 하다.

### 런타임 의존성과 컴파일 타임 의존성

런타임 의존성이 다루는 주제는 객체 사이의 의존성이다.   
반면 코드 관점에서 주인공은 클래스다.

컴파일타임과 런타임의 의존성이 멀면 멀수록 설계가 유연해지고 재사용 가능하다.  
추상화의 정도가 높을 수록 컴파일 타임과 런타임의 의존성이 다르다. ex) Interface


### 의존성해결하기

컴파일타임 의존성은 구체적인 런타임 의존성으로 대체돼야 한다. 이를 의존성 해결이라고 한다.

**의존성 해결 방법.**
1. 객체를 생성하는 시점에 생성지를 통해 의존성 해결 
2. 객체 생성 후 setter 메서드들 통해 의존성 해결
3. 메서드 실행 시 인자를 이용해 의존성 해결  

의존성의 존재는 문제가 아니다 문제는 의존성의 정도이고
재사용 가능성을 열어놓는 의존성, 낮은 결합도를 지니는 의존성을 바람직한 의존성이라고 한다.

### 지식이 결합을 낳는다

결합도의 정도는 한 요소가 자 신이 의존하고 있는 다른 요소에 대해 알고 있는 정보의 양으로 결정된다.   
한 요소가 다른 요소에 대해 더 많은 정보를 알고 있을수록 두 요소는 강하게 결합된다.  

**모든 정보를 은닉하라. 구체적인 세부사항을 모르게 추상화하라.**

### 명시적인 의존성

``` java
public class Movie {

    private DiscountPolicy discountPolicy;
    
    public Movie(String title, Duration runningTime, Money fee) {
        this.discountPolicy = new AmountDiscountPolicy(...)
    }
}
```

예제와 같이 생성자 내부에서 의존하는 객체를 직접 생성하는 방법 대신,  
위에서 이야기한 3가지 의존성 해결 방법을 이용하여 외부에서 의존 객체를 받아오자.

해당 방법을 사용하면 의존 객체가 명시적으로 퍼블릭 인터페이스에 노출된다. 이를 명시적인 의존성이라고 한다.  
**의존성은 명시적으로 표현돼야 한다. 의존성을 구현 내부에 숨겨두지 마라**  
명시적인 의존성을 사용해야만 퍼블릭 인터페이스를 통해 컴파일타임 의존성을 적절한 런타임 의존성으로 교체할 수 있다.


### new는해롭다

new 를 잘못사용하면 결합도가 극단적으로 높아진다.

- new 연산자를 사용하기 해서는 구체 클래스의 이륨을 직접 기술해야 한다. 따라서 new를 사용하는 클라이언트는 추상화 가 아닌 **구체 클래스에 의존할 수밖에 없기 때문에** 결합도가 높아진다.
- new 연산자는 생성하려는 구체 클래스뿐만 아니라 어떤 인자를 이용해 클래스의 생성자를 호출해야 하는지도 알아야 한 다 따라서 new를 사용하면 클라이언트가 알아야 하는 **지식의 양이 늘어나기 때문에** 결합도가 높아진다.


**해결 방법**  
인스턴스를 생성하는 로직과 생성된 인스턴스를 사용하는 로직을 분리하는 것!  
생성은 해당 클래스를 사용하는 클라이언트에게 맡겨라
> 클래스 안에서 객체의 인스턴스를 직접 생성하는 방식이 유용한 경우도 있다. 주로 협력하는 기본 객체 를 설정하고 싶은 경우가 여기에 속한다



### 표준 클래스에 대한 의존은 해롭지 않다

의존성이 불편한 이유는 그것이 항상 변경에 대한 영향을 암시하기 때문이다. 따라서 변경될 획률이 거의 없는 클래스라면 의존성이 문제가 되지 않는다.  
항상 잊지말자 무조건적으로 원칙을 따르는 것은 오히려 독이다.


유연하고 재사용 가능한 설계는 응집도 높은 책임들을 가진 작은 객체들을 다잉한 방식으로 연결함으로써 애플리케이션의 기능을 쉽게 확장할수 있다!  
How 가 아닌 What 에 집중하도록 구현하자

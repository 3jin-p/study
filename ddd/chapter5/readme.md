DDD - 소프트웨어에서 표현되는 모델
--

수많은 실세계의 복잡한 연관관계를 쉽게 다루는 법에는 다음과 같은 방법들이 있다.

1. 탐색방향을부여한다
2. 한정자를 추가해서 사실상 다중성을 줄인다 
3. 중요하지 않은 연관관계를 제거한다

가능한 한 관계를 제약하는 것이 중요하다.  
다대다 연관관계의 탐색 방향을 제약하면 해당 연관관계는 사실상 훨씬 더 구현하기 쉬운 일 대다 연관관계로 줄어든다.  
물론 당면한 문제에 필요한 것이 아니거나 중요한 의미를 담고 있는 모델 객체가 아니라면 궁극적인 단순화는 연관관계를 완전히 제거하는 것이다.  

이러한 방법들을 통해 Model Driven Design 을 위해 투영하는 시스템 요소들을 살펴보자  

## ENTITY
어떤 객체를 일차적으로 해당 객체의 식별성으로 정의할 경우 그 객체를 ENTITY라 한다.  

즉 **ENTITY 란**
1. 자신의 생명주기 동안 형태와 내용이 급격하게 바뀔 수도 있지만 **연속성은 유지해야 한다**  
2. 한 객체가 **속성보다는 식별성으로 구분되는 것** 

ENTITY 에서 가장 중요한 것은 식별성이다. 이를 객체의 주된 정의로 삼아라.  
클래스 정의를 단순하게 하고 생명주기의 연속성과 식별성에 집중하라.  
객체의 형태나 이력에 관계없이 각 객체를 구별하는 수단을 정의하라.  
**-> ENTITY 는 최 우선적으로 식별성과 연속성을 우선시 하여 분류하고 설계하라**  

### ENTITY 를 구성하는 방법
1. ENTITY의 속성이나 행위에 집중하기보다는 ENTITY 객체를 해당 ENTITY 객체의 가장 본질적인 특징 특히 해당 ENTITY를 식별하고 탐색하며 일치시키는 것만으로 정의한다.  
2. 개념에 필수적인 행위만 추가하고 그 행위에 필요한 속성만 추가한다.  
3. 그 밖의 객체는 행위와 속성을 검토해서 가장 중심이 되는 ENTITY와 연관관계에 있는 다른 ENTITY 혹은 VALUE OBJECT 등에 옮긴다.
4. 식별성 문제를 제외하면 ENTITY는 주로 자신이 소유한 객체의 연산을 조율해서 책임을 완수한다.  


## VALUE OBJECT
개념적 식별성이 없는 객체도 많은데, 이러한 객체는 사물의 어떤 특징을 묘사한다.  
개념적 식별성을 갖지 않으면서 도메인의 서술적 측면을 나타내는 객체를 VALUE OBJECT라 한다.  
VALUE OBJECT는 **어느 것인지에 대해서는 관심이 없고 오직 해당 요소가 무엇인지**에 대해서만 관심이 있는 요소를 인스턴스화 하는 객체이다.  

#### VALUE OBJECT 를 구성하는 방법
1. 모델에 포함된 어떤 요소의 **속성에만 관심이 있다면** 그것을 VALUE OBJECT로 분류하라. 
2. VALUE OBJECT에서 해당 VALUE OBJECT가 전하는 속성의 의미를 표현하게 하고 관련 기능을 부여하라. 
3. VALUE OBJECT는 **불변적**으로 다뤄라.
4. VALUE OBJECT에 는 **아무런 식별성도 부여하지 말고** ENTITY를 유지하는 데 필요한 설계상의 복잡성을 피하라.


## SERVICE
설계가 매우 명확하고 실용적이더라도 개념적으로 어떠한 객체에도 속하지 않는 연산이 포함될 때가 있다.  
이럴때 사용하는 설계 요소를 SERVICE 라고 한다.  
이러한 문제를 억지로 해결하려 하기보다는 문제에 따라 SERVICE를 모델에 명시적으로 포함할 수 있다.  

#### SERVICE 의 특징
1. 연산이 원래부터 ENTITY나 VALUE OBJECT의 일부를 구성하는 것이 아니라 도메인개념과관련돼 있다.
2. 인터페이스가 도메인 모델의 외적 요소의 측면에서 정의된다.
3. 연산이상태를 갖지않는다.

SERVICE는 상태를 갖지 않게 만들어라.  
SERVICE의 매개변수와 결과는 도메인 객체여야 한다.  
도메인 계층에서만 이용되는 것은 아니다. 도메인 계층에 속하는 SERVICE와 다른 계층에 속하 는 것들을 구분하고,  
그러한 구분을 분명하게 유지하는 책임을 나누는 데 주의를 기울여야 한다.


## MODULE
여러 서버에 코드를 분산하는 것이 실제로 의도했던 바가 아니라면 동일한 객체는 아니더라도 하나의 개념적 객체를 구현하는 코드는 모두 같은 MODULE에 둬야 한다.
패키지화를 바탕으로 다른 코드로부터 도메인 계층을 분리하라.  
그렇게 할 수 없다면 가능한 한 도메인 개발자가 자신의 모델과 설계 의사결정을 지원하는 형태로 도메인 객체를 자유로이 패키지화할 수 있게 하라.
JAVA Validation
--
Bean Validation 2.0 명세를 따르는 객체의 유효성 검사 
Bean Validation 2.0 : [https://beanvalidation.org/2.0-jsr380/](https://beanvalidation.org/2.0-jsr380/)

여러 레이어에서 필요한 데이터 검증로직을 하나의 도메인 모델로 묶어 처리하여 오류 발생빈도와 성능향상을 도모함. 
주로 DTO 객체에 적용하여 사용한다.

**Dependencies**

1. Validation API 

```jsx
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.0.Final</version>
</dependency>
```

2. Validation API Reference Implementation
3. 
> HibernateValidator 는 기본적인 명세를 따르는 javax.validation 에서 제공하는 기능 외에 더 많은 검증로직이 구현되어 있다.
> validation api를 참조한 구현체, ORM Hibernate 와는 연관이 없다.

```jsx
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.2.Final</version>
</dependency>
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator-annotation-processor</artifactId>
    <version>6.0.2.Final</version>
</dependency>
```

3. Spring Boot Validation Starter 

> Spring boot web 모듈은 Hibernate Validator를 포함하고 있다.

```jsx
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 1.기본적인 javax.validation 어노테이션

 

- **@NotNull** – Null 이 아님을 검증
- **@AssertTrue** – True 값을 지님을 검증
- **@Size** – property 의 길이를 검증 *String*, *Collection*, *Map*, *Array* 프로퍼티에만 적용가능
- **@Min** – 해당 값보다 적을수 없음을 검증
- **@Max** – 해당 값보다 클 수 없음을 검증
- ***@Email*** – 이메일 형식을 갖췄는지 검증
- ***@NotEmpty*** – Null 이 아님과 비어있는지 여부를 검증 *String*, *Collection*, *Map* *Arrav* 에 적용 가능 
- ***@NotBlank*** – Null 이 아님과 공백이 아님을 검증
- ***@Positive*** and ***@PositiveOrZero*** – 양수임을 검증 그리고 양수 와 0 임을 검증 숫자에만 적용가능
- ***@Negative*** and ***@NegativeOrZero*** – 음수임을 검증 그리고 음수 와 0 임을 검증 숫자엠나 적용가능
- ***@Past*** and ***@PastOrPresent* – 시간이 특정시간 이전임을 검증, 특정 시간 이전과 현재를 검증 Java8 이후 부터 추가된 날짜 객체들에 적용가능
- ***@Future** and **@FutureOrPresent*** – 시간이 특정시간 이후임을 검증, 특정 시간 이후와 현재를 검증

### 2. 이외 HibernateValidator 에서 추가로 구현되어있는 어노테이션 참고자료
[https://www.baeldung.com/hibernate-validator-constraints](https://www.baeldung.com/hibernate-validator-constraints)

> 각 어노테이션은 필드단위 외에도 Collection의 엘리먼트도 검증,  @ScriptAssert를 이용한 클래스단위 검증이 가능하다
---

# 사용
### 1. Controller로 들어오는 객체의 검증
```java

@RequestMapping(value = "/test/valid", method = RequestMethod.POST)
public String testValidating(@Valid @RequestBody ValidationObject validationObject, BindingResults bindingResults) throws Exception {  
	if (bindingResult.hasErrors()) { ... }
        ...
}
```
@Valid 가 붙은 프로퍼티들을 검증한다. BindingResult로 핸들링이 가능하다.
BindingResult 에 담지 않는다면 MethodConstraintViolationException 를 던지므로 
@RestControllerAdvice 에서 ExceptionHandler 를 구현하여 처리하여도 된다.

### 2. Controller가 아닌 레이어에서의 검증

```java
@Validated // <<<<< Required
@Service
public class ValidationService{
    public void doValid(@Valid ValidationObject validationObject ) { 
        ... // -> 해당 메서드가 실행될 때 검증로직이 수행된다
    }
}
```
Controller Layer 가 아니라면 클래스에 @Validated 어노테이션을 붙여주어야한다. 
아마 어플리케이션 곳곳이 분포되어 있고 중복적으로 체크하는 검증 로직을 도메인 레이어로 통합시키고자 하는 Bean Validation 의 명세에 
어긋나므로 Layer 간 중복 검증을 지양할 시간을 주는것이 아닐까 생각된다.

### 3. Collection Element 의 검증

```java
public class ValidationObject {
    @Min(1) // Min -> Validation For Collection
    private Collection<@NotBlank String> ids; // NotBlank -> Validation For Elements
}
```

### 4. Custom Annotation

```java
@Constraint(validatedBy = NotIncludeStrValidator.class)
@Target(FIELD)
@Retention(RUNTIME)
public @interface NotIncludeStr {
    String message() default "contains constraint str";
    
    String constraint() default "";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}

public class NotIncludeStrValidator implements ConstraintValidator<NotIncludeStr, String> {
    
    private String constraint;

    public void init(NotIncludeStr annotation) { this.constraint = annotation.constraint(); }
    
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return false;
        }
        return !value.contains(constraint);
    }
}

public class Person {
   @NotIncludeStr(constraint = "철수") // 아들아.. 손주 이름은 철수만큼은 안된다.
   private String name;
}
```

---

이외에도 Marker Interface를 이용한 Grouping 등의 기능이 가능하다.

## Conclusion

Validation Library가 제공하는 커스텀 어노테이션, 그룹핑 등을 적극 활용하면 

별도의 반복되는 검증로직 작성없이 간단하게 클라이언트단에서 들어오는 요청의 유효성 검사를 진행할 수 있다.


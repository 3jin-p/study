JAVA Validation
--
Bean Validation 2.0 명세를 따르는 객체의 유효성 검사 

Bean Validation 2.0 : [https://beanvalidation.org/2.0-jsr380/](https://beanvalidation.org/2.0-jsr380/)

여러 레이어에서 필요한 데이터 검증로직을 하나의 도메인 모델로 묶어 처리하여

오류 발생빈도와 성능향상을 도모함. Java에서는 도메인 모델로 Annotation을 사용

Dependencies 

1. Validation API 

```jsx
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.0.Final</version>
</dependency>
```

2. . Validation API Reference Implementation

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

 

- **@NotNull** – validates that the annotated property value is not

    null

- **@AssertTrue** – validates that the annotated property value is *true*
- **@Size** – validates that the annotated property value has a size between the attributes  and ; can be applied to *String*, *Collection*, *Map*, and array properties
- **@Min** –Validates that the annotated property has a value no smaller than the attribute

    v

    value

- **@Max** – validates that the annotated property has a value no larger than the attribute

    value

- ***@Email*** – validates that the annotated property is a valid email address
- ***@NotEmpty*** – validates that the property is not null or empty; can be applied to *String*, *Collection*, *Map* or *Array* values
- ***@NotBlank*** – can be applied only to text values and validated that the property is not null or whitespace
- ***@Positive*** and ***@PositiveOrZero*** – apply to numeric values and validate that they are strictly positive, or positive including 0
- ***@Negative*** and ***@NegativeOrZero*** – apply to numeric values and validate that they are strictly negative, or negative including 0
- ***@Past*** and ***@PastOrPresent*** – validate that a date value is in the past or the past including the present; can be applied to date types including those added in Java 8
- ***@Future** and **@FutureOrPresent*** – validates that a date value is in the future, or in the future including the present

### 2.  이외 HibernateValidator 에서 추가로 구현되어있는 어노테이션

[https://www.baeldung.com/hibernate-validator-constraints](https://www.baeldung.com/hibernate-validator-constraints)

> 각 어노테이션은 필드단위 외에도 Collection의 엘리먼트도 검증,  @ScriptAssert를 이용한 클래스단위 검증이 가능하다

---

# 사용

### 1. Controller 로 들어오는 객체의 검증

```java

@RequestMapping(value = "/test/valid", method = RequestMethod.POST)
public String testValidating(@Valid @RequestBody  ValidationObject validationObject,
												 BindingResults bindingResults) throws Exception {  
		
		if (bindingResult.hasErrors()) { ... }
        ...
}
```

검증결과가  BindingResult 객체에 담기게 되며, 

검증을 통과하지 못했을 시 해당 객체의 Errors 컬렉션에 검증 지정한 검증 메시지들이 담기게 된다.

> 메시지는 다국어 처리가 가능하며,  메세지 키는 Default로 ‘어노테이션명.모델명.필드명’ 이다

### 2. Controller 가 아닌 레이어에서의 검증

```java
@Validated // <<<<< Required
@Service
public class ValidationService{
    public void doValid(@Valid ValidationObject validationObject ) { 
        ... // -> 해당 메서드가 실행될 때 검증로직이 수행된다
    }
}
```

> Controller Layer 가 아닌 곳에서의 검증은 @Validated 어노테이션이 필요한데, 중복된 검증로직의 수행을 지양하도록 생각해볼 시간을 주는게 아닐지..

### 3. Collection Element 의 검증

```java
public class ValidationObject {
    @Min(1) // Min -> Validation For Collection
    private Collection<@NotBlank String> ids; // NotBlank -> Validation For Elements
}
```

### 4. Custom Annotation

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
@Constraint(validatedBy = RequireMaskValidator.class)
@Documented
public @interface RequireMask{
    String message() default "마스크 안끼면 출입불가";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    @Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
    @Retention(RUNTIME)
    @Documented
    @interface List{
        RequireMask[] value();
    }
}

public class RequireMaskValidator
 implements ConstraintValidator<RequireMask, Collection<String> {
// implements ConstraintValidator Interface

    @Override // 검증 로직의 구현 
    public boolean isValid(Collection collection, ConstraintValidatorContext context) {
        if (collection.contains("Mask") {
            return true;
        }
        return false;
    }
}

public class Person{
	@RequireMask 
	Collection<String> belongings;
}
```

### 5. 검증실패 Handling

검증의 실패는 ConstraintViolationException 을 던지기 때문에 

ControllerAdvice 에서 전역으로 ExceptionHandling 이 가능하다. 

---

 

이외에도 Marker Interface를 이용한 Grouping 등의 기능이 가능하다.

## Conclusion

Validation Library가 제공하는 커스텀 어노테이션, 그룹핑 등을 적극 활용하면 

별도의 반복되는 검증로직 작성없이 간단하게 유효성 검사를 진행하고 

손쉽게 프론트단에 메시지를 전달할 수 있다.

다만, 동일한 객체를 여러 레이어에서 다수의 검증을 거치지 않도록 유의하며 사용할 필요가 있을 것 같다. 


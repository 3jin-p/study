Spring Test Strategy
--

스프링이 제공하는 테스트 전략

**Annotations**  
- *@SpringBootTest*   
  통합테스트를 위한 어노테이션 
  모든 Context 를 Load
- *@WebMvcTest*  
  Controller Layer 를 테스트 하기 위한 어노테이션  
  @Controller, @ControllerAdvise, @JsonComponent, Converter, GenericConverter, Filter, HandlerInterceptor, WebMvcConfigurer, HandlerMethodArgumentResolver Bean 들만 Load
- *@DataJpaTest*  
  Persistence Layer 를 테스트 하기 위한 어노테이션  
  JPA와 관련된 Entity 객체들과 Repository Bean 들만 Load
  

  
통합테스트
--
```java
@RunWith(SpringRunner.class)
@ActiveProfiles("dev")
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
public class BaseTest {
    @Autowired
    protected MockMvc mockmvc;
    @Autowired
    protected ObjectMapper objectMapper;
}
```
일반적으로 테스트를 위한 setup을 편하고 통일성 있게 가지기 위하여, 확장하여 사용하는 BaseTest 클래스를 생성한다.  

- *@RunWith* Junit의 테스트 Runner 를 변경하여 테스트 실행 방법을 확장시킨다. Junit5 에서는 @ExtendWith 으로 변경되었다.  
- *@ActiveProfiles* 테스트의 profile 을 설정한다. 
- *@SpringBootTest* 위에 기술된 내용대로, 어플리케이션의 모든 Context를 로드시키는등 다양한 어노테이션을 내재하고 있다.
- *@AutoConfigureMockMvc* MockMvc 객체를 사용할 수 있도록 해준다.
- *@Transactional* 테스트 코드에서의 Transactional은 트랜잭션을 나누는 것 뿐만아니라, 트랜잭션 종료 즉 테스트 케이스 종료 시점에서 트랜잭션을 롤백 시킨다.  
- *Fields* 공통적으로 상속시켜 사용할 빈들을 주입받고 상속받은 테스트 클래스에서 사용한다.

```java
public class IntegrationTest extends BaseTest {
    @Autowired
    private GoodsRepository goodsRepository;
 
    @BeforeEach
    public setUpTests() {
        Goods goods = Goods.builder()
    }
    
    @Test
    @DisplayName(value = "상품 조회 API 테스트") 
    public void testFindGoods() {
        // given
    }
} 
```

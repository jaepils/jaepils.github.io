---
layout: post
title:  "Kotlin Part#8"
date:   2018-07-29 15:40:56
categories: kotlin
---
## Spring framework
지금까지 공부한 kotlin을 Spring framework에서 동작을 하도록 해보겠습니다.
해당 샘플은 백엔드에서 일반적으로 사용되는 Service - Repository 를 kotlin으로 구현하고 만들어진 클래스가 bean으로 등록이 되는 간단한 코드입니다.

### gradle
gradle에 아래 설정을 추가 하면 1.2.41버전을 사용할 수 있게 됩니다.

```java
compile('org.jetbrains.kotlin:kotlin-stdlib-jdk8')
```
나머지는 Spring사이트에서 제공해주는 기본설정을 따르도록 합니다.

### Interface 추가
PersonRepository는 인터페이스로 사람들 이름을 반환하는 getPersonList을 정의하고 있습니다.

```java
interface PersonRepository {

    fun getPersonList(): List<String>
}
```

### PersonRepositoryImpl 추가
PersonRepository을 구현한 클래스로 PersonRepository을 생성을 합니다.


```java
@Slf4j
@Repository
open class PersonRepositoryImpl : PersonRepository {

    override fun getPersonList(): List<String> {
        return listOf("홍길동", "강감찬")
    }

}
```
클래스 정의 앞에 open을 써주지 않으면
> Common causes of this problem include using a final class or a non-visible class

에러가 발생을 합니다.
동작만을 확인할것이기 때문에 테스트로 유명하신분 이름을 리턴하도록 하였습니다.


### PersonService 추가
Service는 PersonRepository을 Spring에게서 DI(Dependency Injection)을 받아야합니다.
kotlin에서 constructor을 사용을 하거나 아래처럼 클래스 정의시 선언을 하면 Spring을 통해서 주입이 됩니다.

```java
@Slf4j
@Service
open class PersonService(private val personRepository: PersonRepository) {

    private val log = LoggerFactory.getLogger(PersonService::class.java)

    fun printPersonList() {
        val personList = personRepository.getPersonList()

        personList.forEach(::println)
    }
}
```
요새 많이 쓰이는 Lombok의 기능으로 Slf4j을 정의를 해도 log를 사용할 수 없습니다. 예전 방식처럼 현재 클래스 정보를 통해서 얻어와야 합니다.
Repository에서 넘겨받은 이름 목록을 루프를 돌면서 출력을 하도록 코드를 작성을 합니다.

### Test 추가
Test 클래스 역시 kotlin을 작성을 할 수 있습니다. 기존 Java와 크게 다른 것은 없습니다.

```java
@SpringBootTest
@RunWith(SpringRunner::class)
@ActiveProfiles(profiles = arrayOf("local"))
class PersonServiceTest {
    @Autowired
    lateinit var personService: PersonService

    @Test
    fun test() {
        personService.printPersonList()

        Assert.assertTrue(true)
    }

}

홍길동
강감찬

```
lateinit이 위의 Service와 다른 부분입니다. @Autowired을 선언하여 주입이 되기를 원하는 경우 반드시 lateinit을 선언을 해줘야 원하는대로 해당 인스턴스 생성 된 이후에 주입이 됩니다.

테스트 결과는 예상되듯이 사람이름을 출력을 하게 됩니다.

## 결론
기존 Spring에서도 kotlin은 특별한 작업을 하지 않아도 유기적으로 동작이 가능한 것을 확인하였습니다.
물론 위의 코드는 너무 간단해서 kotlin의 장점이 들어나지 않았지만, 다음 예제에서 조금 더 복잡한 기능을 통해 천천히 보도록 하겠습니다.

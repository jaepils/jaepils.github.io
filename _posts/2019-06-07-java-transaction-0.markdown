---
layout: post
title:  "JPA + Mybatis"
date:   2019-06-07 15:40:56
categories: spring
---
## JPA
JPA는 테이블 단위로 CUD 작업이 매우 편리 합니다. 다만, JPA는 동작방식이 direct datasource 혹은 mybatis에서 쿼리를 수행하는 방식과 다릅니다.
JPA도 mybatis와 동일하게 실제 쿼리가 실행이 되어야 db에 데이터가 저장이 됩니다. JPA의 경우 flush를 호출하지 않으면, 트랜잭션이 끝나는 시점에 쿼리가 생성이 되기 때문에 mybatis에서 처럼 호출마다 매번 쿼리가 수행이 되는 것과 동작방식이 다릅니다.

간단한 Spring data jpa와 mybatis가 있는 코드로 설명하겠습니다.

```java
@SpringBootTest
@RunWith(SpringRunner.class)
@ActiveProfiles(profiles = "h2")
public class TransactionTest {

    @Autowired
    private JpaRepository jpaRepository;

    @Autowired
    private MybatisRepository mybatisRepository;

    @Test
    @Transactional
    public void test() {
        SampleTable sample = new SampleTable(999999999L);
        jpaRepository.save(sample);

        System.out.println("Mybatis : "+ mybatisRepository.getAllById(999999999L).size());
    }
}

Mybatis : 0
```
JPA에서 save를 했지만, mybatis에서는 조회가 되지 않습니다. 하지만, 아래 코드처럼 수정을 하면 결과는 달라집니다.


```java
@SpringBootTest
@RunWith(SpringRunner.class)
@ActiveProfiles(profiles = "h2")
public class TransactionTest {

    @Autowired
    private JpaRepository jpaRepository;

    @Autowired
    private MybatisRepository mybatisRepository;

    @Test
    @Transactional
    public void test() {
        SampleTable sample = new SampleTable(999999999L);
        jpaRepository.save(sample);

        System.out.println("JPA : "+ jpaRepository.findAllById(999999999L).size());
        System.out.println("Mybatis : "+ mybatisRepository.getAllById(999999999L).size());
    }
}

JPA : 1
Mybatis : 1
```
처음 JPA는 트랜잭션이 끝난 시점에 쿼리가 수행이 되기 때문에 위의 예제에서는 JPA : 1이 맞지만, Mybatis는 1이 나오면 안되고 0이 나와야 합니다. 하지만, Spring data JPA의 경우 find를 호출을 하면 flush가 되어 조회 시점에 쿼리가 수행이 됩니다.


## 결론
JPA와 Mybatis를 섞어서 쓰더라도 트랜잭션은 보장이 됩니다. 다만, JPA는 쿼리가 수행이 되는 시점이 다르기 때문에 mybatis와 로직이 섞여 있는 경우 주의가 요구 됩니다.

## 참조
* https://stackoverflow.com/questions/2673678/what-transaction-manager-should-i-use-for-jbdc-template-when-using-jpa

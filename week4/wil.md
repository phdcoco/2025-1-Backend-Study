## 레포지토리 계층

DB와 직접 소통하며 데이터를 조작한다. JPA의 EntityManager를 이용해 **CRUD**를 수행한다.

여기서 CRUD는

1. Create 생성
2. Read 조회
3. Update 수정
4. Delete 삭제

네 가지의 구성요소를 가지고 있다.

### 엔티티 매니저

엔티티 매니저는 우리 대신 DB와 직접 소통하는 객체이다.

- 새로 생성한 엔티티 객체를 DB에 추가한다.
- DB에서 조회한 데이터로 엔티티 객체를 만든다
- 엔티티 객체에 대한 수정, 삭제를 DB에 반영한다.

`private EntityManager em;` 처럼 em을 이용하는데, find, createQuery 등으로 CRUD를 구현하면 된다.

### 트랜잭션

트랜잭션 : JPA는 DB와 유사하게 트랜잭션 단위로 동작한다. 트랜잭션이 끝나면 모든 변경사항을 DB에 반영한다. @Transactional 이 붙은 메서드에서 DB변경이 가능하다.

ex) 회원가입 - 가입기념 쿠폰 이 두 관계는 항상 동시에 일어나므로 두 사건이 모두 성공이어야 성공이다. 이런 일련의 동작을 하나로 묶어서 관리하는 것을 트랜잭션이라고 한다. 트랜잭션 중간에 에러가 발생하면 범위안의 모든 변경점을 되돌린다.

### 영속성 컨텍스트

영속성 컨텍스트 : JPA가 앤티티를 저장하고 추적하는 메모리 공간으로, DB에서 조회한 엔티티를 캐싱하한다.  → JPA가 DB에 반영할 엔티티의 모든 변경 사항을 보관한다.

이는 수정/삭제 시 em.find()로 Todo 데이터에서 쿼리를 select하여 조회한 객체를 수정만 하면, 커밋 시 자동으로 update SQL이 생성된다. (변경을 감지한다)

그러니까 엔티티 매니저는 변경 사항을 모았다가 한번에 SQL을 생성하고,  영속성 컨텍스트에 올려둔 엔티티 객체를 변경하면 되는거임.

```java
package com.example.todo_api.todo;

import com.example.todo_api.member.Member;
import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository //이 안에 component 어노테이션 포함
public class TodoRepository {

    @PersistenceContext
    private EntityManager em;

    //데이터 생성
    public void save(Todo todo) {
        em.persist(todo); //영속성 컨택스트에 엔티티 추가
    }

    //조회
    //단건 조회 - 1개의 데이터 조회
    public Todo findById(Long todoId) { //프라이머리 키 기준으로 검색
        return em.find(Todo.class, todoId);
    }

    //다건 조회 - 여러 개의 데이터 조회
    public List<Todo> findAll() {
        return em.createQuery("select t from Todo as t", Todo.class).getResultList(); //직접 쿼리 작성
    }

    //조건 조회 - 여러 개의 데이터 조회
    public List<Todo> findAllByMember(Member member) {
        return em.createQuery("select t from Todo as t where t.member = :todo_member", Todo.class)
                .setParameter("todo_member", member)
                .getResultList();
    }

    //수정
    // 엔티티 클래스의 필드를 수정하는 메서드를 작성해서 수정하자

    //삭제
    public void deleteById(long todoId) {
        Todo todo = findById(todoId);
        em.remove(todo);
    }

    //Test 용도로만 사용
    public void flushAndClear() {
        em.flush(); //모든 변경사항을 반영한다
        em.clear();
    }
}

```
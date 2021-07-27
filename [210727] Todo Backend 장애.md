# TodoRepository Error 

##### 210727

DB를 사용하지 않고 MemoryTodoRepository를 통해 add, edit, delete 구현.

JPA를 이용해 구현하기 위해 TodoRepository 생성

```java
package com.greenmuhly.todo.domain;

import lombok.RequiredArgsConstructor;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import java.util.List;

@Repository
@RequiredArgsConstructor
public class TodoRepository {

    private final EntityManager em;

    public void save(Todo todo) {
        if (todo.getId() == null) {
            em.persist(todo);
        } else {
            em.merge(todo);
        }
    }

    public Todo findOne(Long id) {
        return em.find(Todo.class, id);
    }

    public List<Todo> findAll() {
        return em.createQuery("select t from Todo t", Todo.class)
                .getResultList();
    }


}

```

Repository를 혼자 만들어보고 싶어서 각 기능을 구현 후 Test를 하는데 에러가 계속 발생

```java
package com.greenmuhly.todo.domain;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.*;


@DataJpaTest
@Transactional
public class TodoRepositoryTest {

    private final String content = "content";

    @Autowired TodoRepository todoRepository;
    @Autowired TestEntityManager em;

    private Todo getSaved(){
        Todo todo = Todo.builder()
                .content(content)
                .createDate(LocalDateTime.now())
                .build();
        return em.persist(todo);
    }

    @Test
    public void getTest() throws Exception {
        //given
        Todo todo = getSaved();
        System.out.println("==================================");
        System.out.println("todo.getId() = " + todo.getId());
        System.out.println("todo.getContent() = " + todo.getContent());
        System.out.println("todo.getChecked() = " + todo.getChecked());
        System.out.println("todo.getCreateDate() = " + todo.getCreateDate());
        System.out.println("==================================");
        em.persist(todo);
        Long id = todo.getId();

        //when
        Todo savedTodo = todoRepository.getById(id);

        //then
        assertThat(savedTodo.getContent()).isEqualTo(content);
        assertThat(savedTodo.getContent()).isEqualTo(todo.getContent());

    }

    @Test
    public void deleteTest(){
        //given
        Todo todo = getSaved();
        Long id = todo.getId();
        //when
        todoRepository.deleteById(id);

        //then
        assertThat(em.find(Todo.class, id)).isNull();
    }
}

```

뭐가 문제인지 모르겠지만 우선 JPA Repository를 상속받아서 해결.



```java
import org.springframework.data.jpa.repository.JpaRepository;


public interface TodoRepository extends JpaRepository<Todo, Long> {
}
```


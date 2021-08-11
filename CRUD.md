강의자료
https://www.notion.so/Spring-2-58046d5633454406b67905630ab7bc72

0. 데이터베이스 테이블에 생성일자 & 수정 일자 추가 > Class Timestamped를 만들고 기본 코드 놓고

//기본 코드     
@MappedSuperclass // 상속했을 때, 컬럼으로 인식하게 합니다.
@EntityListeners(AuditingEntityListener.class) // 생성/수정 시간을 자동으로 반영하도록 설정
public abstract class Timestamped {

    @CreatedDate // 생성일자임을 나타냅니다.
    private LocalDateTime createdAt;

    @LastModifiedDate // 마지막 수정일자임을 나타냅니다.
    private LocalDateTime modifiedAt;
}

Timestamped를 넣고싶은 테이블에 상속(extends)으로 추가, 그리고 어플리케이션 파일에 @EnableJpaAuditing 추가 

1. 데이터 베이스 테이블 구축 > 예시: Class Students를 만들고 컬럼을 지정하기    >>>> mysql로 치면 CREATE TABLE

@Getter // lombok으로 getter 대처 >> lombok이란 
@NoArgsConstructor // 기본생성자를 대신 생성해줍니다.
@Entity // 테이블임을 나타냅니다.   ************* 
public class Person extends Timestamped {

    @Id// ID 값, Primary Key로 사용하겠다는 뜻입니다.
    @GeneratedValue(strategy = GenerationType.AUTO)// 자동 증가 명령입니다.
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String age;

    @Column(nullable = false)
    private String job;

    @Column(nullable = false)
    private String seed;

    public Person(PersonRequestDto requestDto){    //<< PersonRequestDto에서 가져옴 
        this.name = requestDto.getName();
        this.age = requestDto.getAge();
        this.job = requestDto.getJob();
        this.seed = requestDto.getSeed();
    }
}

2. 데이터베이스 쿼리 변역기 작성 >> 예시: Interface  PersonRepository 만들고 public interface PersonRepository extends JpaRepository<Person, Long>
import org.springframework.data.jpa.repository.JpaRepository;

3. JPA 적용 spring.jpa.show-sql=true >>> vscode에서 적용 방법 알기.


4. Class PersonRequestDto를 만들어준다  <<<<  Person 데이터 가져오는 Class

@Getter
@RequiredArgsConstructor
public class PersonRequestDto {
    private final String name;
    private final String age;
    private final String job;
    private final String seed;
}

5. controller폴더에서 Class PersonController 만들고, CRUD코드를 만든다

@RequiredArgsConstructor // 권한을 줌
@RestController
public class PersonController {

    private final PersonRepository personRepository;

    private final PersonService personService;

    // oooMapping을 통해서, 같은 주소라도 방식이 다름을 구분합니다.
    // C(Create)
    @PostMapping("/api/persons")
    public Person createPerson(@RequestBody PersonRequestDto requestDto) {
        Person person = new Person(requestDto);
        return personRepository.save(person);
    }

    // R(Read)
    @GetMapping("/api/persons")
    public List<Person> getPerson() {
        return personRepository.findAll();
    }

    // U(Update)
    @PutMapping("/api/persons/{id}")
    public Long updatePerson(@PathVariable Long id, @RequestBody PersonRequestDto requestDto) {
        return personService.update(id, requestDto);
    }
    
    // D(Delete)
    @DeleteMapping("/api/persons/{id}")
    public Long deletePerson(@PathVariable Long id) {
        personRepository.deleteById(id);
        return id;
    }
}

6. service폴더에서 PersonService 추가하고 코드 입력

@RequiredArgsConstructor //권한을 줌
@Service // Service인걸 알려준다
public class PersonService {
    private final PersonRepository personRepository;

    @Transactional // DB상에 반영 해달라
    public Long update(Long id, PersonRequestDto requestDto) {
        Person person = personRepository.findById(id).orElseThrow(
                () -> new IllegalArgumentException("해당 id가 존재하지 않습니다.")
        );
        person.update(requestDto);
        return person.getId();
    }
}

// Update하기 위해 업데이트 메소드를 Class Person에 코드 추가 한다. 

    public void update(PersonRequestDto requestDto) {
        this.name = requestDto.getName();
        this.age = requestDto.getAge();
        this.job = requestDto.getJob();
        this.seed = requestDto.getSeed();
    }

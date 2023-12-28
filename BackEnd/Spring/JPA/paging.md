Spring Data Jpa 페이징
===
## 페이징

**리포지토리 메소드 정의**
```text
Page<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
Slice<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안함
List<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안함
List<Member> findByUsername(String name, Sort sort);
```
**서비스에서 리포지토리 메소드 사용**
```text
PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC,"username"));
Page<Member> page = memberRepository.findByUsername("who", pageRequest);
List<Member> content = page.getContent();
```

count쿼리는 무겁게 동작한다는 점 유념  
**자세한 메소드는 Page와 Slice 인터페이스를 확인**  


## Spring MVC에서의 간편한 페이징 사용
```text
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
    Page<Member> page = memberRepository.findAll(pageable);
    return page;
}
```
Pageable은 인터페이스이고, pageable이 받는 객체는 실제로는 org.springframework.data.domain.PageRequest이다.  

**요청 파라미터**  
예시) /members?page=0&size=3&sort=id,desc&sort=username,desc  
page: 현재 페이지, 0부터 시작한다.  
size: 한 페이지에 노출할 데이터 건수  
sort: 정렬 조건을 정의한다. 정렬 방향을 변경하고 싶으면 sort 파라미터를 추가한다. asc는 기본값이다.  

```text
@RequestMapping(value = "/members_page", method = RequestMethod.GET)
public String list(@PageableDefault(size = 12, sort = “username”, direction = Sort.Direction.DESC) Pageable pageable) {
 ...
}
```
개별 설정으로 위와 같이 페이지의 default값을 줄 수 있다.  
이를 사용하면 파라미터가 줄어드는 이점이 생긴다.  

**Page는 map()을 지원해서 내부 데이터를 다른 것으로 변경할 수 있다.**   
```text
@Data
public class MemberDto {
    private Long id;
    private String username;
    public MemberDto(Member m) {
        this.id = m.getId();
        this.username = m.getUsername();
    }
}
```
```text
// Java
@GetMapping("/members")
public Page<MemberDto> list(Pageable pageable) {
 return memberRepository.findAll(pageable).map(MemberDto::new);
}
```
```text
// Kotlin
@GetMapping("/members")
fun list(pageable: Pageable): Page<MemberDto>{
 return memberRepository.findAll(pageable).map(MemberDto(it));
}
```

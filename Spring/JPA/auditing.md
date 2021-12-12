Spring Data Jpa Auditing
===
엔티티 생성, 변경 시 시간이나 수정자같은 필드는 모든 엔티티에서 공통으로 필요한 경우가 많다.  
따라서 공통으로 등록할 다음 4가지를 고려해야 한다.  
- 등록일
- 수정일
- 등록자
- 수정자

```text
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseTimeEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}
```
```text
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseEntity extends BaseTimeEntity {
    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```
위와 같이 구성할 경우 등록일, 수정일이 필요할 때는 BaseTimeEntity를,  
등록자, 수정자까지 필요할 경우 BaseEntity를 상속해 등록일, 수정일, 등록자, 수정자 모두를 가져올 수 있다.  

```text
@Component
public class LoginUserAuditorAware implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (null == authentication || !authentication.isAuthenticated()) {
            return null;
        }
        User user = (User) authentication.getPrincipal();
        return Optional.of(user.getUserId());
    }
}
```
그리고 수정자, 등록자는 값이 그냥 들어갈 수는 없으므로 위와 같이 빈을 등록해서 사용하면 된다.  
getCurrentAuditor의 로직은 현재 유저를 가져오는 로직이다.   
SecurityContextHolder, HttpSession, AuthenticationToken 등에서 값을 꺼내와야 한다.  

참고로 글로벌하게 운영될 필요가 있는 서비스는  
hibernate의 ZonedDateTime을 사용한다.  

```text
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class AbstractEntity {

    @CreationTimestamp
    private ZonedDateTime createdAt;

    @UpdateTimestamp
    private ZonedDateTime updatedAt;
}
```
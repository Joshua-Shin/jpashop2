# jpashop2

## 단원별 요점 정리

### API 개발 기본
#### 회원 등록 API
- 여태껏 나는 템플릿엔진으로 만든 화면을 Controller에서 뿌려줬는데, 당연하게도 실무 개발에서는 SPA로 만든 웹이나, 앱에서는 서버쪽에 API를 요청, 서버에서 API 응답으로 진행이 됨. 그래야 분업이 잘 되니까.
- 클라에서 API 요청 오는건 form이나 쿼리 형태로 오는게 아니라 json 형태로 http body에 담아서 요청이 들어옴. form이나 쿼리 형태면 @ModelAttribute로 여태껏 잘 대응했음.
- 화면 뿌리는 Controller랑 API 용 Controller는 아예 패키지를 다르게 잡는게 나음. 패키지명 "api"
- @RequestBody : 요청으로 들어오는 JSON을 해당 에노테이션을 붙여준 객체 파라미터에 변환해서 바인딩 시켜줌.
- @ResponseBody : 클래스나 메소드레벨에 선언. 응답으로 나가는 객체를 JSON 형식으로 잘 만들어서 응답 보내줌.
- @RestController : @Responsbody + @Controller. 어차피 api만을 대응하기 위한 클래스들이니까 클래스레벨에 해당 에노테이션을 선언.
- 요청 받는 객체와 응답으로 나가는 객체를 엔티티 객체로 사용하지 마라! 추천이 아니라 강제야! (요청편)
  - 요청으로 받는 객체는 DTO(Data Transfer Object)를 따로 정의해서 그걸로 받아주는게 좋아. 왜?
    - 만약 엔티티 그대로 요청을 받는다. 치면 요청때마다 필드 검증하는 사항들이 다를 수 있는데 이를 @Valid로 대응하기가 어려움. 어떤 메소드는 name이 필수고 어떤 메소드는 age가 필수 인데 어디다가 @NotEmpty를 붙일거야
    - 엔티티의 필드명을 수정한다 치면, 클라쪽에서 들어오는 json의 필드명도 수정해야됨...
    - 클라쪽에서 어떤 필드들을 보내온것인지 파악하기 힘들어.
    - 따라서, 딱 필요한 필드값들만 fit하게 받아오는 그에 맞는 DTO 객체를 따로 정의해서 그걸로 받아주면, 엔티티 처럼 필드명 수정할 일도 없고, Valid도 요청에 맞춰 딱딱 걸어주고, 요청으로 들어온 필드가 무엇인지도 딱 보기좋고.
    - API가 아니더라도 그 화면 뿌리는상황에서도 form 객체로 요청 받았잖아. 똑같애 이것도.

#### 회원 수정 API
- 수정도 등록과 다른 DTO 만들어서 그걸로 요청 파라미터 받아줌.
- DTO를 외부 클래스로 정의하는게 아니라, 해당 api컨트롤러 클래스 내에 static class로 정의해주네.
- 엔티티 객체에는 롬복에서 @Getter만 되도록 쓰는데, DTO는 일단 @DATA 박고, @AllArgu도 박고 좀 편하게 롬복사용.

#### 회원 조회 API
- 요청 받는 객체와 응답으로 나가는 객체를 엔티티 객체로 사용하지 마라! 추천이 아니라 강제야! (응답편)
  - 엔티티의 필드명이 바뀌면, api 스펙 자체가 달라지는거나 다름없어.
  - 모든 값이 노출되기때문에 보안상으로도 안좋고.
  - 엔티티의 필드에다가 @JsonIgnore 같은거 선언하면 해당 필드는 응답 json으로 노출이 안되게 되긴하지만, 이렇게 프레젠테이션 로직이 담기는게 좋지 못함.
  - api 용도가 다 다른데 하나의 엔티티에서 어떤 api 에는 필드A가 노출 안되게하고 어떤 api에는 필드A가 필요하고, 이런걸 다 대응할 수가 없어.
- 전체 회원 조회 같이 List 배열이 나가야 할 경우, 이때도 List<MemberDto>로 반환하는게 아니라 List<MemberDto> 타입의 필드를 가지고 있는 객체로 반환해줘야돼.
  - 그래야 json이 배열 형태 [ ] 가 아니라 Object 형태 { } 로 나갈 수 있음
    - 그래야 해당 json에 count 필드라든가 새로운 필드값을 추가해서 보낼 수 있지. 그냥 배열 형태로 보내버리면 유연성이 확 떨어지게됨.
  - 예시
```
@GetMapping("/api/v2/members")
public Result membersV2() {
    List<Member> findMembers = memberService.findMembers(); 
    //엔티티 -> DTO 변환
    List<MemberDto> collect = findMembers.stream()
                .map(m -> new MemberDto(m.getName()))
                .collect(Collectors.toList());
    return new Result(collect);
}
@Data
@AllArgsConstructor
static class Result<T> {
    private T data;
}
@Data
@AllArgsConstructor
static class MemberDto {
    private String name;
}
```

### API 개발 고급 - 준비
#### API 개발 고급 소개
- 결국 성능에서 문제 되는것은 조회 api
#### 조회용 샘플 데이터 입력
- 회원 userA가 주문 한건 했고 해당 주문에는 주문아이템 jpa book1 과 jpa book2가 들어가있음
- 회원 userB도 주문 한건 했고 해당 주문에는 주문아이템 spring book1 과 spring book2가 들어가있음
- 즉, 회원 2명이 각각 주문 한건씩 했고 각 주문에는 주문아이템 2건씩 들어가있는 상황. 이제 고고.

### API 개발 고급 - 지연 로딩과 조회 성능 최적화
#### 간단한 주문 조회 V1: 엔티티를 직접 노출
- API 응답할때 엔티티를 직접 노출해버리면, 양방향 연관관계 걸린 엔티티끼리 서로를 호출하면서 무한 루프 걸림.
- 이거 나 해봤잖아. /members ㅋㅋㅋ to_string을 내가 직접 정의해줘야하나 어쩌나 했는데 결국 답은
- DTO로 변환해서 딱 필요한 값만 담아서 반환 시킨다.
  
#### 간단한 주문 조회 V2: 엔티티를 DTO로 변환
- List<Order> orders = orderRepositoy.findAll(); 해버린뒤 dto로 변환 하기 위해서 orders for문 돌면서 order에 있는 회원엔티티와 배송주소엔티티를 get 함.
- 이때 주문에서 회원. 주문에서 배송주소. 가 지연로딩으로 되어있기에 프록시객체 넣어놨다가 get을 하는 순간 db에 조회 쿼리를 날리게 되는거.
- 이러한 상황에서 일단 findAll() 한번 하는데 쿼리 1개, 여기서 주문 건수가 2건인데, 각 주문마다 회원이름과 배송주소 조회하는데 쿼리 2건. 즉 총 1 + 2 * 2개의 쿼리가 나감.
- 이를 n + 1 문제라 함. n은 여기서는 order 건수가 되는거지. 회원과 배송이 있기 때문에 위의 예시에서는 2*n이 되어버림.
- 만약 주문 건수가 100개 였다. findAll() 하는 api 한번 하는데, 쿼리가 201번 나가는거야...

#### 간단한 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화
- 쿼리 한번에 여러 데이터를 쭉 가져오는건 괜찮아. 이건 문제가 아니야
- 문제는 자바 코드 한줄에 쿼리가 수십개 수백개가 나가는게 문제. 쿼리라는건 db에 데이터를 요청하는건데, 필요한 데이터를 한번에 달라하는게 낫지 계속 여러번 요청하는건 성능상으로 당연히 안좋지
- 그렇다고 지연로딩을 EAGER로 바꾼다? 이러면 쿼리들이 양방향으로 연결되어있고 즉시로딩인 애들을 아주 끝까지 다 가져오기 때문에 문제.
- 결국 해답은 페치조인.
- 실무의 성능문제의 90%는 페치조인으로 해결하게 된다. 페치조인은 100%로 이해해야 한다.
- 페치조인을 통해 주문과 관련된 회원과 배송을 다 가져오게 됨으로써 쿼리 한방에 해결이 된다. 
- orderRespository에 페치조인을 활용한 전체 주문 조회 메소드
```
public List<Order> findAllWithMemberDelivery() {
    return em.createQuery(
        "select o from Order o" +
            " join fetch o.member m" +
            " join fetch o.delivery d", Order.class)
        .getResultList();
}
```
#### 간단한 주문 조회 V4: JPA에서 DTO로 바로 조회

### API 개발 고급 - 컬렉션 조회 최적화
#### 주문 조회 V1: 엔티티 직접 노출
#### 주문 조회 V2: 엔티티를 DTO로 변환
#### 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화
#### 주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파
#### 주문 조회 V4: JPA에서 DTO 직접 조회
#### 주문 조회 V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화
#### 주문 조회 V6: JPA에서 DTO 직접 조회, 플랫 데이터 최적화
#### API 개발 고급 정리

### API 개발 고급 - 실무 필수 최적화
#### OSIV와 성능 최적화



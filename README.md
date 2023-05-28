# jpashop2

## 단원별 요점 정리

### API 개발 기본
#### 회원 등록 API
- 여태껏 나는 템플릿엔진으로 만든 화면을 Controller에서 뿌려줬는데, 당연하게도 실무 개발에서는 SPA로 만든 웹이나, 앱에서는 서버쪽에 API를 요청, 서버에서 API 응답으로 진행이 됨. 그래야 분업이 잘 되니까.
- 클라에서 API 요청 오는건 form이나 쿼리 형태로 오는게 아니라 json 형태로 http body에 담아서 요청이 들어옴. form이나 쿼리 형태면 @ModelAndView로 여태껏 잘 대응했음.
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
#### 조회용 샘플 데이터 입력

### API 개발 고급 - 지연 로딩과 조회 성능 최적화
#### 간단한 주문 조회 V1: 엔티티를 직접 노출
#### 간단한 주문 조회 V2: 엔티티를 DTO로 변환
#### 간단한 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화
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



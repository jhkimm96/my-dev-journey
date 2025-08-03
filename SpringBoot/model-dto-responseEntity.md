## 📁  Spring MVC 컨트롤러 응답 방식 비교

Spring MVC에서는 컨트롤러가 클라이언트 요청에 응답할 때 여러 방식이 사용됩니다.  
이 문서에서는 대표적인 세 가지 응답 방식의 차이점과 예시, 그리고 RESTful API에 적합한 이유까지 정리합니다.

---

## ✅ 1. 세 가지 주요 방식 개요

| 방식 | 설명 |
|------|------|
| **Model 방식** | 서버에서 HTML 템플릿을 렌더링하여 클라이언트에 전달하는 전통적인 방식 |
| **DTO 직접 반환** | DTO 또는 List<DTO>를 그대로 반환하여 JSON으로 응답하는 간단한 REST API |
| **ResponseEntity 방식** | HTTP 상태코드, 헤더, 응답 바디를 모두 제어할 수 있는 정식 RESTful API 응답 방식 |

---

## ✅ 2. 비교 표

| 항목 | Model 방식 | DTO 직접 반환 | ResponseEntity 방식 |
|------|------------|----------------|----------------------|
| **용도** | SSR(서버 렌더링) | 간단한 JSON 응답 | RESTful JSON API |
| **Controller 어노테이션** | `@Controller` | `@RestController` | `@RestController` |
| **응답 타입** | `String` (뷰 이름) | DTO/List (자동 JSON) | `ResponseEntity<T>` |
| **요청 파라미터 처리** | `@ModelAttribute`, `@RequestParam` 등 | `@ModelAttribute` | `@ModelAttribute`, `@RequestBody`, `@PathVariable` 등 |
| **응답 데이터 처리** | `model.addAttribute()` → View | 객체 직접 반환 → JSON | 상태코드 + 헤더 + 바디 모두 제어 |
| **상태코드 설정** | 불가 (항상 200) | 불가 (항상 200) | 가능 (`200`, `400`, `204` 등) |
| **헤더 제어** | 불가 | 불가 | 가능 |
| **프론트 처리 방식** | HTML 렌더링 | JS에서 JSON 처리 | SPA(Vue/React) 등에서 JSON 처리 |

---

## ✅ 3. 예제 코드

### 📘 3.1 Model 방식 (SSR)

```java
@Controller
public class MemberPageController {

    @GetMapping("/members")
    public String getMemberPage(@ModelAttribute MemberSearchDto searchDto, Model model) {
        List<MemberDto> members = memberService.searchMembers(searchDto);
        model.addAttribute("members", members);
        return "member/list"; // templates/member/list.html
    }
}
```

### 📗 3.2 DTO 직접 반환 (간단한 REST API)

```java
@RestController
public class MemberApiController {

    @GetMapping("/api/members")
    public List<MemberDto> getMembers(@ModelAttribute MemberSearchDto searchDto) {
        return memberService.searchMembers(searchDto); // JSON으로 자동 변환
    }
}
```

### 📙 3.3 ResponseEntity 방식 (RESTful API)

```java
@RestController
public class MemberApiController {

    @GetMapping("/api/members")
    public ResponseEntity<List<MemberDto>> getMembers(@ModelAttribute MemberSearchDto searchDto) {
        List<MemberDto> members = memberService.searchMembers(searchDto);

        if (members.isEmpty()) {
            return ResponseEntity.noContent().build(); // 204 No Content
        }

        return ResponseEntity.ok(members); // 200 OK
    }
}
```

---

## ✅ 4. ResponseEntity가 RESTful API에 적합한 이유

### 1. **HTTP 상태코드 명확히 표현**
- 리소스 생성: `201 Created`
- 요청 성공: `200 OK`
- 내용 없음: `204 No Content`
- 잘못된 요청: `400 Bad Request`
- 권한 없음: `401 Unauthorized`
- 서버 오류: `500 Internal Server Error`

### 2. **헤더 설정 가능**
- `Location`, `Authorization`, `Cache-Control`, `ETag` 등 다양한 헤더 설정 가능

### 3. **응답 구조를 일관성 있게 설계 가능**
```json
{
  "code": 200,
  "message": "Success",
  "data": [{ "id": 1, "name": "홍길동" }]
}
```

---

## ✅ 5. 결론 및 추천

| 상황 | 추천 방식 |
|------|-----------|
| SSR 템플릿 (Thymeleaf 등) | `Model + ViewName` 방식 |
| 간단한 JSON 응답만 필요할 때 | DTO 직접 반환 |
| 실무 RESTful API 구축 시 | `ResponseEntity` 방식 (권장) |

---

## ✅ 추가 팁

모든 API 응답을 `ResponseEntity<BaseResponse<T>>` 패턴으로 통일하면,  
예외 처리(`@RestControllerAdvice`)와 문서화(Swagger)에서도 관리가 쉬워집니다.
## 📁 Entity → DTO 변환

## ✅ Entity와 DTO는 왜 나눠서 써야 할까?

### 📌 Entity
- 실제 **DB 테이블과 1:1 매핑**되는 클래스
- 저장/조회/수정 등 DB 연산의 대상
- 민감한 정보도 포함됨 (`password`, `role`, `emailAuthYn` 등)
- 예: `Member`, `Post`, `Order`

```java
import com.example.member.entity.Member;
@Entity
public class Member {
    private String userId;
    private String password;
    private String email;
    private String role;
    private boolean emailAuthYn;
}
```

---

### 📌 DTO (Data Transfer Object)
- **데이터 전달 목적**으로 사용하는 클래스 (예: 화면에 보여줄 값)
- Entity의 전체 데이터가 아니라 **일부만 담거나 가공해서** 전송
- 민감한 정보는 제외 가능

```java
public class MemberDto {
    private String userId;
    private String email;
}
```

---

## ✅ DTO에 데이터를 어떻게 옮길까?

### 1️⃣ 직접 `set()`으로 옮기는 방식

```java
MemberDto dto = new MemberDto();
dto.setUserId(member.getUserId());
dto.setEmail(member.getEmail());
```

📌 문제점
- 코드가 길어짐
- 누락/실수 발생 가능성
- 매번 반복해야 함

---

### 2️⃣ `of()` 정적 메서드 + `Builder` 패턴 사용

```java
public static MemberDto of(Member member) {
    return MemberDto.builder()
        .userId(member.getUserId())
        .email(member.getEmail())
        .build();
}
```

사용:

```java
MemberDto dto = MemberDto.of(member);
```

📌 장점
- 코드 간결
- 일관된 방식으로 Entity → DTO 변환 가능
- 재사용성 좋음

---

## 🧠 자주 묻는 질문 (Q&A)

### ❓ 왜 set() 말고 of()를 쓰나요?
- `of()`는 DTO 생성 로직을 **한 곳에 몰아두기** 때문에 유지보수가 쉬워요.
- 코드도 훨씬 깔끔하고 **한 줄로 DTO 생성**이 가능합니다.

### ❓ 어차피 데이터 옮기긴 마찬가지 아닌가요?
- 맞지만, `set()`은 매번 복붙 & 실수 위험
- `of()`는 재사용하고 단일 책임 원칙(SRP)을 지키는 더 좋은 설계 방식입니다.

---

## 📌 요약 비교

| 항목        | `set()` 방식                  | `of()` + builder 패턴         |
|-------------|-------------------------------|-------------------------------|
| 코드 길이    | 길고 반복적                    | 한 줄로 가능                   |
| 유지보수     | 불편 (변경 시 여러 곳 수정 필요) | 편함 (한 곳만 수정하면 됨)     |
| 실수 위험    | 높음                          | 낮음                           |
| 재사용성     | 낮음                          | 높음                           |

---

## ✅ 결론

- DTO는 Entity에서 필요한 값만 골라서 가공하기 위한 용도
- `set()` 방식보다 `of()` + `builder()` 조합을 추천
- 특히 실무에서는 유지보수성과 코드 통일성이 매우 중요!
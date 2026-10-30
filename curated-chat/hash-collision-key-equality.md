# 해시 충돌: 해시값이 같아도 키는 다를 수 있다

해시 테이블은 키의 해시값으로 탐색할 위치를 좁힌다. 해시값은 키 그 자체가 아니므로, 같은 해시값이 나온 서로 다른 키를 구분하는 동등성 비교가 필요하다. 좋은 분포와 적절한 용량 관리 아래 평균적으로 빠른 검색을 기대하지만, 충돌이 몰린 최악의 경우까지 상수 시간을 보장하지는 않는다.

## 독립 작성 충돌 실험

다음 예제는 모든 키가 같은 해시값을 내도록 일부러 만든다. 실제 서비스용 키 설계가 아니라, Python 딕셔너리가 충돌한 키를 구분하는지 보는 작은 학습 실험이다. 키는 불변 객체이며 동등한 키는 반드시 같은 해시값을 갖는다.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class DemoKey:
    label: str

    def __hash__(self):
        return 7  # 의도적인 충돌. 실제 설계에 사용하지 않는다.

a, b = DemoKey("alpha"), DemoKey("beta")
assert hash(a) == hash(b)                         # 1. 충돌
assert a != b                                   # 2. 다른 키
lookup = {a: "first", b: "second"}
assert len(lookup) == 2                          # 3. 둘 다 보존
assert lookup[a] == "first"                      # 4. 첫 키 조회
assert lookup[b] == "second"                     # 5. 두 번째 키 조회
lookup[DemoKey("alpha")] = "updated"
assert len(lookup) == 2 and lookup[a] == "updated"  # 6. 같은 키 갱신
assert DemoKey("missing") not in lookup          # 7. 없는 키
del lookup[a]
assert len(lookup) == 1 and lookup[b] == "second"  # 8. 다른 키 보존
print("PASS: 8 collision and equality checks")
```

2026-10-30 UTC 정기 실행에서 이 문서의 정확한 코드 블록을 Python으로 추출해 실행했다. 8개 검사가 통과했다. 성능 측정, 대규모 입력, 동시성 또는 기존 C 저장소 전체의 검증은 아니다.

## 적용할 때 확인할 점

- `a == b`이면 `hash(a) == hash(b)`여야 한다. 역방향은 성립하지 않는다.
- 딕셔너리에 넣은 키의 해시·동등성 의미가 바뀌면 조회 계약이 깨질 수 있으므로 불변 키를 사용한다.
- 해시값을 영구 식별자로 저장하거나 실행 간 같은 값이라고 가정하지 않는다. 특히 Python 문자열 해시는 실행마다 달라질 수 있다.
- 일반 해시 테이블용 해시와 보안용 암호학적 해시는 목적이 다르다. 해시 처리만으로 개인정보가 익명화되었다고 판단하지 않는다.
- 서버 분배, 영구 식별자, 보안용 토큰은 별도의 요구사항을 가진 설계다. 이 예제는 그 구현이 아니다.

## 출처와 범위

[개인 학습 질문과 설명](https://chatgpt.com/c/6ab91656-57a0-83e8-9796-bc95f3865d2c?messageId=23b3330d-4541-5eef-bbd9-39ffa8344e17)의 실제 본문을 읽고 검색·충돌·동등성 개념을 정리했다. 설명과 실험은 새로 구성했고 원문의 식별자 예시, 대화 전문, 후속 과제 안내, 강의자료와 개인 기록은 복제하지 않았다. 출처 링크는 저장소 독자에게 열리지 않을 수 있다.

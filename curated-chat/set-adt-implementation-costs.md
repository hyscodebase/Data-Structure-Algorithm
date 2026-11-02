# 집합의 의미와 구현 비용 분리하기

집합(set)은 중복 원소를 하나로 취급하고 포함 여부를 묻는 추상 자료형이다. 이 의미만으로 내부가 트리인지 해시 테이블인지, 순회가 정렬되는지, 연산 비용이 얼마인지 결정되지는 않는다.

## 세 층으로 질문하기

1. **의미:** 원소의 동일성을 어떤 기준으로 판단하는가? 같은 원소를 다시 넣으면 개수가 유지되는가?
2. **표현:** 배열, 연결 구조, 균형 트리, 해시 테이블 중 무엇으로 저장하는가?
3. **보장:** 평균 비용과 최악 비용, 정렬 순서, 범위 질의를 각각 보장하는가?

해시 기반 구현은 적절한 분산과 부하 관리 아래에서 일반적으로 빠른 평균 탐색을 제공하지만, 모든 입력에서 상수 시간이라는 뜻은 아니다. 균형 탐색 트리는 비교 기반으로 높이를 제한해 단일 탐색·삽입·삭제를 로그 시간에 처리할 수 있다. **균형이 보장되지 않는 일반 BST에는 이 보장을 적용하면 안 된다.** 해시 계산·키 비교 자체의 비용도 따로 고려한다.

집합은 원소의 존재만 표현한다. 맵(map)은 키에 대응하는 값을 저장한다. 맵의 키 집합을 집합으로 사용할 수는 있지만, 이것이 두 추상 자료형의 의미가 같다는 뜻은 아니다. 정렬된 집합을 원한다면 정렬 순서를 제공하는 구현을 선택하거나 별도의 정렬 비용을 지불해야 한다.

## 같은 의미, 다른 표현의 작은 실험

아래 두 구현은 정수 원소만 사용하는 교육용 예다. 하나는 입력 순서의 리스트에서 선형 검색하고, 다른 하나는 정렬 리스트에서 이진 검색한다. 둘 다 균형 트리나 실제 해시 테이블 구현은 아니다. 정렬 리스트의 이진 검색은 로그 횟수의 비교를 하지만, 중간 삽입·삭제는 원소 이동 때문에 선형 시간이 든다. 이 구분이 이 실험의 핵심이다.

```python
from bisect import bisect_left

class LinearSet:
    def __init__(self):
        self.values = []

    def add(self, x):
        if x not in self.values:
            self.values.append(x)

    def discard(self, x):
        if x in self.values:
            self.values.remove(x)

    def contains(self, x):
        return x in self.values

class SortedListSet:
    def __init__(self):
        self.values = []

    def add(self, x):
        i = bisect_left(self.values, x)
        if i == len(self.values) or self.values[i] != x:
            self.values.insert(i, x)

    def discard(self, x):
        i = bisect_left(self.values, x)
        if i < len(self.values) and self.values[i] == x:
            self.values.pop(i)

    def contains(self, x):
        i = bisect_left(self.values, x)
        return i < len(self.values) and self.values[i] == x

cases = [
    [],
    [("add", 7)],
    [("add", 7), ("add", 7)],
    [("add", 9), ("add", -2), ("add", 4)],
    [("discard", 3)],
    [("add", 3), ("discard", 3)],
    [("add", 3), ("discard", 3), ("add", 3)],
    [("add", 8), ("add", 1), ("add", 5),
     ("discard", 8), ("discard", 0), ("add", 1)],
]
for operations in cases:
    a, b, reference = LinearSet(), SortedListSet(), set()
    for operation, value in operations:
        getattr(a, operation)(value)
        getattr(b, operation)(value)
        getattr(reference, operation)(value)
        assert set(a.values) == set(b.values) == reference
        assert len(a.values) == len(b.values) == len(reference)
        assert b.values == sorted(reference)
    for value in range(-3, 11):
        assert a.contains(value) == b.contains(value) == (value in reference)
print("PASS: 8 semantic-equivalence scenarios")
```

실험은 의미의 동등성·중복 제거·없는 원소 삭제·재삽입을 확인한다. 시간 측정이나 점근 복잡도 증명이 아니며, 언어별 표준 컨테이너의 내부 구현을 검증하지 않는다. 실제 라이브러리를 선택할 때는 해당 버전의 문서상 계약을 별도로 확인한다.

## 출처와 재작성 범위

- 개인 학습 Q&A: [Set과 해시 테이블의 차이 질문](https://chatgpt.com/c/678ca26b-8a70-800e-b982-a4460bc65170?messageId=da79a0a6-60fb-4113-b576-70c812e3b86d), 2025-01-20.
- 사용자 질문과 답변 본문 전체를 확인하고 일반 개념만 독립적인 설명·새 정수 예제·검사로 재작성했다. 대화 전문이나 제3자 문제·강의자료는 포함하지 않았다.
- 기존 [해시 충돌과 키 동등성](hash-collision-key-equality.md)은 키 동일성에 집중하고, 이 노트는 추상 자료형과 구현 비용의 구분을 다룬다.
- 확인 환경과 테스트 결과는 manifest에 기록한다. 기존 C 저장소 전체 빌드·성능 검증을 뜻하지 않는다.

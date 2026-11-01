# 버블 정렬: 한 바퀴 뒤 확정되는 구간

인접한 두 항목의 순서가 잘못되었을 때만 교환하면, 한 번의 왼쪽→오른쪽 순회가 끝날 때 남은 구간의 최댓값이 오른쪽 끝에 도착한다. 다음 순회는 이미 확정된 끝부분을 제외한다. 이 설명은 일반적인 알고리즘 개념을 독립적으로 정리한 학습 노트다.

## 계약과 불변식

- 비교 가능한 정수 또는 같은 형식의 키를 사용한다. NaN이나 비교 불가능한 혼합형은 이 예제의 대상이 아니다.
- 호출자의 입력은 복사하여 보존한다. 결과와 각 순회 직후의 스냅샷을 반환한다.
- 각 순회 뒤 오른쪽 확정 구간은 전체 정렬 결과의 같은 접미 구간과 일치한다.
- 같은 키끼리는 교환하지 않으므로 원래 상대 순서를 보존하는 안정 정렬이다.
- 한 바퀴 동안 교환이 없으면 현재 구간도 정렬된 상태이므로 조기에 끝낸다.

## 독립 작성 실험

아래 구현과 입력은 원본 이미지·강의 코드의 복사본이 아니라 불변식 검사용으로 새로 작성했다.

```python
def bubble_trace(items, key=lambda item: item):
    work = list(items)
    snapshots = []
    end = len(work)
    while end > 1:
        changed = False
        for right in range(1, end):
            left = right - 1
            if key(work[right]) < key(work[left]):
                work[left], work[right] = work[right], work[left]
                changed = True
        snapshots.append(work.copy())
        end -= 1
        if not changed:
            break
    return work, snapshots


cases = [[], [12], [1, 3, 7], [7, 3, 1], [2, 2, 2],
         [-4, 8, 0, -4], [9, 1, 6, 3]]
for values in cases:
    original = values.copy()
    result, snapshots = bubble_trace(values)
    expected = sorted(values)
    assert result == expected and values == original
    for passes, snapshot in enumerate(snapshots, start=1):
        assert snapshot[-passes:] == expected[-passes:]

# 8. 동일 키 항목은 입력 순서를 유지한다.
records = [(3, "first"), (1, "x"), (3, "second"), (1, "y")]
result, _ = bubble_trace(records, key=lambda pair: pair[0])
assert result == [(1, "x"), (1, "y"), (3, "first"), (3, "second")]

# 9. 이미 정렬된 길이 3 입력은 한 순회만 필요하다.
assert len(bubble_trace([1, 3, 7])[1]) == 1

# 10. 스냅샷은 결과 리스트의 후속 변경에 영향을 받지 않는다.
result, snapshots = bubble_trace([9, 1, 6, 3])
saved = [row.copy() for row in snapshots]
result.append(99)
assert snapshots == saved
print("PASS: 7 input/invariant cases, stability, early exit, snapshot independence")
```

## 비용을 설명할 때의 주의점

키 평가와 비교가 O(1)이라는 전제에서 비교 작업은 최악 O(n²), 조기 종료가 있는 정렬된 입력은 O(n)이다. 스냅샷을 저장하는 이 추적 도구는 일반 제자리 정렬과 다르다. 입력 복사에 O(n), 최대 n−1개의 길이 n 스냅샷에 O(n²) 추가 공간을 쓴다. 교육용 추적 비용을 제외한 버블 정렬의 공간 설명과 혼동하지 않는다.

## 출처와 검증 범위

[개인 학습 대화의 버블 정렬 설명](https://chatgpt.com/c/6aab7fe3-b918-83ee-8b6b-bbb817336abf?messageId=61644134-9ebb-4d90-8e42-90e5264139d1)의 실제 설명 본문을 읽고 개념만 참고했다. 원래 질문의 이미지 바이트는 확보되지 않았으며, 이미지·원본 코드·시험 표현·대화 전문은 재배포하지 않는다. 출처 링크는 저장소 독자에게 열리지 않을 수 있다.

2026-11-01 UTC에 이 문서의 정확한 Python 블록을 실행해 10개 사례를 검증했다. 기존 C 프로그램 전체 실행이나 성능 측정 결과가 아니다.

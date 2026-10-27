# 스택·큐: 반환값과 남은 상태를 따로 추적하기

스택은 마지막에 넣은 항목을 먼저 꺼내고(LIFO), 큐는 먼저 넣은 항목을 먼저 꺼낸다(FIFO). 같은 값을 넣어도 제거하는 끝이 다르면 결과가 달라진다. 제거 연산을 읽을 때는 반환된 값과 컨테이너에 남은 항목을 별도로 기록한다.

## 예제의 계약

- 아래 함수는 입력을 복사하므로 호출자의 목록을 수정하지 않는다.
- 스택은 list.append/pop, 큐는 deque.append/popleft를 사용한다.
- 빈 컨테이너에서 제거하면 IndexError가 발생한다. 예제에서는 예외를 임의의 값으로 숨기지 않는다.
- 숫자가 중복되어도 개별 삽입 순서는 보존된다.

## 독립 작성 구현과 재현 검사

```python
from collections import deque


def drain_stack(values):
    work = list(values)
    removed = []
    while work:
        removed.append(work.pop())
    return removed


def drain_queue(values):
    work = deque(values)
    removed = []
    while work:
        removed.append(work.popleft())
    return removed


cases = [[], [7], [7, 2, 9], [5, 5, 1], [-2, 0, 3]]
for values in cases:
    original = values.copy()
    assert drain_stack(values) == values[::-1]
    assert drain_queue(values) == values
    assert values == original

stack = [4, 8]
queue = deque([4, 8])
stack.append(12)
queue.append(12)
last = stack.pop()
first = queue.popleft()
assert (last, first) == (12, 4)
assert stack == [4, 8] and list(queue) == [8, 12]
stack.append(first)
queue.append(last)
assert stack == [4, 8, 4] and list(queue) == [8, 12, 12]

for remove in (list().pop, deque().popleft):
    try:
        remove()
    except IndexError:
        pass
    else:
        raise AssertionError("empty removal must fail")
print("PASS: 5 input cases, 1 interleaving scenario, 2 empty-removal cases")
```

2026-10-27 UTC에 Python 3.12.14로 문서의 코드 블록을 추출하여 실행했다. 입력 사례 5개, 교차 삽입·제거 시나리오 1개, 빈 상태의 예외 검사 2개를 통과했다. 저장소의 기존 C 프로그램이나 앱 전체 테스트를 실행한 것은 아니다.

## 상태표를 읽는 요령

반환값 last=12와 first=4는 제거 전 항목에서 결정된다. 제거 직후 스택에는 [4, 8], 큐에는 [8, 12]가 남는다. 반환값을 다시 삽입하면 새로운 항목이 들어가는 것이지, 제거 연산이 취소되거나 같은 위치로 되돌아가는 것이 아니다.

- list.pop(0)도 FIFO 순서를 만들 수 있지만 앞 항목 제거에 뒤 항목 이동 비용이 발생한다. 반복적인 앞 제거에는 deque.popleft가 적절하다.
- list.append와 끝 pop은 상각 O(1), deque 양 끝 추가·제거는 O(1) 수준으로 다룬다. 이 예제의 전체 drain은 O(n) 시간과 복사·결과 목록을 포함한 O(n) 추가 공간을 쓴다.
- 실제 병렬 생산자·소비자 큐, 용량 제한, blocking 동작은 별도 계약이 필요하며 이 예제의 범위 밖이다.

## 출처와 재사용 범위

[개인 학습 대화의 스택·큐 설명](https://chatgpt.com/c/6ac32004-6964-83ee-ab27-cc318e4bb112?messageId=775c8351-6d33-4de6-9502-4b7ea3739329)을 실제 읽고 개념만 참고했다. 문장·구현·입력·테스트는 이 노트를 위해 새로 구성했다. 강의·시험·과제 문제 원문, 개인 채점 기록, 대화 전문은 포함하지 않는다. 비공개 대화 링크는 소유자 외에는 열리지 않을 수 있다.

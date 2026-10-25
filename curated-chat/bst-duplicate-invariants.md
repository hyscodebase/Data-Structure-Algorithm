# 이진탐색트리: 중복값 규칙과 전체 서브트리 조건

이진트리는 자식 수가 최대 두 개인 구조다. 이진탐색트리(BST)는 그 구조에 값의 배치 계약을 더한다. 이 문서는 정수 키를 저장하며, 같은 값도 별도 노드로 남기고 오른쪽으로 보내는 계약을 사용한다.

## 먼저 정할 계약

- 각 노드의 왼쪽 서브트리에는 해당 키보다 작은 값만 둔다.
- 오른쪽 서브트리에는 해당 키보다 크거나 같은 값을 둔다.
- 중복값 정책은 유일하지 않다. 중복을 거부하거나 한 노드에 횟수를 저장하는 구현도 가능하다.
- 탐색은 매번 현재 노드와 비교한다. 오른쪽으로 한 번 이동한 뒤에도 다음 노드에서는 왼쪽으로 갈 수 있다.
- 바로 옆 자식과의 비교만 맞아서는 부족하다. 조상에게서 물려받은 상한·하한도 모든 후손이 지켜야 한다.

예를 들어 15, 9, 24를 넣은 뒤 15를 다시 넣으면, 루트에서는 오른쪽으로 가고 24에서는 왼쪽으로 간다. 중복 15는 24의 왼쪽 자식이지만, 여전히 루트 15의 오른쪽 서브트리에 있다.

## 독립 작성 예제와 검증

아래 코드는 교육용이며 균형 조정이나 삭제 기능은 없다. 빈 트리를 None으로 표현한다. 삽입 결과의 루트를 반드시 다시 받는다.

```python
from dataclasses import dataclass

@dataclass
class Node:
    key: int
    left: object = None
    right: object = None


def insert(root, key):
    if root is None:
        return Node(key)
    current = root
    while True:
        side = "left" if key < current.key else "right"
        child = getattr(current, side)
        if child is None:
            setattr(current, side, Node(key))
            return root
        current = child


def valid(root, lower=None, upper=None):
    if root is None:
        return True
    if lower is not None and root.key < lower:
        return False
    if upper is not None and root.key >= upper:
        return False
    return (valid(root.left, lower, root.key)
            and valid(root.right, root.key, upper))


def inorder(root):
    if root is None:
        return []
    return inorder(root.left) + [root.key] + inorder(root.right)


cases = [[], [15], [15, 9, 24, 15], [4, 4, 4],
         [1, 2, 3, 4], [4, 3, 2, 1], [-3, 0, -8, -3]]
for values in cases:
    root = None
    for value in values:
        root = insert(root, value)
    assert valid(root)
    assert inorder(root) == sorted(values)

# 자식 간 비교만으로 놓치는 조상 상한 위반
assert not valid(Node(15, Node(9, right=Node(18))))
# 같은 값은 왼쪽에 둘 수 없다.
assert not valid(Node(15, Node(15)))
root = None
for value in [15, 9, 24, 15]:
    root = insert(root, value)
assert root.right.left.key == 15
print("PASS: 7 construction cases, 2 invalid trees, 1 duplicate path")
```

2026-10-25 UTC에 Python 3.12.14로 위 코드 블록을 그대로 추출해 실행했다. 구성 사례 7개, 잘못된 트리 2개, 중복 삽입 경로 1개가 통과했다. 기존 저장소의 C 코드나 전체 애플리케이션 테스트는 아니다.

## 복잡도와 한계

삽입은 높이를 h라 할 때 O(h) 시간과 O(1) 추가 작업 공간을 쓴다(새 노드 공간 제외). 균형이 유지되면 높이가 O(log n)이지만, 이 예제는 균형을 보장하지 않으므로 정렬된 입력이나 많은 중복값에서 O(n) 높이가 될 수 있다. 검증 함수는 각 노드를 한 번 방문하고 재귀 호출 공간 O(h)를 쓴다. inorder는 간결한 검사용 코드이며 리스트 연결을 사용하므로 효율적인 대규모 순회 구현으로 제시하지 않는다. 깊은 트리에서는 Python 재귀 제한에도 주의한다.

## 출처와 제외 범위

[개인 학습 대화](https://chatgpt.com/c/6ac32004-6964-83ee-ab27-cc318e4bb112?messageId=775c8351-6d33-4de6-9502-4b7ea3739329)의 BST 설명 부분을 실제 읽고 개념만 참고했다. 문장·구현·입력·검사는 이 노트를 위해 새로 작성했으며, 강의자료·시험 문제·개인 채점 기록·대화 전문을 복제하지 않았다. 링크는 소유자 외에는 열리지 않을 수 있다.

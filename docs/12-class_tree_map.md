# 11. TreeMap 클래스
이진 탐색 트리(Binary Search Tree)에 대한 내용을 공부한다. 요소가 정렬된 Map 인터페이스를 구현할 떄 유용하게 사용된다.

## 해싱의 문제점
HashMap은 핵심 메서드 성능 때문에 널리 사용되지만 다른 Map 인터페이스 구현체가 필요한 이유가 몇 가지 존재한다.

- HashMap 연산이 상수시간이라고 해도 해싱이 느릴 수 있다.
- 좋은 해시 함수를 설계하는 것은 어려운 일이다. 특정 하위맵에 몰리면 성능이 나빠진다.
- 해시 테이블의 키는 어떤 순서를 가지고 저장되지 않는다.

위에 나열된 문제를 어느 정도 해결하는 TreeMap 클래스가 존재한다.

- TreeMap은 `해시 함수를 사용하지 않는다.
- TreeMap 클래스의 내부에서 키는 이진 탐색 트리에 저장되는데 선형 시간으로 키를 순회할 수 있다.
- 핵심 메서드의 실행시간은 log n에 비례하며 상수에는 못 미치지만 우수하다.

## 이진 탐색 트리
이진 탐색 트리는 다음과 같은 속성을 가지고 있다. 
[위키 참조](https://en.wikipedia.org/wiki/Binary_search_tree)

- 각 노드가 키를 포함한다.
- 노드 왼쪽에 자식이 있다면 왼쪽 하위 트리의 모든 키는 노드에 있는 키보다 작다.
- 노드 오른쪽에 자식이 있다면 오른쪽 하위 트리의 모든 키는 노드에 있는 키보다 크다.

트리를 전부 검색할 필요가 없기 때문에 검색 속도가 빠르다. 루트에서 시작하여 다음과 같은 알고리즘을 사용할 수 있다.

- `찾는 키`를 `현재의 노드`의 키와 비교한다. 같다면 검색 완료.
- `찾는 키`가 현재 키보다 작으면 왼쪽 트리를 검색한다. 왼쪽에 트리가 없다면 값은 존재하지 않는다.
- `찾는 키`가 현재 키보다 크면 오른쪽 트리를 검색한다. 오른쪽에 트리가 없다면 값은 존재하지 않는다.

일반적으로 검색시에 비교 횟수는 트리에 있는 키의 개수보다 `트리의 높이에 비례`한다. 트리의 높이인 h와 노드의 개수의 관계 n 사이의 관계는 다음과 같다.

- h = 1이면, 1개의 노드 뿐이므로 n = 1
- h = 2면, 2개의 노드를 추가하여 n = 3
- h = 3이면, 4개의 노드를 추가하여 n = 7
- h = 4면, 8개의 노드를 추가하여 n = 15

트리를 1부터 h 높이까지 늘린다고 가정하면, 각 높이에서 추가되는 노드의 개수는 `2^i-1`이다. 그리고 전체 노드 개수 n은 `2^h - 1`이 된다.

`n = 2^h - 1`의 양변에 log를 씌우면 대략 `log n = h`가 된다. 이는 트리가 가득 차면 트리 높이는 log n에 비례한다는 의미다. 따라서 이진 탐색 트리에서는 `log n에 비례하는 시간`으로 키를 찾을 수 있을 거라 기대한다.

log n에 비례하는 시간이 걸리는 알고리즘을 로그 시간(logarithm 혹은 log time)이라고 한다. 증가 차수는 O(log n)에 해당한다.

## 실습
- MyTreeMap.java

```java
public class MyTreeMap<K, V> implements Map<K, V> {

	private int size = 0;
	private Node root = null;

    @Override
    public int size() {
        return size;
    }
    
    @Override
    public void clear() {
        size = 0;
        root = null;
    }
}
```

- size는 확실하게 상수시간이다.
- clear는 상수시간으로 보이나 root를 null로 설정할 때 가비지 콜렉터에서 수집할 때 선형시간이 걸린다.

## TreeMap 구현
- MyTreeMap.java
- MyTreeMapTest.java

```java
private Node findNode(Object target) {
    if (target == null) {
        throw new IllegalArgumentException();
    }

    Comparable<? super K> k = (Comparable<? super K>) target;

    Node node = root;

    while (node != null) {
        int compareTo = k.compareTo(node.key);
        if (compareTo == 0) {
            return node;
        }

        node = compareTo < 0 ? node.left : node.right;
    }

    return null;
}
```

```java
@Override
public boolean containsValue(Object target) {
    return containsValueHelper(root, target);
}

private boolean containsValueHelper(Node node, Object target) {
    Deque<Node> stack = new ArrayDeque<>(size);

    if (node != null) {
        stack.push(node);
    }

    while (!stack.isEmpty()) {
        Node n = stack.pop();

        if (equals(n.value, target)) {
            return true;
        }

        if (n.right != null) {
            stack.push(n.right);
        }

        if (n.left != null) {
            stack.push(n.left);
        }
    }

    return false;
}
```

```java
@Override
public V put(K key, V value) {
    if (key == null) {
        throw new NullPointerException();
    }
    if (root == null) {
        root = new Node(key, value);
        size++;
        return null;
    }
    return putHelper(root, key, value);
}

private V putHelper(Node node, K key, V value) {
    Comparable<? super K> k = (Comparable<? super K>)key;

    while (node != null) {
        int compareTo = k.compareTo(node.key);
        if (compareTo == 0) {
            V oldValue = node.value;
            node.value = value;
            return oldValue;
        }

        Node next = compareTo < 0 ? node.left : node.right;
        if (next != null) {
            node = next;
            continue;
        }

        if (compareTo < 0)  {
            node.left = new Node(key, value);
        }
        else {
            node.right = new Node(key, value);
        }

        size++;
        break;
    }

    return null;
}
```

```java
@Override
public Set<K> keySet() {
    Set<K> set = new LinkedHashSet<K>();
    keySetHelper(root, set);
    return set;
}

private void keySetHelper(Node node, Set<K> set) {
    if (node == null) return;
    keySetHelper(node.left, set);
    set.add(node.key);
    keySetHelper(node.right, set);
}
```

---
[Home](../README.md)

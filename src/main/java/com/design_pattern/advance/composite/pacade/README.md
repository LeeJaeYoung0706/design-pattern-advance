## 상속보다는 컴포지트 패턴을 사용하라 (1)

### 패키지 경계를 넘어 다른 패키지의 구체 클래스를 상속하는 것은 위험
- 상위 클래스에서 제공하는 메서드 구현이 변경된다면,
- 상위 클래스에서 새로운 메서드가 생성된다면,

### 컴포지션
- 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조
- 새 클래스의 인스턴스 메서드들은 기존클래스에 대응하는 메서드를 호출
- 기존 클래스의 구현이 변경되거나 세로운 메서드가 생기더라도 아무런 영향을 받지 않음.

```java
package com.design_pattern.advance.pacade;

import java.util.Collection;
import java.util.HashSet;
import java.util.List;

public class InstrumentedHashSet<E> extends HashSet<E> {

    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    public int gatAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        com.design_pattern.advance.composite.pacade.InstrumentedHashSet<String> s = new com.design_pattern.advance.composite.pacade.InstrumentedHashSet<>();
        s.addAll(List.of("test", "test1", "test2"));
        System.out.println(s.gatAddCount());
    }
}
```
- 해당 코드에서 addAll을 호출할 경우 부모 객체인 HashSet 의 addAll 함수의 프로세스 가 노출될 가능성 및 해당 함수를 알아야하는 번거로움 존재 
- ex ) 개발자는 3을 예상하지만 결과 값은 6
```java
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
```

- 해당 문제를 해결하기 위해 ForwardSet 을 생성하고 멤버변수로 상속 할 부모객체를 가지고 있게 설계하면,

```java
package com.design_pattern.advance.pacade;

import java.util.Collection;
import java.util.Iterator;
import java.util.Set;

// 재사용 전달 클래스
public class ForwardingSet<E> implements Set<E> {

    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    @Override
    public int size() {
        return s.size();
    }

    @Override
    public boolean isEmpty() {
        return s.isEmpty();
    }

    @Override
    public boolean contains(Object o) {
        return s.contains(o);
    }

    @Override
    public Iterator<E> iterator() {
        return s.iterator();
    }

    @Override
    public Object[] toArray() {
        return s.toArray();
    }

    @Override
    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    @Override
    public boolean add(E e) {
        return s.add(e);
    }

    @Override
    public boolean remove(Object o) {
        return s.remove(o);
    }

    @Override
    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    @Override
    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    @Override
    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    @Override
    public void clear() {
        s.clear();
    }
}

```
- 위의 예와 다르게 super.addAll()을 호출하더라도 멤버변수로 가지고 잇는 set의 addAll이 호출됨.
따라서 사이드 이펙트를 없앨 수 있음. 캡슐화를 지킬 수 있음.
멤버변수로 선언된 HashSet() 에 추가되어도 해당 코드는 변화하지 않아도 정상적으로 돌아갈 수 있음.
- 또한, interface로 설계할 경우 구현체인 ForwardSet 에서 구현하지 않는다면 컴파일 오류가 발생함으로 이를 알아내 빠르고 쉽게 대응할 수 있다.

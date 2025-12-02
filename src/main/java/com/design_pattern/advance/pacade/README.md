## 상속보다는 컴포지트 패턴을 사용하라

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
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("test" , "test1" , "test2"));
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
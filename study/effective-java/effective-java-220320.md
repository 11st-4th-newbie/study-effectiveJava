## Item17 : 변경 가능성을 최소화하라
- Map의 key와 value는 변하지 않는 것으로 만들어야 하기 때문에 불변객체로 구성하는 것이 좋다는 말을 들었다. 
그렇다면 Map에 특정 키와 값을 매핑해주는 엔트리가 SimpleImmutableEntry(불변), SimpleEntry(가변)가 있는데 어떤 것이 쓰이는걸까? 

> `AbstractMap.SimpleImmutableEntry`와 `AbstractMap.SimpleEntry` ➡️ Map에 있는 단일 요소에 해당하는 키와 값을 가지는데 사용
> - SimpleEntry는 값을 바꿀 수 ⭕️
> - SimpleImmutalbeEntry는 값을 바꾸려고 하면 UnSupportedOperationException 발생
```java
public static class SimpleEntry<K,V> implements Entry<K,V>, java.io.Serializable
{
  private static final long serialVersionUID = -8499721149061103585L;

  private final K key;
  private V value;
        
  public V setValue(V value) {
    V oldValue = this.value;
    this.value = value;
    return oldValue;
  }
}
```
```java
public static class SimpleImmutableEntry<K,V> implements Entry<K,V>, java.io.Serializable
{
  private static final long serialVersionUID = 7138329143949025153L;

  private final K key;
  private V value;
        
  public V setValue(V value) {
    throw new UnsupportedOperationException();
  }
}
```
  - Map에서 사용하는 Entry는 구현하는 형태에 따라 다르다. 
  - SimpleEntry와 SimpleImmutableEntry를 정의한 AbstractMap 클래스를 상속하는 Map의 구현체들 (HashMap, TreeMap, …) 은 앞서 말한 2가지의 static class를 사용하는 것이 아닌 자신만의 Entry를 정의하여 따로 정의한 Entry를 사용하는 모습을 볼 수 있다. 
    - HashMap - Node<K, V> implements Map.Entry<K, V>
    ![17AC3190-5A83-4257-A8E1-116CB17CE69C](https://user-images.githubusercontent.com/61124319/160275429-a48a6f23-939c-4966-a128-f1c0ac400677.png)
    - TreeMap - Entry<K, V> implements Map.Entry<K, V>
    ![B69E2B4F-6372-42B8-B5E2-E55CA67331E6](https://user-images.githubusercontent.com/61124319/160275482-c25aa03b-1456-4334-9565-375b868782b0.png)
```
정리
1. SimpleEntry와 SimpleImmutableEntry는 HashMap, TreeMap 등 우리가 흔히 아는 Map의 구현체에서 사용하는 것이 아닌 것 같다. 
2. Java 6.0에 등장한 개념으로 이전에 생성된 구현체에서는 사용하지 않는다. 
```
> 추가정리 :
> 
> **굳이** 구현체들의 Entry가 SimpleEntry인지 SimpleImmutableEntry인지를 따지자면, 각 구현체 내에서 setValue 구현방식을 보면 어떤 개념을 사용하고 있는지 알 수 있다. 하지만 이는 SimpleEntry 혹은 SimpleImmutableEntry와 정확하게 동일하다는 의미가 아니다. 
> - 예를 들어, HashMap은 SimpleEntry의 **개념**이라고 볼 수 있다.  HashMap에서 구현된 Node의 경우, 기존에 존재하는 key에 대한 setValue를 할 때, 어떠한 에러 없이 **정상작동**하는 것을 확인할 수 있다. 이러한 것으로 미루어봤을 때, SimpleEntry와 유사하게 동작한다는 것을 알 수 있다.



- 가변 동반 클래스를 사용하면 다단계 연산 속도를 높여주는 이유는 뭘까? e.g. String, BigInteger
	- BigInteger는 MutableBigInteger 등의 가변 동반 클래스를 package-private하게 두고 있다. ![594C1E0D-854A-46A3-BEB2-E5C3BC1CB038](https://user-images.githubusercontent.com/61124319/160275498-6c854545-3df4-4489-bd14-fb1e6f75e07c.png)
	- BigInteger의 mod를 보면 this.remainder 를 호출하고 있다. remainder로 가면 remainderKnuth 등을 호출하고 remainderKnuth 내부에서는 MutableBigInteger라는 package-private한 가변 동반 클래스를 사용하고 있다. 
	- BigInteger 내부에서 MutableBigInteger와 같은 가변동반 클래스를 사용하여 다단계 연산을 제공하고 있기에 각 단계마다 BigInteger 객체를 새로 생성하지 않아도 된다. 
`정리 : 객체를 생성하지 않아도 되기 때문이다.`

- negate 메서드는 크기가 같고 부호만 반대는 새로운 BigInteger를 생성하는데, 이때 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유해도 된다.‘ 라고 하는데 왜 가능한 것인가?
	- negate 메서드는 어떠한 객체의 값에서 부호만 바뀐 객체를 반환한다. 그래서 원본 객체의 값이 바뀌어도 negate 메서드 의미를 해치지 않는다. 그렇기 때문에 배열이 가변이긴 하지만, 여기선 내부 데이터로 공유해도 문제가 없다. 

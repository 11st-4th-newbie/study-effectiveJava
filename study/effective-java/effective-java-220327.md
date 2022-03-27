## Item18 : 상속보다는 컴포지션을 사용하라

### Q. 래퍼클래스랑 일급컬렉션의 차이가 뭐지?
`래퍼클래스(Wrapper Class)`는 프로그램에 따라 기본 타입의 데이터를 객체로 취급해야 하는 경우가 있는데 기본 타입에 해당하는 데이터를 객체로 포장해주는 클래스이다. 

`일급컬렉션`은 Collection을 Wrapping한 래퍼클래스로, 해당 Collection 외 다른 멤버변수가 없는 상태를 말한다. [일급컬렉션이란?](https://jojoldu.tistory.com/412)

### Q. 상속이랑 컴포지션이랑 뭔 상관?
:point_right: 상속이 문제가 많으니까 상속보다는 컴포지션을 활용하라는거지, 상속의 문제를 컴포지션으로 해결해라 뭐 이런 이야기는 아니다! 상속은 메소드 호출과는 달리 캡슐화를 깨버리거든. 그러니까 조합으로 내부 객체를 갖고 거기서 메소드를 호출해버리자. 캡슐화 깨먹지 말고....

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet(){}

    public InstrumentedHashSet(int initCap, float loadFactor){
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

public class Item18 {
    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));

        System.out.println(s.getAddCount()); //3을 반환할 것 같지만 6을 반환하네?
    }
}
```
### 위 InstrumentedHashSet에서 발생하는 문제

main의 getAddCount가 3을 반환할 것 같았는데, 6을 반환한다.

#### 어째서 그런 문제가 생겼는지 절차로 알아보자.

1. InstrumentedHashSet의 addAll은 addCount에 3을 더한 뒤 마지막에 super.addAll을 통해 부모인 HashSet의 addAll을 호출한다.
2. 근데 그 부모인 HashSet의 addAll은 아래에 코드를 첨부했는데, 구현 내용을 보면 각 원소를 add 메소드를 호룰해서 추가하고 있다. 
<img width="676" alt="image" src="https://user-images.githubusercontent.com/35680213/160293969-2981a243-5f07-46ba-a649-ac1e8b22859e.png">
3. 그래서 그 add를 호출하려고 봤더니 InstrumentedHashSet이 그 add를 오버라이드(재정의) 해놓은거야! 그럼 얘가 호출되겠지?!
4. 위 코드에서 보이다시피 InstrumentedHashSet이 재정의한 HashSet의 add은 addCount를 더해주고 있어. 
5. 이렇게 재정의된 메소드로서 add가 세 번 실행이 되니까 addCount가 최종적으로 6이 되는거야!


> 위처럼 HashSet을 바로 상속하면 그냥 HashSet이 내부에서 자신(HashSet)의 add를 호출하더라도 그게 호출되지 않고 자식이 재정의한 add가 호출되는거임.
이미 내가 HashSet의 메소드인 add를 오버라이드해놔서 자꾸만 재정의된 이 메소드가 호출되고 있다.
난 이게 싫음!!! HashSet만의 본래 add메소드를 멀쩡하게 호출하고 싶어! 자식이 재정의한 메소드 말고!      
   

:point_right: 그래서! 이번엔 ForwardingSet이라는 중간부모를 하나 만들거야. 얘는 어떤 메소드 안에서 
다른 메소드(즉. 누군가에 의해 재정의될 가능성이 있는 메소드)를 또다시 호출해서 혼란이 생기는 일은 없게 할 거야. 그냥 지가 가진 객체에 바로 작업을 걸어버릴거야.
메소드 안에 또다른 메소드 호출이 없게 할거야. 괜히 그랬다가는 그 호출되는 메소드를 자식이 재정의 해버리면 또 의도치 않게 자신의 메소드가 아닌 자식의 재정의 메소드가 호출되어 문제가 발생할테니까.        


```java
public class ForwardingSet<E> implements Set<E> {
   private final Set<E> s;
   public ForwardingSet(Set<E> s) {this.s = s;}
   
   public boolean add(E e) {
       return s.add(e);
   }

   public boolean addAll(Collection<? extends E> c) {
      return s.addAll(c);
   } //간략히 보여주려고 필요한 메소드만 썼다.
}

public class InstrumentedSet<E> extends ForwardingSet<E> {
   private int addCount = 0;

   public InstrumentedSet(Set<E> s) {
      super(s);
   }

   @Override
   public boolean add(E e) {
      addCount++;
      return super.add(e);
   }

   @Override
   public boolean addAll(Collection<? extends E> c) {
      addCount += c.size();
      return super.addAll(c);
   }

   public int getAddCount() {
      return addCount;
   }
}

public class Item18 {
   public static void main(String[] args) {
      InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
      s.addAll(List.of("틱", "탁탁", "펑"));
      System.out.println(s.getAddCount());

      InstrumentedSet<String> s2 = new InstrumentedSet<>(new HashSet<String>());
      s2.addAll(List.of("틱", "탁탁", "펑"));
      System.out.println(s2.getAddCount());
   }
}
```

:point_right: ForwardingSet을 상속하면 얘는 다른 메소드 호출을 안하고 바로 자기 내부 객체에 작업을 걸어버리니까 재정의된 다른 메소드가 호출될 일이 없겠지?
애초에 다른 메소드를 메소드 안에서 또다시 호출하지도 않고 그냥 s에 때려넣으니까.

:point_right: 이러이러한 문제가 있으니까 상속 함부로 쓰지 말아라. 컴포지션을 써라....


### Q. 아래 문장에서 `자기자신을 참조`하는 건 무슨 의미일까?
> 다른 Set의 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다. 컴포지션과 전달의 조합은 넓은 의미로 위임이라고 부른다. 단, 엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당된다.(p.119)`

:point_right: 현재 상황에서 래퍼 객체는 ForwardingSet이고, 내부 객체는 ForwardingSet의 멤버변수인 s다. 
즉 외부에서 ForwardingSet이 참조되면서 add라는 메소드가 호출됐을 때 그 내부 객체인 s가 참조되면서 s.add가 실행되고 있다. 
이 상황이 지금 래퍼 객체(ForwardingSet)가 자기 자신의 참조(this.add, 즉 ForwardingSet의 add, 자기자신을 참조하는 것)를
내부 객체인 s 에게 넘기는(위임하는) 경우다.


#### 추가내용
> 래퍼클래스는 단점이 거의 없지만 한 가지, `래퍼 클래스가 콜백 프레임워크와는 어울리지 않는다는 점만 주의하면 된다.` 콜백 프레임워크에서는 자기 자신의 참조(this)를 다른 객체에 넘겨서 
> 다음 호출(==콜백) 때 사용하도록 한다. 여기서 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니까 내부 객체 관점에서 자신의 참조(내부 객체를 참조하는 바로 그 참조)를 넘기고,
> 결국 이로 인해 콜백 때는 래퍼객체가 아닌 내부 객체를 호출하게 된다.

## Item28 : 배열보다는 리스트를 사용하라
### Q. 코드 28-3 이해 (p.165) 여기서 언제 stringLists[0]에 Integer 값이 저장된 것인가?
```java
String s = stringLists[0].get(0);
```
:point_right: 이 코드에서 같은 주소값을 바라보는 객체에 objects[0] = intList 했으니 이때 Integer값이 저장된 것 같다.
```java
// 여기서 objects와 stringLists가 같은 주소값을 바라보게 됨
Object[] objects = stringLists;
objects[0] = intList; 
```

### Q. 아래 문장에 대해, 내 생각이 맞는지?
`배열은 공변이라서 Sub가 Super의 하위타입이라면 배열 Sub[] 는 배열 Super[]의 하위 타입이다.
제네릭은 불공변이라 List<Sub> 는 List<Super>의 하위 타입도 아니고 상위 타입도 아니다.`

```java
Super[] supers; 라고 선언하고 초기화할 때는
        supers = new Sub[]라고 해도 문제가 안생기는데
        ArrayList<Super> supers;
        supers = new ArrayList<Sub>;
라고 하면 안되니까 리스트는
        List<Sub>와 List<Super>가 서로 상위도 하위도 아닌 아예 다른 관계라는 말인가? 
```
:point_right: ㅇㅇ 맞아 그냥 List<Sub> 랑 List<Super>는 어떤 관계도 갖지 않는다고 이해.

### Q. 아래 문장에 대해 p.306
`(같은패키지에 속하는 이유로) 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있다.`

:point_right: 같은 패키지라면 내부 동작을 잘 이해하고 있다고 가정, 수정하지 않을 것이라고 확신할 수 있다.


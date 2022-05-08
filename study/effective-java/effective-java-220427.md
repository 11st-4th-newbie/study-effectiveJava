# Item 10 : equals 는 일반 규약을 지켜 재정의하라
`@EqualsAndHashCode` 애노테이션에 include 등의 필드를 지정해주는 속성들을 사용하고 build해보면 아래와 같이 `instance of` 로 구현되어 있음.
```java
public boolean equals(final Object o) {
    if (o == this) {
        return true;
    } else if (!(o instanceof DisplaySpaceCategory)) {
        return false;
    } else {
        DisplaySpaceCategory other = (DisplaySpaceCategory)o;
      if (!other.canEqual(this)) {
          return false;
      } else {
          label47: {
              Object this$displaySpaceV1 = this.getDisplaySpaceV1();
              Object other$displaySpaceV1 = other.getDisplaySpaceV1();
          if (this$displaySpaceV1 == null) {
              if (other$displaySpaceV1 == null) {
                  break label47;
              }
          } else if (this$displaySpaceV1.equals(other$displaySpaceV1)) {
              break label47;
          }
  
          return false;
      }
  
      Object this$category = this.getCategory();
      Object other$category = other.getCategory();
      if (this$category == null) {
          if (other$category != null) {
              return false;
          }
      } else if (!this$category.equals(other$category)) {
          return false;
      }
  
      Object this$subCategory = this.getSubCategory();
      Object other$subCategory = other.getSubCategory();
      if (this$subCategory == null) {
          if (other$subCategory != null) {
              return false;
          }
      } else if (!this$subCategory.equals(other$subCategory)) {
          return false;
      }
  
      return true;
    }
  }
}
```
### 🤔 추이성 등의 규약이 깨지는 예시가 무엇이 있을까?
→ 책에서 예시로 나온 상속관계 객체에서의 비교? 이렇게 비교하는 일이 있을까?

# Item 11 : equals를 재정의하려거든 hashCode도 재정의하라
### 🤔 equals, hashcode를 커스텀하게 작성할 일이 있을까?
→ 애노테이션이나, IDEA 에서 잘 지원을 해줘 거의 없는 것 같다.

# Item 55 : 옵셔널 반환은 신중히 하라
### 🤔Empty List 를 자주 사용하는지 Optional 을 자주 사용하는지?
→ 컬렉션 타입에서 굳이 옵셔널 값을 판단하는 추가 코드를 작성하는 것은 번거롭다. 그냥 빈 리스트를 돌려주어 예외 걱정없이 사용한다.    
  옵셔널은 비어있다는 의미를 나타내기 어려운 타입에서 주로 사용한다고 생각한다.


# Item 65 : 리플렉션보다는 인터페이스를 사용하라
### 🧐 리플렉션의 단점
- 느리다
- 컴파일 타입 검사의 이점을 누릴 수 없다.
- 코드가 지저분하다.

때문에 리플렉션은 인스턴스의 생성에만 사용하고, 이를 인터페이스나 상위 클래스로 참조해 사용하는 등 제한된 형태로 사용하자.

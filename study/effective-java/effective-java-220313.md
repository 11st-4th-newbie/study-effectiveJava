## Item1 : 생성자 대신 정적 팩터리 메서드를 고려하라 
- 정적팩토리 메서드와 디자인 패턴의 팩터리 메서드가 다르다고 하는데 왜 다른지 모르겠다. 
    - 정적팩토리는 이름을 부여하거나 등의 목적을 위해 사용하는것이고 팩터리 메서드 패턴은 생성할 인스턴스의 결정 책임을 가지기 때문에 다르다고 본다.
    - 팩토리 패턴은 조건에 따라 생성되어야 하는 인스턴스가 뭔지 결정하는 역할을 가진 클래스가 하나 추가된다, 근데 정적팩토리메소드는 다른 인스턴스를 반환하고 싶으면 서로 다른 인스턴스의 정적팩토리메소드로 구현되어 있음
    
    `정리 : 팩토리 메서드 패턴은 추상화를 통해 하위 클래스에 구현을 위임하는것이다. 정적 팩토리 메서드는 객체 생성의 역할을 하는 클래스 메서드의 의미로 고려할 수 있다.`

- 플라이웨이트 패턴에서 목표는 인스턴스 통제가 맞는것일까요?
    - 재활용으로 사용하는것 같습니다.

    `정리 : 생성되는 객체의 수를 줄이고 메모리 공간을 줄이는 것으로 통제의 역할도 하는것 같습니다.`

- 싱글톤, 플라이웨이트 패턴의 차이는?
    - 싱글톤은 단 하나의 종류를 가지는것이고 플라이웨이트는 하나씩 여러종류를 가질 수 있는 객체를 생성할 수 있다. 
    - 플라이웨이트 패턴은 다른 종류가 필요한 경우라면 새로운 인스턴스 생성을 허용한다.

    ```html
    빨간 나무 인스턴스 1개, 초록 나무 인스턴스 1개….
    빨간 나무 인스턴스 2개 이상은 금지
    ```
    - 싱글톤 패턴은 같은 종류든 다른 종류든 새로운 인스턴스 생성을 모두 통제한다. 새 종류가 필요하다면 기존에 만들어둔 하나의 인스턴스의 종류를 변경해야 한다.

    ```html
    나무 무조건 1개. 빨간색을 원하면 빨간색 나무로 1개 가져야 하고, 초록색을 원한다면 기존의 빨간 나무를 초록색으로 바꿔서 나무 1개로 유지해야 함.
    ```

    `정리 : 싱글톤 패턴은 클래스 자체가 오직 하나의 인스턴스만을 허용하지만 플라이웨이트 패턴은 싱글톤이 아닌 클래스를 팩토리에서 제어하게 된다.`

    추가 정리 : 
    아래 클래스가 존재할 때 첫번째 인자는 변하는 것에 비해 두번째, 세번째, 네번째는 변하지 않는것을 확인할 수 있다. 이런 경우 플라이웨이트 패턴을 적용해볼 수 있다. 
    ```java
    public class Client {
 
    public static void main(String[] args) {
        Character c1 = new Character('h', "white", "Nanum", 12);
        Character c2 = new Character('e', "white", "Nanum", 12);
        Character c3 = new Character('l', "white", "Nanum", 12);
        Character c4 = new Character('l', "white", "Nanum", 12);
        Character c5 = new Character('o', "white", "Nanum", 12);
        }
    }   
    ```
    그래서 팩토리 메소드를 만들어주고
    ```java
    public class FontFactory {
 
    private Map<String, Font> cache = new HashMap<>();
 
    public Font getFont(String font) {
        if (cache.containsKey(font)) {
            return cache.get(font);
        } else {
            String[] split = font.split(":");
            // 폰트이름, 폰트사이즈 분리
            Font newFont = new Font(split[0], Integer.parseInt(split[1]));
            cache.put(font, newFont);
            return newFont;
            }
        }
    }
    ```
    
    Client에서는 기존에 만들어준 인스턴스를 반환해준다. 
    ```java
    public static void main(String[] args) {
        FontFactory fontFactory = new FontFactory();
        Character c1 = new Character('h', "white", fontFactory.getFont("nanum:12"));
        Character c2 = new Character('e', "white", fontFactory.getFont("nanum:12"));
        Character c3 = new Character('l', "white", fontFactory.getFont("nanum:12"));
    }
    ```
    [이 글](https://dev-youngjun.tistory.com/217) 을 참고해보자. 😊

## Item4 : 인스턴스화를 막으려거든 private 생성자를 사용하라.
   - 없음 (추가 시 작성해주세요.)
    
## Item17 : 변경 가능성을 최소화 해라.
   - public을 최소한으로 사용해라.
        - 105p, 마지막 부분 ( public final은 내부 표현을 바꾸지 못하므로 권하지 x) 이게 왜? public 으로 배포된 상태에서는 이미 참조된 관계가 존재할 수 있기 때문에 public -> private 로 바꾸는 식의 수정, 교체, 제거 등의 내부 변경이 어렵게 된다. 
    - 불변객체의 단점을 해결하기 위해 가변을 동반하는 클래스인 StringBuilder을 만들었다고 하는데 해당 StringBuilder가 어떻게 가변을 만드는가?
        - StringBuilder 코드를 확인해보면 abstract 에서 직렬화 하는것을 확인해볼 수 있다.
        - 
        `PATH : append > AbstractStringBuilder > putStringAt > inflate`
        
        ```java
            /**
            * If the coder is "isLatin1", this inflates the internal 8-bit storage
            * to 16-bit <hi=0, low> pair storage.
            */
            private void inflate() {
                if (!isLatin1()) {
                    return;
                }
                byte[] buf = StringUTF16.newBytesFor(value.length);
                StringLatin1.inflate(value, 0, buf, 0, count);
                this.value = buf;
                this.coder = UTF16;
            }
        ```
        
   - 불변객체 안에 정적 팩토리 메소드를 사용하는것이 불변성을 깨드리는것이 아닌지 궁금하다.
        - 하나의 수단으로 접근할 수 있다. 예를 들어서 정적 팩터리 메소드안에 생성자를 구현해서 객체를 생성할 수 있다.
   - 불변객체에서 개발자는 상태 변경을 하고 싶은데 불변객체이기 때문에 새로운 객체, 즉 상태가 다른 또다른 객체를 생성해내는 메소드가 필요하다. 이때 이 메소드는 정적 팩토리메소드일까? 
        - 불변객체라는 것 자체가 상태들을 (static이 아닌필드) 가지니까 애초에 static 메소드를 통해 이 값들을 변경할 수도 없음. 그래서 상태 변경을 
위한 새 객체 반환하는 메소드를 정적 팩토리메소드라고 볼 수는 없고, 최초 객체 생성을 위한 정적팩토리메소드와 이렇게 상태 변경을 원해서 새로운 객체를 생성하는 메소드는 둘을 구분짓고 싶다는 의견이 우세했음
    - 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉽다고 하는데 설계랑 구현이 왜 쉬운가? 
        - 방어적 복사, 에러 상황에 대한 처리를 많이 고려하지 않아도 된다.  


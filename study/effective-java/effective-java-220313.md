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







# equals & hashCode

equals와 hashCode 메서드는 객체의 동등성 비교와 해시값 생성을 위해 사용

함께 재정의하지 않으면 문제가 발생할 수 있음

```java
class EqualsHashCodeTest {

    @Test
    @DisplayName("equals만 정의하면 HashSet이 제대로 동작하지 않는다.")
    void test() {
        // 아래 2개는 같은 구독자
        Subscribe subscribe1 = new Subscribe("team.maeilmail@gmail.com", "backend");
        Subscribe subscribe2 = new Subscribe("team.maeilmail@gmail.com", "backend");
        HashSet<Subscribe> subscribes = new HashSet<>(List.of(subscribe1, subscribe2));

        // 결과는 1개여야하는데..? 2개가 나온다.
        System.out.println(subscribes.size());
    }

    class Subscribe {

        private final String email;
        private final String category;

        public Subscribe(String email, String category) {
            this.email = email;
            this.category = category;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Subscribe subscribe = (Subscribe) o;
            return Objects.equals(email, subscribe.email) && Objects.equals(category, subscribe.category);
        }
    }
}
```

해시값을 사용하는 자료구조는 hashCode 메서드의 반환값을 사용

hashCode 메서드의 반환 값이 일치한 이후 equals 메서드의 반환값이 참일 때만 논리적으로 같은 객체

위 코드에서 Subscribe 클래스는 hashCode 메서드를 제정의하지 않았기에,

Object 클래스의 기본 hashCode 메서드를 사용

Object 클래스의 기본 hashCode 메서드는 객체의 고유한 주소를 사용하기 때문에 객체마다 값이 다름

따라서 2개의 Subscribe 객체는 다른 객체로 판단되어 HashSet에서 중복 처리되지 않음
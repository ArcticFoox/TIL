# Strategy Pattern
객체들이 할 수 있는 행위 각가에 대해 전략 클래스를 생성하고,  
유사한 행위들을 캡슐화 하는 인터페이스를 정의하여,  
객체의 행위를 동적으로 바꾸고 싶은 경우 직접 행위를 수정하지 않고,  
전략을 바꿔주기만 함으로써 행위를 유연하게 확장하는 방법

## 구현
운송 수단의 예시로 움직이는 두 방식에 대해 Strategy 클래스 생성  
두 클래스는 move() 메서드를 구현하여, 어떤 경로로 움직이는지 구현  
```java
public interface MovableStrategy {
    public void move();
}
```
```java
public class RailLoadStrategy implements MovableStrategy{
    public void move(){
        System.out.println("선로를 통해 이동");
    }
}
```
```java
public class LoadStrategy implements MovableStrategy{
    public void move() {
        System.out.println("도로를 통해 이동");
    }
}
```
다음으론 어떻게 운직일 것인지에 대한 전략을 설정
```java
public class Moving {
    private MovableStrategy movableStrategy;

    public void move(){
        movableStrategy.move();
    }

    public void setMovableStrategy(MovableStrategy movableStrategy){
        this.movableStrategy = movableStrategy;
    }
}
```
```java
public class Bus extends Moving{

}
```
```java
public class Train extends Moving{

}
```
마지막으로 Client 를 구현
```java
public class Client {
    public static void main(String args[]){
        Moving train = new Train();
        Moving bus = new Bus();

        /*
            기존의 기차와 버스의 이동 방식
            1) 기차 - 선로
            2) 버스 - 도로
         */
        train.setMovableStrategy(new RailLoadStrategy());
        bus.setMovableStrategy(new LoadStrategy());

        train.move();
        bus.move();

        /*
            선로를 따라 움직이는 버스가 개발
         */
        bus.setMovableStrategy(new RailLoadStrategy());
        bus.move();
    }
}
```

출처  
https://victorydntmd.tistory.com/292
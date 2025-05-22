# 팩토리 패턴

인스턴스를 만드는 절차를 추상화하는 생성 패턴 중 하나

생성 패턴의 중요점

- 시스템이 어떤 Concrete Class를 사용하는지에 대한 정보를 캡슐화
- 이들 클래스의 인스턴스들이 어떻게 만들고 어떻게 결합하는지에 대한 부분을 완전히 가려줌

위 두가지를 특징을 통해 무엇이 생성되고, 누가, 어떻게, 언제 같은 육하원칙을 결정하는데 유연성 확보

## 팩토리 패턴이란?

객체를 생성하는 인터페이스를 미리 정의하지만,

인스턴스를 만들 클래스의 결정은 서브 클래스 쪽에서 결정하는 패턴

여러개의 서브 클래스를 가진 슈퍼 클래스가 있을 때,

들어오는 인자에 따라서 하나의 자식클래스의 인스턴스를 반환해주는 방식

→ 클래스의 인스턴스를 만드는 시점 자체를 서브 클래스로 미루는 것

**활용성**

- 어떤 클래스가 자신이 생성해야 하는 객체의 클래스를 예측할 수 없을 때
- 생성할 객체를 기술하는 책임을 자신의 서브클래스가 지정했으면 할 때

**Code**

```java
public interface Figure {
	void draw();
}

public class Circle implements Figure {

    @Override
    public void draw() {
    	System.out.println("Circle의 draw 메소드");
    }
    
}

public class Rectangle implements Figure {

    @Override
    public void draw() {
    	System.out.println("Rectangle의 draw 메소드");
    }
    
}

public class Square implements Figure {

    @Override
    public void draw() {
    	System.out.println("Square의 draw 메소드");
    }
    
}

public class FigureFactory {
    public Figure getFigure(String figureType) {
    	if(figureType == null) {
            return null;
        }
        if(figureType.equalsIgnoreCase("CIRCLE") {
            return new Circle();
        } else if (figureType.equalsIgnoreCase("RECTANGLE") {
            return new Rectangle();
        } else if (figureType.equalsIgnoreCase("SQUARE") {
            return new Square();
        }
        
        return null;
    }
    
}
```

**장점**

- 클라이언트 코드로부터 서브 클래스의 인스턴스화를 제거하여 서로 간의 종속성을 낮추고, 결함도를 느슨하게 하며, 확장을 쉽게 한다.
- 클라이언트와 구현 객체들 사이에 추상화를 제공

**단점**

- 새로 생성할 객체가 늘어날 때마다, Factory 클래스에 추가해야 되기 때문에 클래스가 많아짐

출처

[https://velog.io/@lsj8367/자바-팩토리패턴](https://velog.io/@lsj8367/%EC%9E%90%EB%B0%94-%ED%8C%A9%ED%86%A0%EB%A6%AC%ED%8C%A8%ED%84%B4)
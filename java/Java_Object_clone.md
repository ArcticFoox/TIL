# Java Object.clone()

자신을 복제하여 새로운 인스턴스를 생성

```java
protected native Object clone() throws CloneNotSupportedException;
```

## Cloneable 인터페이스

자체 추상 메서드와 같은 동작이 하나도 존재하지 않는 마커 인터페이스

Cloneable이 구현되지 않는 재정의된 clone 메서드에서는 CloneNotSupportedException 예외를 발생

즉 Cloneable을 implements 함으로써 Object의 clone 메서드를 호출하지 않고 재정의된 clone 메서드가 있음을 보장

## clone() 동작

Object의 clone메서드는 필드의 모든 내용을 복사할 때 얕은 복사로 결과를 반환

이는 사용 시 여러 불편한 점과 필수적인 추가 작업이 필요함을 요구

### 얕은 복사

자바에서 일반적으로 값을 복사할 때는 대입연산자(=)를 이용

Primitive 값을 복사할 때는 상수값을 가져와 복사하기 때문에 문제가 없지만, reference 값을 복사할 때 문제

```java
int[] a = new int[2];
a[0] = 3;
a[1] = 2;
int[] b = a;
b[0] = 1;
a[1] = 5;
System.out.println(a[0]);
System.out.println(b[0]);
/*
	출력 
	1
	1
*/
```

Reference 변수가 갖고 있는 값은 해당 레퍼런스의 해시코드 값이기에,

단순 대입연산을 수행하게 되면 해시코드 값을 복사하여 같은 객체를 레퍼런싱하게 되므로 위 코드와 같이 동작

이렇게 레퍼런스의 해시코드(c 에서는 주소값)만을 복사하는 경우를 얕은 복사라고 함

```java
System.out.println(a == b);
/*
	출력
	true;
*/
```

위 두 값은 완전히 같은 레퍼런스를 가르키므로 **동일**

### 깊은복사

깊은복사를 위해서는 해당 객체에 대한 메모리를 새로 할당하고 모든 primitve 값을 새 메모리에 복사해서 넣음

```java
//다음과 같이 수행하여 배열의 복사를 구현합니다.
int[] a = new int[2];
a[0] = 3;
a[1] = 2;
int[] b = new int[2];
b[0] = a[0];
b[1] = a[1];
```

이는 객체 내에 다른 레퍼런스가 있는 경우에도 마찬가지로 수행해야하며,

그렇지 않다면 새로운 객체 내에도 얕은 복사가 된 객체가 있을 수 있음

```java
class Apple{
	int size;
	public Apple(){
		this.size = 10;
	}
}
class Tree{
	int height;
	Apple apple;
}

//...

//tree2 에 tree1 복사
Tree tree1 = new Tree();
Tree tree2 = new Tree();

tree1.apple = new Apple();
tree1.height = 40;
tree1.apple.size = 5;

//tree2에 대한 메모리는 새로 할당되었지만, apple은 얕은 복사로 같은 객체를 레퍼런싱
tree2.height = tree1.height;
tree2.apple = tree1.apple;

//따라서 아래와 같은 경우 tree1.apple.size도 변경됨
tree2.apple.size = 7;

//apple 객체도 깊은 복사가 필요
tree2.apple = new Apple();
tree2.apple.size = tree1.apple.size;
```

## Clone() 단점

### 반드시 재정의

`Object` 의 `clone` 메서드는 `protected` 단계로 접근을 제한하기 때문에 재정의하지 않은 클래스의 `clone()` 메서드를 외부에서 호출 할 수 없음

또한 내부적으로 모든 필드의 값을 복사해 새로 생성한 객체의 필드에 넣기 때문에 `clone()` 을 재정의하지 않고 사용한다면 얕은 복사만 가능

따라서 깊은 복사를 위해서는 해당 클래스에 맞게 `clone()` 을 재정의해줘야 함

- ex ) `LinkedList`의 `clone()`

```java
/*
	superClone 메서드로 상위 클래스의 clone()을 호출하고 얕은 복사 객체를
	재정의된 clone() 메서드로 목적에 맞게 값 복사
*/

private LinkedList<E> superClone() {
	try {
		return (LinkedList<E>) super.clone();
	} catch (CloneNotSupportedException e) {
		throw new InternalError(e);
	}
}

public Object clone() {
	LinkedList<E> clone = superClone();

	// Put clone into "virgin" state
	clone.first = clone.last = null;
	clone.size = 0;
	clone.modCount = 0;

	// Initialize clone with our elements
	for (Node<E> x = first; x != null; x = x.next)
		clone.add(x.item);

	return clone;
}
```

### super.clone() 체인

위 `LinkedList` 의 코드를 보고 확인할 수 있듯이 상속 관계에 있는 클래스를 복제하기 위해서,

`super.clone()` 의 체인으로 클론 메서드를 호출

`super.clone()` 의 체인으로 복제하지 않는다면,

서브 클래스에서 재정의되지 않은 속성을 포함하여 복제를 해야하므로 가독성이 낮은 코드가 됨

- 부모의 속성을 직접 복사

```java
class Fruit{
	int size;
}

private static class Apple extends Fruit implements Cloneable {
	@Override
	public Object clone() throws CloneNotSupportedException{
		Apple clone = new Apple();
//상속 관계가 여럿이거나, 부모의 속성이 많아질 경우 구성이 복잡해짐
		clone.size = this.size;
		return clone;
	}
}
```

- `super.clone()` 체인

```java
private static class Fruit {
	int value;

	@Override
	public Object clone() throws CloneNotSupportedException{
		Fruit clone = (Fruit)super.clone();
		clone.value = this.value;
		return clone;
	}
}

private static class Apple extends Fruit implements Cloneable {
	@Override
	public Object clone() throws CloneNotSupportedException{
//super.clone() 호출
		return super.clone();
	}
}
```

이는 상속 관계 중 하나의 클래스라도 얕은 복사가 된다면,

복제 후의 객체가 원본 객체의 레퍼런스를 참조할 수 있다는 위험성이 있음

### 예외 처리

clone()의 원형에서 CloneNotSupportedException을 던지기에,

모든 하위의 호출 스택에서 매번 예외처리를 해줘야 함

### 캐스팅

`super.clone()` 호출 시 부모타입의 객체를 가져오므로 반드시 다운캐스팅으로 데이터 형을 맞춰야 함

이 때 다운캐스팅은 컴파일 단계에서 오류가 확인되지 않으므로 주의해서 사용해야 함

```java
/*
	반환형이 Object 이므로 다운캐스팅 필요
	에러 발생 시 런타임에 ClassCastException
*/
Apple apple1 = new Apple();
Apple apple2 = (Apple)apple1.clone();
```

```java
/*
	혹은 clone()의 반환형을 바꾸고 clone() 내에서 캐스팅
*/
private static class Apple extends Fruit implements Cloneable {
	@Override
	public Apple clone() throws CloneNotSupportedException{
		return (Apple)super.clone();
	}
}
```

### final 멤버 제어

final로 정의된 멤버는 선언과 동시에 초기화 되거나 단 한번 수행되는 생성자나 초기화 블럭에 의해 초기화 되어야함

하지만 clone()메서드는 메서드를 통한 객체 반환이므로 생성자 및 초기화 블록을 사용할 수 없음

따라서 final 멤버는 임의로 제어할 수 없음
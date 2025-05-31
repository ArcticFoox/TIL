# Kotlin 기초

### Main

```jsx
fun main() {
// 내용
}
```
<br>

### Hello World

```jsx
fun main(){
 println("Hello, world!")
}
```
<br>


### 세미콜론

코틀린은 세미콜론이 선택 사항

일반적으로 코틀린 코드에서 각 줄은 하나의 문으로 간주

각 줄의 끝네 세미콜론을 삽입하지 않아도 됨

한 줄에 여러 개의 문이 포함된 경우, 세미콜론을 통해 구분

<br>

### 주석

자바와 동일

<br>

### 변수

var : 가변 변수, 내용 변경 가능

val : 불면 변수, 내용 변경 불가

```jsx
fun main() {
    var num1 = 5// 가변 변수 var
    num1 = 10// 변수 내용 변경 가능
    println(num1)// 출력: 10val num2 = 5// 불변 변수 val
    num2 = 10// 에러 발생! 변수 내용 변경 불가능
    println(num2)
}
```
<br>

### Data Type

```jsx
// 1, 타입지정val 식별자(변수명) : (타입) = (초기화)
// 2. 타입추론val 식별자(변수명) = (초기화)
```

타입을 지정하여 변수를 생성할 수 있고,

타입을 지정하지 않으며 초기화되는 값에 따라 타입 추론(type infernece) 진행

```jsx
fun main() {
  val integer: Int = 11           (1)
  val fractional: Double = 1.4    (2)
  val trueOrFalse: Boolean = true (3)
  val words: String = "A value"   (4)
  val character: Char = 'z'       (5)
```

정수 : Byte, Short, Int, Long

부동소수점형 : Float, Double

문자형 : Char

논리형 : Boolean

문자열 : String

**멀티라인 문자열**

```jsx
val multiLineString = """
    This is a
    multi-line
    string.
"""
```

각 줄은 개행 문자로 구분되어 저장

<br>

### 함수

```jsx
fun 함수명(param: String) : 반환타입 {
// 코드return 반환내용
}
```

변수명, 파라미터, 반환 타입을 합쳐 함수 시그니처라고 함
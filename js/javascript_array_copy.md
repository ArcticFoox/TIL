# JavaScript

### 원시 타입( Primitive Type )

Number & Bigint

String

Boolean

null

undefined

Symbol

### 객체 타입( Object / Reference Type)

Array

Function

Date

…

### 차이점

|  | 원시타입 | 객체타입 |
| --- | --- | --- |
| 값의 종류 | 변경 불가능 한 값 | 변경 가능한 값 |
| 저장 내용 | 실제 값 저장  | 참조 값(메모리 주소) 저장 |
| 전달 방식 | 값에 의한 전달 | 참조에 의한 전달 |

원시값을 갖는 변수를 다른 변수에 할당하면 원본의 원시 값이 복사되어 전달(pass by value)

객체를 가리키는 변수를 다른 변수에 할당하면 원본의 참조 값이 복사되어 전달(pass by reference)

### 배열 값 복사하는 방법

```jsx
let arrA = [1, 2, 3];
let arrB = arrA;

arrB[0] = 5;

console.log(arrA); // [5, 2, 3]
console.log(arrB); // [5, 2, 3]
```

위에서 알아본 듯이 기본적으로 참조 값을 복사해 버리기 때문에 서로 영향을 받는다.

이것을 참조값 복사(얕은 복사)라고 하며 이를 통해 메모리를 효율적으로 사용하고 객체 타입을 복사해 생성하는 비용을 줄일 수 있다.

그렇다면 pass by value를 위해선 어떻게 사용해야 할까?

### slice()

```jsx
let arr = [1, 2, 3];
let copy = arr.slice();

copy[0] = 10;

console.log(arrA); // [1, 2, 3]
console.log(arrB); // [10, 2, 3]
```

MDN 문서에 ‘slice 메서드는 어떤 배열의 begin부터 end까지에 대해 얕은 복사본을 새로운 배열 객체로 반환’으로 기록되어 있음

### spread 연산자

```jsx
let arr = [1, 2, 3];
let copy = [...arrA];
 
copy[0] = 10;
 
console.log(arr); // [1, 2, 3]
console.log(copy); // [10, 2, 3]
```

### concat()

```jsx
let arr = [1, 2, 3];
let copy = arr.concat();
 
copy[0] = 10;
 
console.log(arr); // [1, 2, 3]
console.log(copy); // [10, 2, 3]
```

### map(), filter(), reduce(), from()

위 모든 함수는 기존의 배열로 얕은 복사후 그것을 새로운 배열 객체로 변환

### JSON.parse() & JSON.stringify()

다차원 배열을 정상적으로 복사

JSON.stringify() 함수로 원본 배열을 문자열로 변환 후 JSON.parse() 함수로 JavaScript Object로 파싱

```jsx
let arr = [[1, 2, 3,], ['A', 'B', 'C']];
let copy = JSON.parse(JSON.stringify(arrA));
 
copy[0][0] = 10;
 
console.log(arr); // [[1, 2, 3], ['A', 'B', 'C']]
console.log(copy); // [[10, 2, 3], ['A', 'B', 'C']]

```
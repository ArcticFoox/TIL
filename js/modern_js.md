# Modern JavaScript

2015년에 나온 ECMAScript 2015(6판)와 그 이후의 판을 구현한 자바스크립트

class, 모듈, 제너레이터, 화살표 함수, let & const 키워드 등의 굵직한 문법들은 ES6에서 처음 정의

![image.png](/js/img/image.png)

## 명시적 타입변환(Explicit Coercion) vs 암묵적 타입변환(implicit Coercion)

### 명시적 타입변환 == 타입 캐스팅(type casting)

- 개발자가 의도적으로 값의 타입을 변환
- 표준 빌트인(built-in) 생성자 함수 사용 → String(), Number(), Boolean() 을 new 키워드 없이 호출

### 암묵적 타입변환 == 타입 강제 변환(type coercion)

- 자바스크립트 엔진에 의해 암묵적으로 타입이 변환
- 원시 타입(primitive type) 중 하나로 타입을 자동 변환됨 → string, number, boolean

### 주의점

- 원시 값은 변경 불가능한 값(immutable value) → 타입 변환이 기존 원시 값을 직접 변경하는 것은 아님(기존 원시값을 사용해 다른 타입의 새로운 원시 값을 생성)
- 명시적 타입변환은 다른 사람이 코드를 이해하는데 좀 더 직관적이나 JS 문법을 잘 아는 사람과 협업할 땐 암시적 타입변환이 효율적일 수 있음

## 문자열 타입 변환

### 명시적 타입변환(explicit coercion)

```jsx
// 1) String 생성자 함수를 new 연산자 없이 호출하는 방법
// 숫자 타입 => 문자열 타입
String(1);        // -> "1"
String(NaN);      // -> "NaN"
String(Infinity); // -> "Infinity"
// 불리언 타입 => 문자열 타입
String(true);     // -> "true"

// 2) Object.prototype.toString 메서드를 사용하는 방법
// 숫자 타입 => 문자열 타입
(1).toString();        // -> "1"
(NaN).toString();      // -> "NaN"
(Infinity).toString(); // -> "Infinity"
// 불리언 타입 => 문자열 타입
(true).toString();     // -> "true"

// 3) 문자열 연결 연산자를 이용하는 방법
// 숫자 타입 => 문자열 타입
1 + '';        // -> "1"
NaN + '';      // -> "NaN"
Infinity + ''; // -> "Infinity"
// 불리언 타입 => 문자열 타입
true + '';     // -> "true"
```

### 암묵적 타입변환(implicit coercion)

```jsx
// 숫자 타입
1 + ''         // -> "1"
NaN + ''       // -> "NaN"
Infinity + ''  // -> "Infinity"
-Infinity + '' // -> "-Infinity"

// 불리언 타입
true + ''  // -> "true"
false + '' // -> "false"

// null 타입
null + '' // -> "null"

// undefined 타입
undefined + '' // -> "undefined"

// 심벌 타입
(Symbol()) + '' // -> TypeError: Cannot convert a Symbol value to a string

// 객체 타입
({}) + ''           // -> "[object Object]"
Math + ''           // -> "[object Math]"
[] + ''             // -> ""
[10, 20] + ''       // -> "10,20"
(function(){}) + '' // -> "function(){}"
Array + ''          // -> "function Array() { [native code] }"
```

## 숫자 타입 변환

### 명시적 타입변환(explicit coercion)

```jsx
// 1. Number 생성자 함수를 new 연산자 없이 호출하는 방법
// 문자열 타입 => 숫자 타입
Number('0');     // -> 0
Number('-1');    // -> -1
Number('10.53'); // -> 10.53
// 불리언 타입 => 숫자 타입
Number(true);    // -> 1
Number(false);   // -> 0

// 2. parseInt, parseFloat 함수를 사용하는 방법(문자열만 변환 가능)
// 문자열 타입 => 숫자 타입
parseInt('0');       // -> 0
parseInt('-1');      // -> -1
parseFloat('10.53'); // -> 10.53

// 3. + 단항 산술 연산자를 이용하는 방법
// 문자열 타입 => 숫자 타입
+'0';     // -> 0
+'-1';    // -> -1
+'10.53'; // -> 10.53
// 불리언 타입 => 숫자 타입
+true;    // -> 1
+false;   // -> 0

// 4. * 산술 연산자를 이용하는 방법
// 문자열 타입 => 숫자 타입
'0' * 1;     // -> 0
'-1' * 1;    // -> -1
'10.53' * 1; // -> 10.53
// 불리언 타입 => 숫자 타입
true * 1;    // -> 1
false * 1;   // -> 0
```

### 암묵적 타입변환(implicit coercion)

```jsx
// 문자열 타입
+''       // -> 0
+'0'      // -> 0
+'1'      // -> 1
+'string' // -> NaN

// 불리언 타입
+true     // -> 1
+false    // -> 0

// null 타입
+null     // -> 0

// undefined 타입
+undefined // -> NaN

// 심벌 타입
+Symbol() // -> ypeError: Cannot convert a Symbol value to a number

// 객체 타입
+{}             // -> NaN
+[]             // -> 0
+[10, 20]       // -> NaN
+(function(){}) // -> NaN
```

## 불리언 타입 변환

### 명시적 타입변환(explicit coercion)

```jsx
// 1. Boolean 생성자 함수를 new 연산자 없이 호출하는 방법
// 문자열 타입 => 불리언 타입
Boolean('x');       // -> true
Boolean('');        // -> false
Boolean('false');   // -> true
// 숫자 타입 => 불리언 타입
Boolean(0);         // -> false
Boolean(1);         // -> true
Boolean(NaN);       // -> false
Boolean(Infinity);  // -> true
// null 타입 => 불리언 타입
Boolean(null);      // -> false
// undefined 타입 => 불리언 타입
Boolean(undefined); // -> false
// 객체 타입 => 불리언 타입
Boolean({});        // -> true
Boolean([]);        // -> true

// 2. ! 부정 논리 연산자를 두번 사용하는 방법
// 문자열 타입 => 불리언 타입
!!'x';       // -> true
!!'';        // -> false
!!'false';   // -> true
// 숫자 타입 => 불리언 타입
!!0;         // -> false
!!1;         // -> true
!!NaN;       // -> false
!!Infinity;  // -> true
// null 타입 => 불리언 타입
!!null;      // -> false
// undefined 타입 => 불리언 타입
!!undefined; // -> false
// 객체 타입 => 불리언 타입
!!{};        // -> true
!![];        // -> true
```

### 암묵적 타입변환(implicit coercion)

```jsx
// 전달받은 인수가 Falsy 값이면 true, Truthy 값이면 false
function isFalsy(v) {
  return !v;
}

// 전달받은 인수가 Truthy 값이면 true, Falsy 값이면 false
function isTruthy(v) {
  return !!v;
}

// 모두 true를 반환
isFalsy(false);
isFalsy(undefined);
isFalsy(null);
isFalsy(0);
isFalsy(NaN);
isFalsy('');

// 모두 true를 반환
isTruthy(true);
isTruthy('0'); // 빈 문자열이 아닌 문자열은 Truthy
isTruthy({});
isTruthy([]);
```

## 단축 평가(short-circuit evaluation)

### 논리 연산자를 사용한 단축 평가

- 논리합, 논리곱 연산자

```jsx
// 논리합(||) 연산자
'Cat' || 'Dog'  // -> "Cat"
false || 'Dog'  // -> "Dog"
'Cat' || false  // -> "Cat"

// 논리곱(&&) 연산자
'Cat' && 'Dog'  // -> "Dog"
false && 'Dog'  // -> false
'Cat' && false  // -> false
```

- if문 대체 가능

```jsx
if (done) message = '완료';

// 위와 같은 결과
message = done && '완료';
```

- 삼항연산자

```jsx
message = done ? '완료' : '미완료';

// if...else 문 대체 가능
if (done) message = '완료';
else      message = '미완료';
```

- 객체 원소 사용 전에 null 또는 undefined 확인 가능

```jsx
var elem = null;
var value = elem.value; // TypeError: Cannot read property 'value' of null
// =>
var value = elem && elem.value; // -> null
```

- 매개변수 기본 값 설정

```jsx
// 단축 평가를 사용한 매개변수의 기본값 설정
function getStringLength(str) {
  str = str || '';
  return str.length;
}

getStringLength();     // -> 0
getStringLength('hi'); // -> 2

// 다른 방법(ES6의 매개변수의 기본값 설정)
function getStringLength(str = '') {
  return str.length;
}

getStringLength();     // -> 0
getStringLength('hi'); // -> 2
```

### 옵셔널 체이닝(optional chaining) 연산자

- `.?` 연산자 이용해서 객체 원소가 null 또는 undefined가 아닌지 확인 가능좌항의 피연산자가 null 또는 undefined → undefined그외 -> 우항 property

```jsx
var elem = null;
var value = elem && elem.value; // null
```

- 논리곱(&&) vs 옵셔널 체이닝

```jsx
// 논리곱
var str = ""; //
var length = str && str.length; // ''
// 좌항 피연산자가 Falsy -> 좌항 피연산자를 그대로 반환

// 옵셔널 체이닝
var str = "";
var length = str?.length; // 0
// 좌항 피연산자가 Falsy값이라도 null 또는 undefined만 아니면, 우항의 프로퍼티를 참조
```

### null 병합(nullish coalescing) 연산자

- `??` 연산자 이용해서 변수에 기본값 설정할 때 활용 가능좌항 피연산자가 null 또는 undefined -> 우항의 피연산자 반환그외 -> 좌항의 피연산자 반환

```jsx
var foo = null ?? "my string"; // "default string"
```

- 논리합(||) vs null 병합 연산자

```jsx
// 논리합
var foo = "" || "my string"; // "default string"
// 좌항의 피연산자가 Falsy -> 우항의 피연산자를 반환
// => 의도적으로 0 이나 ''을 기본값으로 사용한다면 예기치 않은 동작이 발생

// null 병합 연산자
var foo = "" ?? "default string"; // ''
// 좌항 피연산자가 Falsy값이라도  null 또는 undefined가 아니면 , 좌항 피연산자 반환
```
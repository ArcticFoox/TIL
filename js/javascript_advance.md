# JS Advance

## Truthy & Falsy

참, 거짓이 아닌 값도 참, 거짓으로 평가

```jsx
if (123) {
	console.log("123");
} else {
	console.log("error");
} // 123

if (undefined) {
	console.log("true");
} else {
	console.log("undefined");
} // undefined
```

### Falsy value

```jsx
let f1 = undefined;
let f2 = null;
let f3 = 0;
let f4 = -0;
let f5 = NaN;
let f6 = "";
let f7 = 0n;
```

## 단락 평가(short-circuit Evaluation)

```jsx
let varA = false;
let varB = true;

// AND 연산
console.log(varA && varB);

// OR 연산
console.log(varB || varA);
```

첫 번째 피연산자의 값만으로도 어떠한 값이 확정되면 두 번째 값에는 접근하지 않음

## 구조 분해 할당

배열이나 객체에 저장된 여러 개의 값을 분할해서 각각 다른 변수에 할당하는 문법

```jsx
// 배열
let arr = [1, 2, 3];

let [one, two, three, four] = arr;
console.log(one, two, three, four);
// 1, 2, 3, undefined

let [one, two, three, four = 4] = arr;
console.log(one, two, three, four);
// 1, 2, 3, 4
```

## Spread 연산자 & Reest 매개변수

### Spread

```jsx
let arr1 = [1, 2, 3];
let arr2 = [4, ...arr1, 5, 6];

console.log(arr2);
//콘솔 : [4, 1, 2, 3, 5, 6]

let obj1 = {
	a: 1,
	b: 2,
}

let obj2 = {
	...obj1,
	c: 3,
	d: 4,
}

console.log(obj2);
// 콘솔 : {a: 1, b: 2, c: 3, d:4}

function funcA(p1, p2, p3){
	console.log(p1, p2, p3);
}

funcA(...arr1);
//콘솔 : 1 2 3
```

### Rest

```jsx
function funcB(...rest){
	console.log(rest);
}

funcB(...arr1);
// 콘솔 : [1, 2, 3]

function funcB(one, ...rest){
	console.log(rest);
}
// 콘솔 : [2, 3];
```

## Promise

비동기 작업을 효율적으로 처리할 수 있도록 도와주는 자바스크립트의 내장 객체

resovle와 reject에 인수로 결과값을 전달할 수 있음

```jsx
const promise = new Promise((resolve, reject) => {
//비동기 작업을 실행하는 함수 : executor

	setTimeout(() => {
		console.log("안녕");
			reject("왜 실패했는지 이유...");
		}, 2000);
});

setTimeout(()=> {
	console.log(promise);
}, 3000);
```

promise가 관리하는 비동기 작업이 성공하거나 실패했을 때의 결과값을 이용할 수 있음

```jsx
const promise = new Promise((resolve, reject) => {
//비동기 작업을 실행하는 함수 : executor

	setTimeout(() => {
	const num = 10;
	
		if (typeof num === "number"){
			resolve(num + 10);
		} else {
			reject ("num이 숫자가 아닙니다");
		}
	}, 2000);
});

//promise가 성공하게 되면 then메서드에 전달한 callback 함수를 실행함 
promise.then((value) => {
	console.log(value);
});
//콘솔 : 20 

// promise가 실패했을때 콜백함수 실행
promise.catch((error) => {
	console.log(error);
});

// then과 catch는 promise 객체를 반환하기 때문에 다음과 같이 사용 가능
promise.then((value) => {
	console.log(value);
}).catch((error) => {
	console.log(error);
});
```
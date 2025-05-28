# Vuex

## One-Way Data Binding in Vue

- Vue.js에서 부모-자식 컴포넌트 사이에 단방향 바인딩을 형성
- 부모는 props를 통해 자식에게 데이터를 전달하고, 자식은 events(emit)를 통해 부모에게 전달
    - props는 아래로, events 위로 전달
- 하위 컴포넌트가 실수로 상위 컴포넌트의 상태를 변경하여 앱의 데이터 흐름을 추론하기 더 어렵게 만드는 것을 방지
- 컴포넌트의 개수가 많아지면 컴포넌트 간에 데이터 전달이 어려워짐
    
    만약, 한 컴포넌트에서 데이터를 수정할 때마다 prop, emit을 사용해 전달해야하는데 과정이 복잡해질 수 있음
    
    - 해결 방법으로 Event Bus와 Vuex 등이 있음
- ‘단방향 흐름’ 컨셉의 간단히 묘사한 그림( Vue 인스턴스 내 상태 관리 )
    - state : 컴포넌트 간에 공유할 data
    - view : 데이터가 표현될 template
    - actions : 사용자의 입력에 따라 반응할 methods

## 상태 관리 ( State Management )

### 상태

- 어떤 객체가 가진 데이터
    - 객체 간 데이터를 주고 받으며 application이 구현
    - 예를 들어, Vue에서는 컴포넌트(객체)에 data가 상태라 할 수 있음
- 지역적 또는 전역적 상태를 적절히 구분해서 설계해야 함
    - 지역적으로 props와 emit으로 컴포넌트 내 데이터를 전달
    - 전역적으로 vuex 스토어에 저장하던지, 브라우저의 쿠키(Cookie) 또는 로컬 스토리지 (localStorage)에 저장하여 데이터 공유 가능

### 특징

### Single Source of Truth

- 하나의 어플리케이션은 하나의 store만 가진다는 것을 의미
    - 하나의 객체가 어플리케이션의 전체 state를 포함하고, 이 객체가 하나의 원본 소스 임
- 단일 상태 트리는 특정 state를 바로 찾을 수 있게 만들고, 현재 앱의 state의 스냅샷을 가져올 수 있어 디버깅을 간편하게 도움

### state is Reactive

- 상태가 변경되면 이 상태를 공유하는 다른 곳에서도 상태가 업데이트 됨

### state management pattern + library

- Vuex는 상태 관리 패턴이자 라이브러리
- 여러 컴포넌트 간의 데이터 전달과 이벤트 통신을 한곳에서 관리하는 패턴을 Vuex를 통해 구현 가능
- 상태 관리 및 특정 규칙 적용과 관련된 개념을 정의하고 분리함으로써, 코드의 구조와 유지 관리 기능 향상

### 주의 사항

- 페이지 새로고침시 store에 데이터가 사라짐이 문제는 브라우저 쿠키 or 스토리지나 다른 라이브러리 등으로 처리 가능
    - vuex-persistedstate 라이브러리
    - 모든 store 값들을 localstorage로 저장할 시 속도 이슈가 발생할 수 있음
- 공통적으로 사용하는 상태가 아닌 경우는 메모리를 낭비할 수 있음

## Vuex 구조(Core Concept)

### state

- Vue의 data와 같음
- 원본 소스의 역할을 하며, View와 직접적으로 연결되어있는 Model
- **state는 mutation을 통해서만 변경이 가능**
    - mutation을 통해 state가 변경이 일어나면 반응적으로 View가 업데이트

### mutations

- state를 변경하는 유일한 방법
- 일반적으로 commit을 통해서만 호출 할 수 있음
    - Helper 함수로 직접 호출 가능
- 첫 번째 인자는 state, 두 번째 인자는 payload

```jsx
store.commit({
  type: 'increment',
  amount: 10
})
/*-----------------------------*/
// store.js
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

- 주로 API를 통해 전달받은 데이터를 mutations에서 가공하여 state를 설정하는데 사용됨

### actions

- 비동기 작업이 가능
    - 그 외에는 mutation과 비슷
- mutation을 호출하기 위한 commit이 가능
    - action에서도 mutation을 통해 state를 변경 가능
- dispatch를 통해 호출할 수 있음
- 첫 번째 인자는 context, 두 번째 인자는 payload
    - context는 state, commit, dispatch, rootstate 속성들을 포함

```jsx
store.dispatch('incrementAsync', {
  amount: 10
})
// or
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
/*-----------------------------*/
// store.js
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

- 주로 axios를 통한 api 호출과 그 결과에 대해 return을 하거나 mutation으로 commit하는 용도로 사용

### getters

- Vue의 computed와 같음 (계산된 속성)
    - getter의 결과는 종속성에 따라 캐시 되고 변경된 경우에만 다시 계산
- state에 대해 연산을 하고 그 결과를 view에 바인딩 할 수 있음
- state의 변경 여부에 따라 다시 계산하고 view를 업데이트

### modules

- 모듈별(기능 또는 페이지)로 store를 분리하고 관리 가능
- 실무에서는 단 하나의 store를 사용하기 힘듦
- [모듈화](https://velog.io/@yjyoo/vue.js-Vuex-%EC%A0%95%EB%A6%AC#store-%EB%AA%A8%EB%93%88%ED%99%94)에서 추가 내용 참고
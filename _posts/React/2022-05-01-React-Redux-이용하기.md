---
title:  "React Redux 이용하기" 
excerpt: ""

categories:
  - React
tags:
  - []

toc: true
toc_sticky: true
 
date: 2022-05-01
last_modified_at: 2022-05-01

---

# 사용하는 이유

1. state 종속성

    react는 컴포넌트 내부에 state를 만들고 함수를 이용해 state 값을 변경하며 사용합니다. 따라서 state는 컴포넌트에 종속됩니다. 하지만 redux를 이용하면 컴포넌트 바깥에서 state를 관리하기 때문에 바깥의 state를 변경하면 해당 state를 사용하는 컴포넌트는 리랜더링 되도록 구현이 가능합니다.

2. props 반복 문제

    예를 들어 컴포넌트 a가 b라는 자식 컴포넌트를 가지고 있고, b는 c라는 자식 컴포넌트가 있다고 가정을 합니다. 이때 c라는 컴포넌트가 a에 있는 state를 참조하기 위해서는 a가 b라는 컴포넌트에 props를 이용해 값을 전달해 주고, b는 c에 props로 전달해서 참조해야 합니다. 수정할 때 또한, 함수를 똑같이 props를 반복해 사용하며 전달해 줘야 합니다. 이 경우 계속 props를 전달해 줘야 하기 때문에 가독성이 떨어지고 개발자가 실수할 확률이 높아집니다. 대신 redux를 이용할 경우 어디에 있는 컴포넌트인지 상관없이 단 한 번 만에 state를 참조할 수 있습니다.


# Redux의 기본 작업 순서

컴포넌트에서 Dispatch라는 함수를 통해 action을 전달해 주면 reducer에 정의된 로직에 따라 store의 state가 변하고 해당 state를 쓰는 컴포넌트가 다시 렌더링 하는 순서로 동작합니다.

# 설치

    npm install --save redux react-redux

# Store 만들기

우선 첫 번째로 store를 만들어 컴포넌트 외부에서 state를 관리할 수 있도록 해보겠습니다.

store.js를 만들고 다음과 같이 작성합니다.

    import {createStore] from 'redux';
    
    export default createStore(function(state, action) {
        if (state === undefined) { // 최초로 실행할때 state를 정의
            return {
                number: 0
            }
        }
    
        // action에 따라 state를 변경하도록 정의
        if(action.type === "Action1") {
            return {...state, number: action.num});
        }
    
    
    },  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()); // Dev Tools를 사용하기 위한 인자

# Dispatch 하기

dispatch는 store에 있는 상태를 바꾸기 위한 기능입니다. 우선 store를 import 합니다. 그리고 store.dispatch를 이용해 action과 인자를 넘겨주도록 하면 됩니다. 이때 위에서 action.type에 따라 action을 구분하기 때문에 type에 해당 action 이름을 일치하게 작성하면 됩니다.

    import store from '../store';
    ...
    store.dispatch({type: "Action1", num: newValue})

# State 값 가져오기

Store에서 데이터를 가져오기 위해서는 다음과 같이 getState()를 이용해 가져오면 됩니다.

하지만 지금 상태로는 store의 state가 변경되더라도 해당 컴포넌트는 알 수 없어 리랜더링이 되지는 않습니다.

    import store from '../store';
    ...
    store.getState().number; // state 값 가져오기

# Subscribe

Subscribe를 하는 이유는 위에서 State 값을 가져올 때의 문제점인 store의 state가 변경되더라도 해당 컴포넌트는 알 수 없다는 문제를 해결하기 위해서입니다. Subscribe를 하면 store는 state가 변경될 때마다 값이 변경되었다고 알려줍니다. 이때 해당 컴포넌트는 리랜더링이 진행되도록 구현이 가능합니다. 사용법은 아래와 같습니다.

    import store from '../store';
    
    ...
    constructor(props) {
        super(props);
        this.state = {
            number : 0
        }
    
        // store의 state값이 변경될때마다 해당 function이 호출됩니다.
        store.subscribe(function() {
            let num = store.getState().number;
            
            this.state.setState({number: num});  // setState로 설정해서 render()가 호출되도록 구현해야합니다.
        }.bind(this));
    }

# 컴포넌트에서 Redux 종속성 분리

지금까지 정리한 내용만 이용해도 Redux는 충분히 동작합니다. 하지만 지금 상태로는 컴포넌트의 재사용성이 매우 떨어집니다. 해당 컴포넌트를 다른 곳에서 사용하기 위해서는 해당 컴포넌트에서 redux를 사용한 부분을 전부 찾아내 다른 곳에서 사용할 수 있도록 수정해야 하는 문제점이 있습니다. 이는 개발자의 실수로 이어지기 쉽습니다. 그래서 Container 컴포넌트를 만들어 Redux에 대한 기능을 모두 Container를 이용해 처리하고 기존 컴포넌트는 핵심 기능을 처리하도록 구현하면 다른 곳에서 사용할 경우 Container만 수정하면 되기 때문에 생산성이 올라갑니다.

###  Container 컴포넌트

    import MyComponent from "../Component/MyComponent";
    import React, {Component} from 'react';
    import store from '../store';
    
    // Redux 처리는 여기서 진행
    export default class extends Component {
        constructor(props) {
            super(props);
            this.state = {
                number : 0
            }
        
            // store의 state값이 변경될때마다 해당 function이 호출됩니다.
            store.subscribe(function() {
                let num = store.getState().number;
            
                this.state.setState({number: num});  // setState로 설정해서 render()가 호출되도록 구현해야합니다.
            }.bind(this));
        }
        
        render() {
            return (
                <MyComponent 
                    number={this.state.number}
                    // props를 이용해 해당 기능을 전달합니다.
                    onClick={function(num) {
                        // store에 dispatch하는 기능
                        store.dispatch({type:"Action1", num: num});
                    }.bind(this)
                />
            );
        }
    }

### 컴포넌트

    import React, {Component} from 'react';
    
    class MyComponent extends Component {
        render() {
            return (
                <div>
                    ...
                    <input type="button" value="+" onClick={
                        function(){
                            this.props.onClick(this.props.number + 1);  // props의 onClick을 호출하고 값을 전달
                        }.bind(this)
                    }> </input>
                </div>
            );
        }
    }
    
    export default AddNumber;

# React-Redux를 이용해 Container를 편하게 만들기

이번에는 react-redux를 이용해 container를 훨씬 편하게 만들어보겠습니다.

우선 index.js에 provider를 추가해 줘서 각각 파일에 store를 import 하지 않고 모든 부분에서 store를 이용할 수 있도록 해주겠습니다.

    ...
    import {Provider} from 'react-redux';
    import store from './store';
    ...
    
    ReactDOM.render(
        <Provider store={store}>
            <App />
        </Provider>
        , document.getElementById('root');
    )
    ...

react-redux를 사용하는 기본 구조는 다음과 같습니다.

react-redux에서 connect를 가져와서 state를 가져오는 함수와 dispatch 하는 함수를 만들고 컴포넌트와 connect 시켜주면 위에서 Container 컴포넌트를 따로 만드는 작업을 대신 진행해 줍니다.

    import MyComponent from "../components/MyComponent';
    import {connect} from 'react-redux';
    
    // state를 가져오는 함수
    function mapReduxStateToReactProps(state) {
        return {
            number: state.number
        }
    }
    
    // state를 dispatch하는 함수
    function mapReduxDispatchToReactProps() {
        return {
            onClick: function(number) {
                dispatch({type: "Action1", num:number});
            }
        }
    }
    
    export default connect(mapReduxStateToReactProps, mapReduxDispatchToReactProps) (MyComponent);

이때 connect는 getState와 dispatch를 해주는 함수의 유무에 따라 다음과 같이 4가지 유형으로 connect가 가능합니다.

    // 둘다 없는 경우
    export default connect() (MyComponent);
    // State를 가져오기만 하는 경우
    export default connect(mapReduxStateToReactProps) (MyComponent);
    // dispatch만 하는 경우
    export default connect(null, mapReduxDispatchToReactProps) (MyComponent);
    // 둘다 있는 경우
    export default connect(mapReduxStateToReactProps, mapReduxDispatchToReactProps) (MyComponent);

이제 컴포넌트는 다음과 같이 사용이 가능합니다

    import React, {Component} from 'react';
    
    class MyComponent extends Component {
        render() {
            return (
                <div>
                    ...
                    <input type="button" value="+" onClick={
                        function(){
                            // props를 이용해 Container에서 전달해주는 state값과 dispatch 함수 이용 가능
                            this.props.onClick(this.props.number + 1);
                        }.bind(this)
                    }> </input>
                </div>
            );
        }
    }
    
    export default AddNumber;
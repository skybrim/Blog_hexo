---
title: react-useReducer-useContext
comments: true
date: 2020-09-23 16:19:44
tags: [web, react]
---

react 通过使用 useReducer和useContext，实现类似 redux 的效果
<!--more-->

## 核心思想：

利用 useContext，将 state 和 dispatch 升格共享出去

利用 useMemo 和 useCallback 优化

## 使用时机

1. 如果页面的 state 很简单，直接使用 useState
2. 如果页面的 state 比较复杂，使用 useReducer
3. 如果页面的层级比较深，并且需要子组件触发 state 的变化，使用 useReducer + useContext

## 代码

```js
// 初始化 state
const initState = {
    name: '',
    pwd: '',
    isLoading: false,
    error: '',
    isLoggedIn: false,
};
```

```js
// 定义 reducer 函数
const loginReducer = (state, action) => {
    switch (action.type) {
        case 'changeUserName':
            return {
                ...state,
                name: action.value,
            }
        case 'changePassword':
            return {
                ...state,
                pwd: action.value,
            }
        case 'login':
            return {
                ...state,
                isLoading: true,
                error: '',
            };
        case 'success':
            return {
                ...state,
                isLoading: false,
                isLoggedIn: true,
            };
        case 'error':
            return {
                ...state,
                error: action.payload.error,
                name: '',
                pwd: '',
                isLoading: false,
            };
        default:
            return state;
    }
}
```

```js
// 定义 context
const LoginContext = React.createContext();

// component
const LoginPage = () => {
    const [state, dispatch] = useReducer(loginReducer, initState);
    // 背景色，改变背景色，不会导致子组件重新渲染
    const [color, setColor] = useState('blue');
    const changeUserName = (e) => {
        dispatch({
            type: 'changeUserName',
            value: e.target.value,
        })
    }
    const changePassword = (e) => {
        dispatch({
            type: 'changePassword',
            value: e.target.value,
        })
    }
    return (
        <LoginContext.Provider value={{state, dispatch}}>
            <div className='background' style={{backgroundColor:color}}>
                <input placeholder='username' value={state.name} onChange={changeUserName}/>
                <br/>
                <input placeholder='password' value={state.pwd} onChange={changePassword}/>
                <LoginButton />
                <button onClick={()=>{setColor(color === 'blue' ? 'red':'blue')}}>Change Color</button>
            </div>
        </LoginContext.Provider>
    );
};
```

```js
// 子组件
const LoginButton = () => {
    // 获取 共享出来的 state 和 dispatch
    const {state, dispatch} = useContext(LoginContext)

    // useCallback
    const login = useCallback(() => {
        dispatch({type: 'login'});
        console.log('======', state.name, state.pwd);
        //发起请求
        // Axios.post()
        //     .then(() => {
        //         dispatch({type: 'success'});
        //     })
        //     .catch((error) => {
        //         dispatch({
        //             type: 'error',
        //             payload: {error: error.message}
        //         });
        //     });
    }, [state.name, state.pwd, dispatch]);

    // useMemo
    return useMemo(() => {
        console.log('LoginButton is rendering.')
        return (
            <div>
                <button onClick={login}>登录</button>
            </div>
        )
    }, [login]);
}
```


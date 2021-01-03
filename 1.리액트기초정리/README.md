### 리액트 기초정리

#### UI데이터 : 상태값과 속성값
- 리액트는 UI데이터가 변경되면 리액트 컴포넌트 함수를 이용해 화면을 자동으로 갱신한다.
- 리액트에서 UI데이터는 상태값과 속성값으로 구성된다.
- 상태값(state) : 컴포넌트 내부에서 관리되는 값
- 속성값(props) : 부모컴포넌트로부터 전달받는 값
```
//컴포넌트에 상태값을 추가하기 위해 useState를 사용한다.
import React, { useState } from "react";

function MyComponent(){
    const [color,setColor] = useState("red");
    //구조분해할당 문법을 통해서 배열 첫번쨰 원소에는 상태값,
    //두번째 원소에는 상태값을 변경하는 함수를 반환한다.

    function onClick(){
        setColor("blue");
    }
    //상태값 변경함수를 통해 상태값을 blue로 변경하는 함수이다.

    return (
        <button style={{ backgroundColor:color }} onClick={onClick}>
            버튼
        </button>
    );
    //클릭시 onClick함수가 실행되면서 상태값이 blue로 변경된다.
}
```

```
function Title(props){
    return <p>{props.title}</p>;
} 
//props매개변수를 통해 부모컴포넌트로 부터 속성값을 전달받을수 있다.

export default React.memo(Title);
//React.memo를 사용하면 전달받는 속성값이 변경될때만 리렌더링되게 할수있다.
```
```
function Todo(){
    const [count, setCount] = useState(0);
    function onClick(){
        setCount(count+1);
    }
    return (
        <div>
            <Title title={`현재 카운트 : ${count}`} />
            //key와 ref이외에는 props로 자식컴포넌트로 속성값을 전달한다.
            <button onClick={onClick}>증가</button>
            //onClick시 count가 1증가한다.
        </div>
    )
}
```

- 상태값은 같은 컴포넌트라도 사용된 각 컴포넌트별로 독립적으로 존재한다.
```
function App(){
    return (
        <div>
            <MyComponent />
            <MyComponent />
        </div>
    )
}
//Mycomponent 두개의 상태값은 따로 관리된다.
```

- 속성값은 불변변수이나, 상태값은 불변변수가 아니다. 하지만 불변변수처럼 관리하는것이 좋다.
```
function MyComponent(){
    const [count,setCount] = useState({value:0});

    function onClick(){
        count.value = 2;
        setCount(count);
    }
    //다음과 같이하면 해당 컴포넌트는 리렌더링되지않는다.
    //count(상태값)가 참조하는 객체가 동일하기때문이다.
    ...
}
```

- 컴포넌트 함수는 컴포넌트,HTML태그,문자열,숫자,배열,boolean,null등을 반환할수 있다.
```
return <MyComponent title="hello" />; //컴포넌트 반환
return <p>hello</p>; //태그 반환
return '안녕'; //문자열반환
return 123; //숫자반환
return [<p key="a">a</p>,<p key="b">b</p>]; //배열 반환
return (
    <React.Fragment>
        <p>hi</p>
        <p>aa</p>
    </React.Fragment>
); //프레그먼트 반환 (배열 대용)
return (
    <>
        <p>hi</p>
        <p>aa</p>
    </>
);//프레그먼트 축약형
return null;
return false; //null과 false는 아무것도 렌더링하지 않을때 쓴다.
return ReactDOM.createPortal(<p>hello</p>,domNode); //포탈을 사용하면 원하는 돔에 렌더링할 수 있다.
```

#### 리액트 요소와 가상 돔
- JSX문법으로 작성된 코드는 리액트의 createElement코드로 변경된다.
```
const element = <a href="https://github.com/">깃헙</a>; //JSX
//위의 코드는 바벨에 의해 밑 코드로 변환된다.
const element = React.createElement(
    'a',
    { href: 'https://github.com'},
    '깃헙',
);
//element는 createElement함수가 반환한 리액트요소이다.
//element를 출력해보면 다음과같다.
/*
{
    type:'a',   //type속성이 문자열이면 HTML태그를 나타낸다.
    key:'key1', //key속성 지정시 여기에 값이 설정된다.
    ref:null, //ref속성 지정시 여기에 값이 설정된다.
    props:{ //나머지속성들은 전부 여기에 지정
        href:'https://github.com',
        style:{
            width:100,
        },
        chilren: '깃헙',//태그사이의 값이 들어온다.
    },...
}
*/
```
- 하나의 화면구성을 위해 여러개의 요소가 트리형태로 구성된다.
- 리액트는 렌더링을 할떄마다 이전의 가상돔과 현재의 가상돔을 비교하여 변경된 부분만 실제돔에 반영하여 렌더링한다.(변경된 부분만 리렌더링)
```
function Todo({title,desc}){
    const [priority,setPriority] = useState("high");
    function onClick(){
        setPriority(priority==="high"?"low":"high");
    }
    return (
        <div>
            <Title title={title} />
            <p>{desc}</p>
            <p>{priority === "high"?"우선순위 높음":"우선순위 낮음"}</p>
            <button onClick={onClick}>우선순위 변경</button>
        </div>
    )
}

const Title = React.memo(({title}) => {
    return <p style={{ color : "blue" }}>{title}</p>;
})

ReactDOM.render(
    <Todo title="리액트 공부하기" desc="실전 리액트" />,
    document.getElementById("root"),
);

/*
트리구조의 변화

{
    type:Todo,
    props:{
        title:'리액트 공부하기',
        desc:'실전 리액트',
    },
    ...
}
* Todo는 컴포넌트의 이름이다. 실제 돔에 적용하기위해서는 모든 트리요소의 type이 문자열이 되어야한다.
그러기위해서는 컴포넌트 함수가 호출되어야한다.


{
    type:'div',
    props:{
        children:[
            {
                type:Title,
                props:{title:'리액트 공부하기'},
                //...
            },
            {
                type:'p',
                props:{children:'실전 리액트'},
                //...
            },
            {
                type:'p',
                props:{children:'우선순위 높음'},
                //...
            },
            {
                type:'button',
                props:{
                    onClick: function(){
                        //함수 로직
                    },
                    children:'우선순위 변경',
                },
                //...
            }
        ]
    }
}
* Todo 컴포넌트 함수 호출후의 트리구조이다. Title이 아직 문자열이 아니다. 고로 컴포넌트 함수 호출

* onClick시에도 state값의 변화로 리렌더링이 진행되지만 Title의 경우 React.memo로 작성되었고, 속성값이 변하지 않았기에
이전값이 재사용된다. 즉, Title 컴포넌트 함수가 호출되지않음.

*/
```

#### 리액트 훅 기초
- useState 훅을 이용하면 컴포넌트 상태값을 추가할 수 있다.
- 상태값 변경함수는 배치(한번에 묶여)로 처리된다.(하나의 함수안에서 변경함수가 호출횟수만큼 리렌더링된다면 비효율적이기때문)
- 상태값 변경함수가 리액트 요소에서 사용되지 않는 경우는 배치로 처리되지 않는다. 외부에서 사용되는 경우는 등록하는 함수내에 ReactDOM.unstable_batchedUpdates함수를 사용하여 배치처리시켜야한다.
```
function MyComponent(){
    //const [count,setCount] = useState({ value: 0 });
    const [count,setCount] = useState(0);
    function onClick(){
        //setCount({value : count.value + 1 });
        //setCount({value : count.value + 1 });
        //위처럼하면 count는 1만 증가한다. setCount는 비동기적으로 호출되기때문이다.
        //그러나 밑처럼 함수형태로 처리하면 직전의 결과값을 인수로 받아 처리할 수 있으므로 2만큼 증가한다.
        setCount(prev => prev + 1 );
        setCount(prev => prev + 1 );
    }
    //onClick안에서 상태변경이 두번일어나지만 render call은 한번만 호출된다.
    //위 setCount두개는 배치로 처리되기때문임(두개가 한번에 실행되고 리렌더는 한번만 수행)
    console.log('render call');
    return (
        <div>
            <h2>{count.value}</h2>
            <button onClick={onClick}>증가</button>
        </div>
    )
}
```
- 상태값 변경함수는 호출 순서가 보장된다.
```
function MyComponent(){
    const [count1,setCount1] = useState(0);
    const [count2,setCount2] = useState(0);
    function onClick(){
        setCount1(count1+1);
        setCount2(count2+1);
    }
    const result = count1>=count2; //다음 코드는 항상 true를 반환한다. 항상 setCount1이 먼저실행되기때문임
}
```
- 훅의 상태값 변경함수는 이전 상태값을 덮어쓰므로, 여러 데이터를 가지고 있는 경우에는 전개연산자를 활용해 이전값들을 값을 같이 포함시킨다.
```
import React, { useState } from 'react';

function Profile(){
    const [state,setState] = useState({name:'',age:0});
    return (
        <div>
            <p>{`name is ${state.name}`}</p>
            <p>{`age is ${state.age}`}</p>
            <input
                type="text"
                value={state.name}
                onChange={e => setState({...state,name:e.target.value})}//전개연산자를 활용해 변경하는 속성이외의 이전속성들을 포함시킨다. 
                //전개연산자를 안쓰고 name속성만 포함한다면 state에는 age속성이 사라진다.
            />
            <input
                type="number"
                value={state.age}
                onChange={e => setState({...state,age:e.target.value})}
            />
        </div>
    )
}
```
- useEffect는 부수효과를 처리하는 훅이다.
- useEffect에 등록된 부수효과함수는 렌더링 결과가 실제 돔에 반영된 후 비동기로 호출된다.
```
import React, {useState, useEffect } from "react";

function MyComponent(){
    const [count,setCount] = useState(0);
    useEffect(()=> {
        document.title = `업데이트 횟수 : ${count}`;
    });
    return <button onClick={() => setCount(count+1)}>increase</button>
}

//버튼을 누를때마다 useEffect의 부수효과함수가 호출된다.(리렌더링 되기때문에)
```
- useEffect 두번쨰 인수로 의존성 배열을 넣을 수 있다. useEffect는 의존성배열의 값이 변경된 경우에만 호출된다.
```
import React, { useEffect,useState } from "react";

function Profile({userId}){
    const [user,setUser] = useState(null);
    useEffect(()=>{
        getUserApi(userId).then(data => setUser(data));
    },[userId]);//userId값이 바뀌는 경우에만 호출됨.

    return (
        <div>
            {!user && <p>사용자 정보 받아오는중.. </p>}
            {user && (
                <>
                    <p>{`name is ${user.name}`}</p>
                    <p>{`age is ${user.age}`}</p>
                <>
            )}
        </div>
    );
}
```
- useEffect에 등록된 부수효과함수는 함수를 리턴할 수 있는데 리턴된 함수는 부수효과함수가 호출되기 직전과 컴포넌트가 사라진 후에 호출된다.
```
import React,{ useEffect,useState } from "react";

function WidthPrinter(){
    const [width,setWidth] = useState(window.innerWidth);
    useEffect(()=>{
        const onResize = () => setWidth(window.innerWidth);
        window.addEventListener('resize',onResize);
        //창 크기가 변경될때마다 onResize함수가 호출된다.
        //해당 리스너는 컴포넌트 생성시에만 등록된다.
        return () =>{
            window.removeEventListener('resize',onResize);
        };
        //반환된 함수는 부수효과함수가 호출되기 직전과 컴포넌트가 사라질때만 호출된다.
    },[]);//의존성 배열로 빈배열을 전달하면 부수효과함수는 생성시에만 리턴된 함수는 컴포넌트가 사라질때만 호출된다.
    return <div>{`width: is ${width}`}</div>;
}
```
- 리액트 훅을 사용하여 커스텀 훅을 만들수 있다. 훅을 직접만들면 로직을 쉽게 재사용할 수 있다.
```
//useUser 커스텀훅
function useUser(userId){
    const [user,setUser] = useState(null);
    useEffect(()=>{
        getUserApi(userId).then(data => setUser(data));
    },[userId]);//userId값 변경시에만 호출되어 user값을 갱신한다.
    return user;//state값 리턴
}

function Profile({userId}){
    const user = useUser(userId); //userId값이 바뀔때마다 user변수로 넘어오는 state값이 바뀐다.
    //...
}
```

```
//useWindowWidth 커스텀 훅
import { useEffect, useState } from "react";

function useWindowWidth(){
    const [width,setWidth] = useState(window.innerWidth);
    useEffect(()=>{
        const onResize = () => setWidth(window.innerWidth);
        window.addEventListener('resize',onResize);

        return () => {
            window.removeEventListener('resize',onResize);
        };
    },[]);//의존성배열로 빈배열을 설정하면 부수효과함수는 생성시에만 리턴된 함수는 컴포넌트가 사라질때만 호출된다.
    return width;
}

function WidthPrinter(){
    const width = useWindowWidth();//innerWidth가 변하면 width도 바뀌며 다시렌더링된다.
    return <div>{`width is ${width}`}</div>;
}
```

```
//useMounted 커스텀 훅
function useMounted(){
    const [mounted,setMounted] = useState(false);
    //마운트는 실제돔에 반영된 상태를 말한다.

    useEffect(()=> setMounted(true),[]);//빈배열로 의존성 배열 설정시 부수효과함수는 생성시에만 호출된다.

    return mounted;
}
```
- 훅 사용시 규칙 1 : 하나의 컴포넌트에서 훅 호출순서는 항상 같아야한다.(조건문,반복문,함수안에서 훅을 선언하면 안된다.)
- 훅 사용시 규칙 2 : 훅은 함수형 컴포넌트 또는 커스텀 훅 안에서만 호출돼야 한다.
```
function Profile(){
    const [age,setAge] = useState(0);
    const [name,setName] = useState('');
    //...
    useEffect(()=> {
    //...
    setAge(23); 
    //const [age,setAge] = useState(0); 구문이 조건에 의해 실행되지 않는다면,
    //name 상태값은 23이된다. 내부적으로 훅은 배열로 관리되고, 배열에는 상태값이 선언한 순서대로 들어간다.
    //따라서 setAge시 함수호출시 배열에 첫번째 원소에 저장된 name상태값이 변하므로 name값이 변경된다.
    },[]);
    //...
}
```

#### 콘텍스트 API(상위에서 하위로 데이터 전달)
- 상위컴포넌트에서 하위컴포넌트로 데이터를 전달할때 속성값을 사용하는데, 전달할 컴포넌트가 많거나, 깊이가 깊다면 코드가 복잡해진다. <strong>콘텍스트API를 사용하면</strong> 깊이가 어떻든간에 <strong>상위컴포넌트에서 하위컴포넌트로 직접 데이터</strong>를 전달할 수 있다.
```
const UserContext = React.createContext('');//기본값은 빈문자열

function App(){
    return (
        <div>
            <UserContext.Provider value="mike">//mike라는 값을 consumer로 전달
                <div>상단 메뉴</div>
                <Profile />
                <div>하단 메뉴</div>
            </UserContext.Provider>
        </div>
    )
}

function Profile(){
    return (
        <div>
            <Greeting />
        </div>
    )
}

function Greeting(){
    return (
        <UserContext.Consumer>
            {username => <p>{`${username}님 안녕하세요.`}</p>}//mike값을 username으로 받는다.
        </UserContext.Consumer>
    )
}
```

- 콘텍스트의 Provider와 Consumer컴포넌트를 중첩해서 사용할 수 있으며, 상위컴포넌트에서 하위컴포넌트로 함수를 전달할 수도 있다.(상위컴포넌트의 상태값을 변경하기위한 상태변경함수도 전달가능)
```
const UserContext = React.createContext({ username:"", helloCount:0 });
const SetUserContext = React.createContext(()=>{});

function App(){
    const [user,setUser] = useState({ username:"mike", helloCount:0 });
    return (
        <div>
            <SetUserContext.Provider value={setUser}> //setUser함수 Consumer로 전달
                <UserContext.Provider value={user}>//user 상태값 Consumer로 전달
                    <Profile />
                </UserContext.Provider>
            </SetUserContext>
        </div>
    );
}

function Profile(){
    return(
        <div>
            <Greeting />
        </div>
    )
}

function Greeting(){
    return (
        <SetUserContext.Consumer>
            {setUser => (
                <UserContext.Consumer>
                {({ username,helloCount }) => ( //구조분해할당을 통해 상태값(객체)의 프로퍼티 전달받음 
                    <>
                        <p>{`${username}님 안녕하세요`}</p>
                        <p>{`인사횟수 : ${helloCount}`}</p>
                        <button
                            onClick={() =>
                                setUser({ username,helloCount:helloCount + 1 });//클릭시 user상태값을 덮어씀
                            }
                        >
                    </>
                )}
                </UserContext.Consumer>
            )}
        </SetUserContext.Consumer>
    )
}
```
- 콘텍스트API 사용시 Provider에서 새로운 객체로 데이터를 전달하면 컴포넌트가 생성될때마다 Consumer컴포넌트도 다시 렌더링된다.
```
const UserContext = React.createContext({ username:"" });

function App(){
    const [username,setUsername] = useState("");

    return (
        <div>
            //<UserContext.Provider value={{username}}>
            //다음과같이 새로운 객체로 생성하여 전달하면 
            //Consumer는 동일한 값을 받아도 매번 리렌더링된다.

            <UserContext.Provider value={username}> //이렇게 보내야한다.
            //...
        </div>
    )
}
```

#### ref속성(자식요소 접근하기)
- 자식요소(하위 DOM,컴포넌트)에 접근하기 위해서 ref속성값을 이용할 수 있습니다.
```
import React,{ useRef, useEffect } from "react";

function TextInput(){
    const inputRef = useRef();//useRef훅이 반환하는 ref객체를 통해서 자식요소에 접근가능하다.
    
    useEffect(()=>{
        inputRef.current.focus();//inputRef(ref객체)를 통해 자식에 접근하여 focus함수 실행
    },[]);

    return(
        <div>
            <input type="text" ref={inputRef}>//자식요소 ref속성에 ref객체를 입력해야한다.
            <button>저장</button>
        </div>
    )
}
```
- 손자요소에 접근하기 위해서 속성값 이용해 ref객체를 전달한다.
```
function TextInput({ inputRef }){//속성값을 통해 ref객체를 전달받는다.
    return (
        <div>
            <input type="text" ref={inputRef} />
            <button>저장</button>
        </div>
    )
}

function Form(){
    const inputRef = useRef();
    useEffect(()=>{
        inputRef.current.focus();
    },[]);

    return (
        <div>
            <TextInput inputRef={inputRef} />
            <button onClick={() => inputRef.current.focus()}>텍스트로 이동</button>
        </div>
    );
}
```

- forwardRef함수를 사용하면 부모컴포넌트에서 넘어온 ref값을 직접 처리할 수 있다.
```
const TextInput = React.forwardRef((props,ref)=> (
    //forwardRef함수를 통해 ref객체를 ref라는 이름으로 받는다.
    <div>
        <input type="text" ref={ref} />//ref속성에 ref객체 입력
        <button>저장</button>
    </div>
));

function Form(){
    //...
    return (
        <div>
            <TextInput ref={inputRef} />//컴포넌트를 통해 ref객체전달
            <button onClick={() => inputRef.current.focus() }>텍스트로 이동</button>
        </div>
    )
}
```             
- ref속성값으로 함수를 입력하는 경우 해당함수는 자식요소가 생성되거나 제거되는 시점에 호출된다.
```
function Form(){
    const [text,setText] = useState(INITIAL_TEXT);
    const [showText,setShowText] = useState(true);

    return (
        <div>
            {showText && (
                <input 
                type="text"
                ref={ref => ref && setText(INITIAL_TEXT)}
                //input이 제거될때 함수가 실행된다(ref인수로 null이 넘어옴).
                //즉 아무짓도 하지않음
                //input이 생성될때 함수가 실행된다.(ref인수로 해당 DOM요소(input)참조값이 넘어온다)
                value={text}
                onChange={e => setText(e.target.value)}
                //지금 ref속성값에 함수를 직접 입력하였는데 이러면, 상태변경함수가 실행될때마다,
                //ref로 입력된 함수가 실행된다.(계속 새로운 함수를 입력하기때문)
                //함수가 매번 실행되지 않게 하기위해서는 useCallback(함수재사용훅)을 사용해야한다.
                />
            )}
            <button onClick={() => setShowText(!showText)}>
                보이기/가리기
            </button>
            //최초버튼 클릭시 input이 사라진다.
            //두번째로 버튼클릭시 input이 생성된다.
        </div>
    );
}
const INITIAL_TEXT = "안녕하세요";
```
- useCallback훅을 사용하면 생성된 함수(함수도 일급객체로 주소를 가지고있다.)를 계속 재사용할 수 있다.(참조값 고정)
```
import React,{ useState , useCallback } from "react";

function Form(){
    const [text,setText] = useState(INITIAL_TEXT);
    const [showText,setShowText] = useState(true);

    const setInitialText = useCallback(ref => ref && setText(INITIAL_TEXT),[]);

    return (
        <div>
            {showText && (
                <input 
                type="text"
                ref={setInitialText}
                //상태변경함수가 실행되도 함수주소는 고정이므로 실행되지않음
                value={text}
                onChange={e => setText(e.target.value)}
                />
            )}
            //...
        </div>
    );
}
```

#### 리액트 내장 훅
- useContext를 사용하면 Provider로부터 Consumer컴포넌트없이 데이터를 전달받을 수 있다.
```
const UserContext = React.createContext();
const user = { name:"mike", age:23 };

function ParentComponent(){
    return (
        <UserContext.Provider value={user}>
            <ChildComponent />
        </UserContext.Provider>
    )
}

function ChildComponent(){
    const user = useContext(UserContext);//Provider로 부터 value를 전달받음
    console.log(`user: ${user.name}, age: ${user.age}`);
    //...
}
```

- useRef는 자식요소에 접근하기 위한 용도 이외에도 렌더링과 무관한 값을 저장하는 용도에도 사용한다.
- 무관한값을 상태값에 저장하는 것은 렌더링과 연관되기때문에 적절치않다.
```
import React, { useState, useRef, useEffect } from 'react';

function Profile(){
    const [age,setAge] = useState(20);
    const prevAgeRef = useRef(20);
    useEffect(()=> {
        prevAgeRef.current = age;
    },[age]);//컴포넌트가 마운트된후 호출된다.
    
    const prevAge = prevAgeRef.current;//useEffect보다 먼저 실행됨 즉, 이전 age값이 저장된다.
    const text = age === prevAge ? 'same' : age > prevAge ? 'older' : 'younger';//이전값과 같다면 same 이전값보다 크다면 older 작다면 younger
    return (
        <div>
            <p>{`age ${age} is ${text} than age ${prevAge}`}</p>
            <button
                onClick = {() => {
                    const age = Math.floor(Math.random()*50 + 1);//Math.floor는 정수값 하한을 반환
                    setAge(age);
                }}
            >
            나이 변경
            </button>
        </div>
    );
}
``` 
- useMemo는 함수의 반환값을 재활용하는 용도로 사용된다.
```
import React,{ useMemo } from "react";
import { runExpensiveJob } from './util';

function MyComponent({v1,v2}){
    const value = useMemo(()=> runExpensive(v1,v2),[v1,v2]);//v1,v2가 다를 경우에만 다시 함수를 실행하여 value를 갱신하고 같다면 동일한 반환값을 사용한다.
    return <p>{`value is ${value}`}</p>;
}
```
- useCallback 훅은 컴포넌트가 렌더링 될때마다 새로운 함수가 속성에 입려되는 것으로 인해 자식컴포넌트가 매번 리렌더링되는 것을 방지하기 위함이다. 
```
import React,{ useState, useCallback } from 'react';
import { saveToServer } from './api';
import UserEdit from './UserEdit';

function Profile(){
    const [name,setName] = useState('');
    const [age,setAge] = useState(0);
    const onSave = useCallback(() => saveToServer(name,age),[name,age]);
    return (
        <div>
            <p>{`name is ${name}`}</p>
            <p>{`age is ${age}`}</p>
            <UserEdit
                //onSave={() => saveToServer(name,age)}
                //Profile 컴포넌트가 렌더링될때마다 onSave속성으로 매번 새로운 함수가 입력된다.
                //UserEdit이 React.memo를 사용한다해도 매번 함수주소가 바뀌기때문에 소용이없다.
                //그렇기때문에 함수주소 고정을 위해 useCallback을 사용하면 쓸모없는 리렌더링을 방지할수있다.
                onSave={onSave}
                setName={setName}
                setAge={setAge}
            />
        </div>
    );
}
```

- useReducer를 사용하면 redux처럼 dispatch와 액션을 통해 상태값을 갱신할 수 있으며, 갱신된 상태값을 참조할 수 있다.
```
import React,{ useReducer } from 'react';

const INITIAL_STATE = {name: 'empty',age: 0};
function reducer(state,action){
    switch(action.type){
        case 'setName'://dispatch로 날아온 action type이 setName일경우
            return {...state,name: action.name};//이전 state에서 name속성만 갱신한뒤 상태값덮어씀
        case 'setAge':
            return {...state,age: action.age};
        default:
            return state;
    }
}

function Profile(){
    const [state,dispatch] = useReducer(reducer,INITIAL_STATE);
    return (
        <div>
            <p>{`name is ${state.name}`}</p>
            <p>{`age is ${state.age}`}</p>
            <input
                type="text"
                value={state.name}
                onChange={e =>
                    dispatch({ type:'setName',name:e.currentTarget.value })
                }
            />
            <input
                type="number"
                value={state.age}
                onChange={e =>
                    dispatch({ type:'setAge',age:e.currentTarget.value })
                }
            >
        </div>
    );
}
```

- 하위 컴포넌트로 dispatch를 전달해야 하위컴포넌트에서 dispatch호출을 통해 state를 갱신 할 수있다.
```
//...
export const ProfileDispatch = React.createContext(null);
//...
function Profile(){
    const [state,dispatch] = useReducer(reducer,INITIAL_STATE);
    return (
        <div>
            <p>{`name is ${state.name}`}</p>
            <p>{`age is ${state.age}`}</p>
            <ProfileDispatch.Provider value={dispatch}>
                <SomeComponent /> 
                //해당 컴포넌트 하위에 있는 모든 컴포넌트는 콘텍스트를 통해
                //dispatch를 호출할수 있습니다.
            </ProfileDispatch.Provider>
        </div>
    );
}
```

- useImperativeHandle훅을 사용하면 부모 컴포넌트에서 접근가능한 함수를 만들 수 있다.
```
//자식 컴포넌트
import React, { forwardRef,useState,useImperativeHandle } from "react";

function Profile(props,ref){
    const [name,setName] = useState('');
    const [age,setAge] = useState(0);

    useImperativeHandle(ref, () => ({
        addAge: value => setAge(age+value),
        getNameLength: () => name.length,
    }));

    return (
        <div>
            <p>{`name is ${name}`}</p>
            <p>{`age is ${age}`}</p>
            //...
        </div>
    )
}

export default forwardRef(Profile);
```
```
//부모 컴포넌트
function Parent(){
    const profileRef = useRef();
    const onClick = () => {
        if(profileRef.current){
            console.log('current name length:',profileRef.current.getNameLength());
            profileRef.current.addAge(5);
        }
    };

    return (
        <div>
            <Profile ref={profileRef} />
            <button onClick={onClick}>add Age 5</button>
        </div>
    );
}
```
- useLayoutEffect는 useEffect가 렌더링결과가 돔에 반영된후 비동기적으로 호출되는거와 달리 동기적으로 호출된다.(렌더링 직후 돔요소의 값을 읽는 경우에는 useLayoutEffect가 적절하다.)
- useDebugValue는 커스텀 훅에서 디버깅을 위해 사용할 수 있다.
```
function useToggle(initialValue){
    const [value,setValue] = useState(initialValue);
    const onToggle = () => setValue(!value);

    useDebugValue(value? 'on':'off');//디버깅시 확인할 값을 훅의 매개변수로 입력한다.
    return [value,onToggle];
}
```
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


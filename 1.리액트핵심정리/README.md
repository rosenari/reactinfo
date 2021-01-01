### 리액트 핵심정리

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

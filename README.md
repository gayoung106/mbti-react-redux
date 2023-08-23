#  redux를 활용한 mbti프로젝트

##  사용스택 및 라이브러리(React)
~~~
npx create-react-app mbti-test

npm install react-redux redux
npm install styled-components

npm start
~~~


![](https://velog.velcdn.com/images/gazero_/post/1c0ca269-9c47-4c90-b4d7-2e96ac9282a3/image.gif)

## 구조
- 메인(home)화면에서 시작버튼을 누르면
- 질문 1번이 시작됨 질문은 그렇다/아니다
~~~
I/E, S/N, T/F, P/J 중
각각 하나씩
E S T J의 포인트 점수를 조정하는 방식
예를들어, '그렇다'를 누르면 E포인트 INCREASE상태가 되고,
'아니다'를 누르면 E포인트 DECREASE상태로 점수를 조절
~~~
- 질문이 다 끝나면, 포인트 점수를 합산해서 결과를 보여줌
- 결과페이지에는 다시하기(retry)버튼이 있음

## 1. 저장고(store) 만드는 과정
### mbtiStore.js 파일생성
#### 초기상태 정의
~~~js
const initialState = {
  isTestBegin: false,
  questionCount: 0,
  ePoint: 0,
  sPoint: 0,
  tPoint: 0,
  jPoint: 0,
};
~~~
- 초기상태(시작하기 전): 시작여부 false/ 질문숫자는 0/ e, s, t, j 포인트 상태는 0으로 정의해둠

#### reducer함수 정의
~~~js
function reducer(currentState, action) {
  ///여기에 들어갈 조건들///
}
~~~
- reducer 함수는 매개변수로 현재상태와 action을 담고 있음
- 첫번째 조건, 만약에 현재 상태값이 undefined거나 type이 'RESET'이라면 초기 상태로 리턴해라.
- 새로운 상태(업데이트된 현재상태)에 현재상태를 담고(undefined가 아니고, 초기 상태가 아니면)있다는 전제하에
- 두번째 조건, 만약에 type이 'BEGIN'이라면 상태의 isTestBegin은 true로 바꿔라(질문으로 넘어감)
- 세번째 조건, type이 'INCREASE'일때 각각 포인트의 상태 변화
- 네번째 조건, type이 'DECREASE'일때 각각 포인트의 상태 변화

#### 이제 만들어 둔 reducer함수를 store에 담아둠(with. createStore)
~~~js
import { createStore } from "redux";

/* ...초기상태
   ...리듀서함수*/

export const store = createStore(reducer);
~~~

## 2. 상태를 전역적으로 뿌려주겠다! (with. Provider)
~~~js
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
~~~
- 이런식으로 App컴포넌트를 Provider컴포넌트로 감싸줌

#### 일단 Redux 준비 끝!

## 3. Home 화면
- 메인(home)화면에서 시작버튼을 눌러서 상태에 변화를 줄 것임
- 따라서, useDispatch가 필요
~~~js
import { useDispatch } from "react-redux";

const Home = () => {
  const dispatch = useDispatch();
  
  return (
    
  <StButton onClick={() => dispatch({ type: "BEGIN" })}>
    
  );
};

export default Home;
~~~
- 이런식으로 dispatch로 원하는 action type으로 수정함
- BEGIN
cf. 만약에 type이 'BEGIN'이라면 상태의 isTestBegin은 true로 바꿔라
- 다음 질문 페이지로 넘어감 

## 4. 질문 화면
- 현재 몇번째 질문은지에 대한 현재 상태를 받아와야 하기 때문에
- useSelector필요

~~~	js
import { useSelector, useDispatch } from "react-redux";

const Question = (props) => {
  const questionCount = useSelector((state) => state.questionCount);

  
  return (
       <Title>
          <TitleNum>{questioncontent[questionCount].number}</TitleNum>
          <h1>{questioncontent[questionCount].question}</h1>
        </Title>


    
  );  
};

export default Question;
~~~
- 그리고 버튼을 누를때마다 포인트 점수를 업데이트 해줘야 하므로
- useDispatch필요
- 버튼을 누르고나면 다음 질문으로 넘어감
~~~js
import { useSelector, useDispatch } from "react-redux";

const Question = (props) => {

  const dispatch = useDispatch();
  
  return (
     
    
      <OkButton
            onClick={() =>
              dispatch({
                type: "INCREASE",
                quizType: questioncontent[questionCount].type,
              })
            }
          >
            <span> {questioncontent[questionCount].answer1} </span>
          </OkButton>
          <br />
          <OkButton
            onClick={() =>
              dispatch({
                type: "DECREASE",
                quizType: questioncontent[questionCount].type,
              })
            }
          >
    
  );  
};

export default Question;
~~~

## 5. 질문의 상태변화
- 질문에 응답할때마다 다음질문으로 넘어감(+1씩 상태변화)
- 질문은 총16개, 따라서 16개의 질문이 다 끝나면 결과페이지로 넘어가고, 질문이 아직 16개가 다 끝나지 않았다면 계속 질문페이지에서 다음 질문으로 넘어감
~~~js

const Game = () => {
  const questionCount = useSelector((state) => state.questionCount);
  return (
    <div style={{ textAlign: "center" }}>
      {questionCount === 16 ? <Result /> : <Question />}
    </div>
  );
};

export default Game;
~~~

## 6. 결과페이지
- 각각 포인트의 합산 점수 상태(useSelector)를 받아오고
~~~js
  const ePoint = useSelector((state) => state.ePoint);
  const sPoint = useSelector((state) => state.sPoint);
  const tPoint = useSelector((state) => state.tPoint);
  const jPoint = useSelector((state) => state.jPoint);
~~~
- 페이지가 최초 로딩될 때, 그 합산 값에 대한 결과가 노출되게 만듦
~~~js
  const [resultCharacter, setCharacter] = useState({});

  useEffect(() => {
    let result = "";
    result += ePoint > 0 ? "E" : "I";
    result += sPoint > 0 ? "S" : "N";
    result += tPoint > 0 ? "T" : "F";
    result += jPoint > 0 ? "J" : "P";

    const finalResult = resultcontent.filter((content) => {
      return content.type === result;
    });
    setCharacter(...finalResult);
  }, []);

~~~
- retry버튼으로 상태에 변화를 줄 것이므로 useDispatch를 사용
~~~js
 <StButton onClick={() => dispatch({ type: "RESET" })}>RETRY</StButton>
~~~

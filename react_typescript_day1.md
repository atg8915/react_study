# React & TypeScript 학습 노트 - Day 1

> 범위: React 기초 개념 정리(어제) + JSX/Hooks 문법 + 리스트 렌더링 + axios 서버 연동 + TypeScript 기초(오늘)

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|---|---|---|
| 컴포넌트 정의 | `function App(props){ return (...) }` | 함수형 컴포넌트, 첫 글자는 대문자 |
| 컴포넌트 내보내기 | `export default App` | 외부에서 `import App from './App'`로 사용 |
| JSX 최상위 태그 | `<>...</>` (Fragment) | JSX는 최상위 태그가 반드시 1개, `<> </>`로 감싸면 실제 DOM에 태그 안 남음 |
| 속성에 변수 넣기 | `<img src={변수}/>` | HTML 속성값에 JS 표현식은 `{}` 사용 |
| 인라인 스타일 | `style={{width:'30px'}}` | 바깥 `{}`는 JS 표현식, 안쪽 `{}`는 객체 |
| className | `className="text-center"` | JSX에서는 `class` 대신 `className` |
| ⭐ state 선언 | `const [count,setCount]=useState(0)` | `[값, 값을 바꾸는 함수] = useState(초기값)`, set 함수 호출 시 재렌더링 |
| ⭐ state 갱신(콜백형) | `setCount(count => count+1)` | 이전 값을 기준으로 갱신할 때 함수형 업데이트 사용 |
| ⭐ 리스트 렌더링 | `list.map((item,index)=> <tr key={index}>...</tr>)` | 배열 → JSX 배열로 변환, `key` 필수 |
| ⭐ 마운트/의존성 훅 | `useEffect(()=>{...},[curpage])` | 두 번째 인자 배열의 값이 바뀔 때마다 재실행, `[]`면 최초 1회만 |
| 서버 통신 | `axios.get(url).then(res=>{...})` | 비동기로 서버 데이터를 받아 state에 저장 |
| 엔트리 포인트(Vite) | `createRoot(el).render(<App/>)` | `react-dom/client`의 `createRoot` 사용 |
| 엔트리 포인트(CRA) | `ReactDOM.createRoot(el).render(<App/>)` | Create-React-App 방식, import 경로만 다름 |
| ⭐ TS 타입 지정 | `let a:number = 10` | `변수명:타입 = 값` |
| TS 유니언 타입 | `let a:number\|string = 10` | 여러 타입 중 하나 허용 |
| TS 타입 추론 | `let b = 100 // number로 자동 추론` | 초기값이 있으면 타입 생략 가능 |

## 1. 기술 흐름 / 개념

**React를 쓰는 이유**
- 서버(Spring Boot 등)와 화면(React)을 분리 → 서버는 API(JSON)만 제공, 화면은 React가 전담 (MSA 구조)
- 페이지 전체 새로고침 없이 필요한 부분만 갱신 (JSP/Thymeleaf: 요청→서버→HTML 생성→갱신 vs React: 요청→JSON→state 변경→화면 갱신)
- 컴포넌트 단위 재사용, 가상 DOM 사용, 비동기 처리로 서버 통신과 화면 UI 분리 가능

**단점 / 실무 이슈**
- 코드 복잡도 증가
- CORS(Cross Domain) 문제 → 세션/쿠키 처리 별도 필요
- API URL과 화면 라우팅 URL 관리를 구분해야 함
- 단방향 데이터 흐름이라 컴포넌트 간 공유 데이터는 별도 저장소(Redux 등) 필요

**코딩 방식 변화**
- 과거: class형 컴포넌트 (`class App1 extends Component`, `componentDidMount`, `componentDidUpdate`)
- 현재: function형 컴포넌트 + Hooks (`useState`, `useEffect`)

**JSX(JavaScript + XML) 규칙**
1. 최상위 태그는 반드시 1개
2. 여는 태그/닫는 태그 반드시 일치, 단독 태그는 `<input/>`, `<br/>`, `<img/>`처럼 슬래시로 닫기
3. 속성값은 `''`, `""` 또는 변수식 `{}`
4. HTML 태그는 소문자, 컴포넌트(함수/클래스)는 대문자로 시작
5. `<태그>{변수}</태그>` 형태로 데이터 출력
6. 변수 종류: 지역변수(`let`,`const`) / state(변경되는 데이터) / props(변경 안 되는 데이터, 불변)

**변수 관점에서 본 React 핵심 문법**
1. `import` - 외부 라이브러리/컴포넌트 읽기
2. `function 컴포넌트명()` - React 컴포넌트 정의
3. `useState()` - 상태(데이터) 관리
4. `useEffect()` - 렌더링 후(또는 데이터 변경 후) 작업 수행
5. `axios` - 서버 연동
6. `{}` - 화면에 데이터 출력
7. `style={{}}` - 인라인 스타일
8. `className=""` - CSS 클래스 적용 (`class`는 사용 불가)
9. `export default` - 컴포넌트 외부 공개

## 2. React 핵심 개념 정리 (App1.jsx)

props로 전달받은 값을 화면에 출력하는 가장 단순한 함수형 컴포넌트 예시.

```jsx
function App1(props) {
  // 서버에서 데이터 읽기 (예정)
  return (
    <div>
      <h1>Hello React</h1>
      <h1>{props.name}</h1>
    </div>
  );
}
export default App1;
```

- ⭐ `props`는 부모 컴포넌트가 내려준 읽기 전용 데이터. `{props.name}`처럼 JSX 안에서 바로 출력.
- 주석으로 정리된 Hooks 개념:
  - `useState(초기값)` → `[값, set함수]` 반환, set함수 호출 시 재렌더링되며 값이 바뀜 (문자열/배열/객체 모두 가능: `useState("")`, `useState([])`, `useState({})`)
  - `useEffect(()=>{...})` → mounted 시점 + 의존성 배열의 값이 바뀔 때마다 실행. 서버 데이터 읽기, 이벤트 등록, 타이머, 외부 API 호출, DOM 작업 등에 사용
  - `useEffect(()=>{...},[])` → 최초 1회만 실행
  - `useEffect(()=>{...},[curpage])` → `curpage`가 바뀔 때마다 재실행
  - `useMemo`, `useCallback` → 불필요한 재계산/재호출 방지 (React 17에서 활성화)

## 3. 프로젝트 구조 비교: Vite vs CRA

같은 "React 시작하기" 화면이지만 빌드 도구에 따라 엔트리 코드가 다름.

**Vite 방식 (main.jsx + App.jsx)**
```jsx
// main.jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```
```jsx
// App.jsx - useState로 카운터 구현
function App() {
  const [count, setCount] = useState(0)
  return (
    <button onClick={() => setCount((count) => count + 1)}>
      Count is {count}
    </button>
  )
}
```

**CRA(Create React App) 방식 (index.js + App.js)**
```jsx
// index.js
import ReactDOM from 'react-dom/client';
import App from './App';
// NodeJS 설치 후: npm -g install create-react-app

const root = ReactDOM.createRoot(document.querySelector("#root"));
root.render(<App />);
```
```jsx
// App.js
function App() {
  return (
    <div className="App">
      <a href="https://reactjs.org" target="_blank" rel="noopener noreferrer">
        React Basic!!
      </a>
    </div>
  );
}
```

| 구분 | Vite | CRA |
|---|---|---|
| 엔트리 파일 | `main.jsx` | `index.js` |
| Root 생성 | `createRoot(...)` (react-dom/client에서 직접 import) | `ReactDOM.createRoot(...)` |
| 개발 서버 | 빠름(esbuild 기반) | 상대적으로 느림 |

## 4. 리스트 렌더링과 테이블 출력 (App2.js)

⭐ 배열 데이터를 `map`으로 순회하며 JSX 배열을 만들고, 테이블로 출력하는 패턴.

```jsx
function App2(props) {
  const html = props.movie.map((m, index) =>
    <tr key={index}>
      <td className="text-center">{m.rank}</td>
      <td className="text-center">
        <img src={"https://www.kobis.or.kr" + m.thumbUrl} alt="영화포스터"
             style={{ width: '30px', height: '30px' }} />
      </td>
      <td>{m.movieNm}</td>
      <td>{m.director}</td>
      <td>{m.genre}</td>
    </tr>
  )

  return (
    <table className="table table-striped">
      <thead>
        <tr>
          <th>순위</th><th></th><th>영화명</th><th>감독</th><th>장르</th>
        </tr>
      </thead>
      <tbody>{html}</tbody>
    </table>
  );
}
export default App2
```

- 반복 렌더링은 `for...of`가 아니라 배열의 `map`을 사용 (JSX는 표현식만 `{}` 안에 넣을 수 있음)
- `key`는 각 행을 식별하기 위해 필수 (여기서는 index 사용)
- 이미지 경로처럼 baseURL + 상대경로를 문자열 결합(`+`)으로 완성하는 패턴

## 5. useEffect + axios로 서버 연동 & 페이지네이션 (Food.jsx)

⭐ 실무에서 가장 많이 쓰는 패턴: state로 페이지 정보를 관리하고, `curpage`가 바뀔 때마다 서버에서 새 데이터를 받아온다.

```jsx
import { useState, useEffect } from 'react'
import axios from "axios";

function Food() {
  const [list, setList] = useState([]);
  const [curpage, setCurpage] = useState(1);
  const [totalpage, setTotalpage] = useState(0);
  const [startPage, setStartPage] = useState(0);
  const [endPage, setEndPage] = useState(0);

  // curpage가 변경될 때마다 서버 재호출
  useEffect(() => {
    axios.get(`http://localhost:8080/food/list/${curpage}`)
      .then(res => {
        setList(res.data.list)
        setCurpage(res.data.curpage);
        setTotalpage(res.data.totalpage);
        setStartPage(res.data.startPage);
        setEndPage(res.data.endPage);
      })
  }, [curpage]);

  let html = list.map((f, index) =>
    <div className="col-md-3" key={index}>
      <div className="thumbnail">
        <img src={f.poster} alt={f.address}
             style={{ width: "250px", height: "130px", objectFit: "cover" }} />
        <p>{f.name}</p>
      </div>
    </div>
  )

  return <div className="row">{html}</div>
}
export default Food;
```

- 서버 주소는 개발용 로컬 서버(`localhost:8080`) — 배포 시 환경변수로 분리 필요
- `useEffect`의 의존성 배열에 `curpage`를 넣어 페이지 번호가 바뀔 때마다 자동으로 재조회
- 서버 응답으로 리스트뿐 아니라 페이지네이션 정보(`curpage`, `totalpage`, `startPage`, `endPage`)까지 함께 state로 관리

## 6. TypeScript 기초 (index.ts)

TypeScript = JavaScript에 데이터형(Type) 문법을 추가한 언어. 가독성과 유지보수성이 목적.

```ts
let b: number = 100        // 타입을 생략해도 초기값으로 자동 추론됨
let a: number | string = 10 // ⭐ 유니언 타입: 여러 타입 허용
a = "aaa"                   // string도 허용되므로 에러 없음
```

- ⭐ 기본 타입 지정 문법: `변수명:타입 = 값`
- ⭐ 유니언 타입(`|`)으로 하나의 변수에 여러 타입 허용 가능
- 초기값이 있으면 타입 표기를 생략해도 TS가 자동으로 타입을 추론 (`let b:number`와 동일하게 동작)
- 오늘은 변수 타입 선언까지 학습, 다음 단계는 함수 타입 / 인터페이스 / React(tsx) 연동 예정

## 실습 파일 목록

| 파일명 | 주제 | 핵심 내용 |
|---|---|---|
| react-basic-project/src/App1.jsx | React 개념 정리 + props | React 장단점, JSX 문법 규칙, Hooks 개념 주석 정리, props 출력 |
| react-basic-project/src/App2.js | 리스트/테이블 렌더링 | `map`으로 배열 → `<tr>` 리스트 변환, 이미지 경로 결합 |
| react-basic-project/src/Food.jsx | axios + useEffect + 페이지네이션 | 서버 통신, state 기반 페이지 정보 관리 |
| react-basic-project/src/App.jsx | Vite 기본 템플릿 | `useState`로 카운터, 이미지 import |
| react-basic-project/src/App.js | CRA 기본 템플릿 | CRA 기본 구조, 정적 링크 출력 |
| react-basic-project/src/main.jsx | Vite 엔트리 포인트 | `createRoot` + `StrictMode` |
| react-basic-project/src/index.js | CRA 엔트리 포인트 | `ReactDOM.createRoot`, 초기 데이터로 컴포넌트 렌더링 |
| type-project/src/index.ts | TypeScript 기초 | 변수 타입 선언, 유니언 타입, 타입 추론 |

## README 한 줄

| Day 1 | React 핵심 개념/JSX 문법/Hooks(useState·useEffect) + 리스트 렌더링 + axios 서버 연동 + TypeScript 변수 타입 기초 | [📄](./react_typescript_day1.md) |

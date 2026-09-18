# React & TypeScript 학습 노트 - Day 2

> 범위: react-basic-project2(TSX) 기준 — React Router 라우팅 + TypeScript interface로 props/state 타입 지정 + axios 인스턴스 패턴 + Kakao Map API 연동
> 이전 내용: [Day 1](./react_typescript_day1.md) (React 기초 개념, JSX/Hooks, 리스트 렌더링, axios, TS 변수 타입)

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|---|---|---|
| ⭐ 라우터 설정 | `<Router><Routes><Route path="/" element={<Home/>}/></Routes></Router>` | `react-router-dom` v7. `Router`=관리자, `Routes`=화면 모음, `Route`=화면 1개 |
| ⭐ 화면 이동 링크 | `<Link to="/food/find">맛집 검색</Link>` | `<a>` 대신 사용, 새로고침 없이 화면 전환 |
| ⭐ 코드로 이동 | `const nav = useNavigate(); nav(-1)` | `nav(-1)`=뒤로가기, `nav("/path")`=특정 경로 이동 |
| ⭐ URL 파라미터 읽기 | `const {no} = useParams<{no:string}>()` | `request.getParameter("no")`에 대응, `/food/detail/:no` 형태 경로에서 사용 |
| interface로 타입 정의 | `interface Food{ no:number; name:string; }` | 객체(props, 서버 응답 데이터) 모양을 미리 정의 |
| state에 타입 지정 | `useState<FoodProps>()` | 제네릭으로 state 타입 명시, 초기값 없으면 `undefined` 가능 |
| ⭐ 옵셔널 체이닝 | `foodData?.list.map(...)` | state가 `undefined`일 수 있을 때 안전하게 접근 (에러 방지) |
| axios 인스턴스 분리 | `axios.create({baseURL, headers})` | 매번 전체 URL을 쓰지 않고 공통 설정을 재사용 |
| async/await 서버 통신 | `const res = await apiClient.get(url)` | `.then()` 체이닝 대신 동기처럼 작성 (Day1은 `.then` 방식) |
| 조건부 렌더링 | `{detail && <KakaoMap .../>}` | 값이 있을 때만 컴포넌트 출력 |
| 외부 전역 객체 타입 선언 | `declare global{ interface Window{ kakao:any } }` | TS에서 모르는 외부 스크립트 전역 변수(`window.kakao`) 타입 선언 |

## 1. React Router로 화면 분리 (App.tsx)

Day1까지는 컴포넌트 하나(`Food`)만 `index.js`에서 바로 렌더링했지만, Day2는 여러 화면을 URL 경로별로 나눠서 관리한다.

```tsx
import { BrowserRouter as Router, Route, Routes } from "react-router-dom";
import Header from "./components/main/Header";
import FoodFind from "./components/food/FoodFind";
import Detail from "./components/food/Detail";
import Home from "./components/main/Home";

function App() {
  return (
    <Router>
      <Header/>
      <Routes>
        <Route path="/" element={<Home/>}></Route>
        <Route path="/food/detail" element={<Detail/>}></Route>
        <Route path="/food/find" element={<FoodFind/>}></Route>
      </Routes>
    </Router>
  );
}
export default App;
```

- ⭐ 구조: `Router`(관리자) → `Routes`(화면 모음) → `Route`(화면 1개, `path`+`element`)
- `Header`는 `Routes` 밖에 둬서 모든 페이지에 공통으로 표시 (네비게이션 바)
- JSP의 "요청 URL → 컨트롤러 매핑"과 유사한 개념으로 이해하면 쉬움

## 2. 컴포넌트 폴더 구조 정리

Day1은 `src/` 바로 아래 컴포넌트를 뒀지만, Day2부터는 역할별로 폴더를 나눔.

```
src/
 ├─ App.tsx                     # 라우터 설정
 └─ components/
     ├─ main/
     │   ├─ Header.tsx          # 공통 네비게이션
     │   └─ Home.tsx            # 메인(목록) 화면
     ├─ food/
     │   ├─ FoodFind.tsx        # 검색 화면
     │   └─ Detail.tsx          # 상세 화면
     └─ commons/
         ├─ http-commons.ts     # axios 공통 설정
         └─ KakaoMap.tsx        # 지도 컴포넌트
```

- `main`(공통 레이아웃/홈), `food`(도메인 기능), `commons`(재사용 유틸/공통 컴포넌트)로 관심사 분리
- 실무에서 자주 쓰는 구조: 기능(도메인) 단위 폴더 + 공통 모듈 폴더

## 3. Link와 useNavigate로 화면 이동 (Header.tsx / Detail.tsx)

```tsx
// Header.tsx - 메뉴 클릭 시 새로고침 없이 이동
import { Link } from "react-router-dom";
function Header() {
  return (
    <nav className="navbar navbar-inverse">
      <Link className="navbar-brand" to={"/"}>MiniReact</Link>
      <li><Link to="/food/find">맛집 검색</Link></li>
    </nav>
  );
}
```

```tsx
// Detail.tsx - 버튼 클릭 시 코드로 이전 화면 이동
import { useNavigate } from "react-router-dom";
function Detail() {
  const nav = useNavigate();
  return <button onClick={() => nav(-1)}>목록</button>;
}
```

- ⭐ `<Link>`는 화면 위 클릭 이동, `useNavigate()`는 이벤트 핸들러 안에서 코드로 이동시킬 때 사용
- `nav(-1)` = 브라우저 뒤로가기 1칸, `nav("/food/find")`처럼 경로를 직접 줄 수도 있음
- `<a href="#">`로 남아있는 메뉴(레시피 목록 등)는 아직 라우트 미구현 상태 (다음 학습 예정 부분으로 보임)

## 4. TypeScript interface로 데이터 모양 정의 (Home.tsx / Detail.tsx)

Day1에서는 변수에 `number`, `string` 같은 단순 타입만 지정했다면, Day2는 서버 응답 객체 전체의 모양을 `interface`로 미리 정의하고 state/props에 적용한다.

```tsx
interface Food {
    no: number;
    name: string;
    poster: string;
    address: string;
}
interface FoodProps {
    list: Food[];
    curpage: number;
    totalpage: number;
    startPage: number;
    endPage: number;
}

function Home() {
    const [curpage, setCurpage] = useState<number>(1);
    const [foodData, setFoodData] = useState<FoodProps>(); // 초기값 없음 → undefined 가능

    useEffect(() => {
        const fetchList = async () => {
            const res = await apiClient.get(`/food/list/${curpage}`);
            setFoodData(res.data);
            return res.data;
        };
        fetchList();
    }, [curpage]);

    // ⭐ foodData가 undefined일 수 있으므로 ?. (옵셔널 체이닝) 사용
    const html = foodData?.list.map((food: Food) =>
        <Link to={'/food/detail/' + food.no}>{food.name}</Link>
    );
    ...
}
```

- ⭐ `interface`는 객체(주로 서버 응답 데이터, props)의 필드와 타입을 미리 정의 → 자동완성 + 오타/타입 실수 방지
- `useState<FoodProps>()`처럼 제네릭(`<>`)으로 state의 타입을 지정, 초기값을 안 주면 타입은 `FoodProps | undefined`
- `foodData?.list`처럼 `?.`을 붙여야 `foodData`가 `undefined`일 때 에러 없이 안전하게 접근 (Day1의 JS 버전에서는 이런 타입 체크가 없었음)
- `Detail.tsx`도 동일한 패턴으로 `FoodDetailData` interface를 만들어 상세 데이터 타입을 지정

## 5. axios 인스턴스 분리 (http-commons.ts)

Day1(`Food.jsx`)에서는 매번 `axios.get("http://localhost:8080/food/list/...")`처럼 전체 URL을 직접 썼지만, Day2는 공통 설정을 인스턴스로 분리해서 재사용한다.

```ts
// components/commons/http-commons.ts
import axios, { AxiosInstance } from "axios";

const apiClient = axios.create({
    baseURL: "http://localhost:8080",
    headers: { "Content-Type": "application/json" }
});
export default apiClient;
```

```tsx
// 사용하는 쪽 (Home.tsx)
import apiClient from "../commons/http-commons";
const res = await apiClient.get(`/food/list/${curpage}`); // baseURL이 자동으로 붙음
```

- ⭐ `axios.create()`로 `baseURL`, 공통 헤더 등을 한 곳에서 관리 → 서버 주소가 바뀌어도 파일 하나만 수정하면 됨
- 여러 컴포넌트에서 같은 서버와 통신할 때 매번 전체 URL/헤더를 반복 작성하지 않아도 됨 (실무 필수 패턴)

## 6. async/await 방식의 서버 통신 (Home.tsx / Detail.tsx)

Day1은 `.then()` 체이닝으로 비동기 처리를 했는데, Day2는 `useEffect` 내부에 `async` 함수를 선언해 `await`로 처리하는 방식으로 바뀜.

```tsx
useEffect(() => {
    const fetchDetail = async () => {
        const res = await apiClient.get(`/food/detail_react/${no}`);
        setDetail(res.data);
        return res.data;
    };
    fetchDetail();
}, []);
```

- ⭐ `useEffect`의 콜백 함수 자체는 `async`로 만들 수 없어서, 내부에 별도 `async` 함수(`fetchDetail`)를 선언 후 즉시 호출하는 패턴 사용
- `.then(res => {...})`(Day1) vs `const res = await ...`(Day2) — 결과는 같지만 코드가 동기 코드처럼 읽혀 가독성이 좋아짐

## 7. URL 파라미터로 상세 페이지 구현 (Detail.tsx)

```tsx
import { useNavigate, useParams } from "react-router-dom";

function Detail() {
    const { no } = useParams<{ no: string }>(); // request.getParameter("no")에 대응
    const [detail, setDetail] = useState<FoodDetailData | null>(null);

    useEffect(() => {
        const fetchDetail = async () => {
            const res = await apiClient.get(`/food/detail_react/${no}`);
            setDetail(res.data);
        };
        fetchDetail();
    }, []);

    return (
        <table>
            <tbody>
            <tr><td>{detail?.name}</td></tr>
            <tr><td>{detail?.address}</td></tr>
            {detail && <KakaoMap address={detail?.address} name={detail?.name}/>}
            </tbody>
        </table>
    );
}
```

- ⭐ 목록 화면(`Home.tsx`)에서 `<Link to={'/food/detail/' + food.no}>`로 이동 → 상세 화면(`Detail.tsx`)에서 `useParams()`로 그 `no` 값을 꺼내 서버에 상세 정보 요청
- `{detail && <KakaoMap .../>}` : `detail`이 아직 `null`(서버 응답 전)이면 지도를 렌더링하지 않음 → 데이터 없는 상태에서 에러 방지

## 8. Kakao Map API 연동 (KakaoMap.tsx)

외부 지도 API(카카오맵 JS SDK)를 React 컴포넌트로 감싸서 재사용하는 패턴.

```tsx
declare global {
    interface Window { kakao: any }
}
interface MapProps { address: string, name: string }

const KakaoMap = ({ address, name }: MapProps) => {
    useEffect(() => {
        const { kakao } = window;
        const map = new kakao.maps.Map(document.getElementById('map'), {
            center: new kakao.maps.LatLng(33.450701, 126.570667),
            level: 3
        });
        const geocoder = new kakao.maps.services.Geocoder();
        // 주소 문자열 → 좌표로 변환 후 마커 표시
        geocoder.addressSearch(address, (result: any, status: string) => {
            if (status === kakao.maps.services.Status.OK) {
                const coords = new kakao.maps.LatLng(result[0].y, result[0].x);
                new kakao.maps.Marker({ map, position: coords });
                map.setCenter(coords);
            }
        });
    }, [address]);

    return <div id="map" style={{ width: "100%", height: "350px" }}></div>;
};
export default KakaoMap;
```

- ⭐ `declare global { interface Window { kakao: any } }` : `index.html`에 `<script>`로 불러온 카카오맵 SDK처럼, TS가 타입을 모르는 외부 전역 객체는 이렇게 타입을 직접 선언해줘야 컴파일 에러가 안 남
- `geocoder.addressSearch(주소, 콜백)` : 주소 문자열 → 위경도 좌표로 변환(Geocoding), 결과로 지도에 마커 표시
- `useEffect(..., [address])` : `address`(부모로부터 받은 props)가 바뀔 때마다 지도를 다시 그림
- props로 `address`, `name`을 받아 여러 상세 페이지에서 재사용 가능한 지도 컴포넌트로 분리

## 실습 파일 목록 (Day 2 신규/추가분)

| 파일명 | 주제 | 핵심 내용 |
|---|---|---|
| react-basic-project2/src/App.tsx | 라우터 설정 | `BrowserRouter`, `Routes`, `Route`로 화면 분리 |
| react-basic-project2/src/components/main/Header.tsx | 공통 네비게이션 | `Link`로 새로고침 없는 화면 이동 |
| react-basic-project2/src/components/main/Home.tsx | 목록 + 페이지네이션 | interface 타입 지정, 옵셔널 체이닝, async/await |
| react-basic-project2/src/components/food/Detail.tsx | 상세 화면 | `useParams`, `useNavigate`, 조건부 렌더링 |
| react-basic-project2/src/components/food/FoodFind.tsx | 검색 화면(뼈대) | 라우트 연결만 된 상태, 기능 구현 예정 |
| react-basic-project2/src/components/commons/http-commons.ts | axios 인스턴스 | `baseURL`/헤더 공통화 |
| react-basic-project2/src/components/commons/KakaoMap.tsx | 지도 API 연동 | 전역 타입 선언, Geocoding, 마커 표시 |

## 다음 학습 예정

- `FoodFind.tsx` 실제 검색 기능 구현 (검색어 state + axios 조회)
- Header의 미구현 메뉴(레시피 목록/검색, 쉐프, 커뮤니티) 라우트 연결
- 로그인/인증 처리, 전역 상태 관리(Redux 등) 도입 여부

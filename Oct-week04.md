# GraphQL

### 앞으로 아래의 기술 사용할 예정
- scratch만을 이용해 Idiomatic GraphQL server 만들기
- graphql-yoga: 
- Prisma
- GraphQL Playground

## GraphQL(gql)이란?
> 페이스북에서 만든 쿼리언어로 REST API를 간편하게 대체할 수 있기에 상승세.

## SQL과 GQL
1. SQL: 데이터베이스 시스템에 저장된 데이터를 효율적으로 가져오는 것이 목적
2. GQL: 웹 클라이언트가 데이터를 서버로부터 효율적으로 가져오는 것이 목적

## REST API와 GQL


## 시작하기
> 프로젝트 디렉토리 만들고 Node.js app을 위한 환경설정(configuratoin) 완료.
1. 프로젝트 디렉토리 만들기: ... cmd창에 `init -y` 입력하면 해당 디렉토리 `package.json`으로 초기화
2. raw GraphQL 서버 만들기: `src` 디렉토리 만들고 안에 `touch src/index.js` 
3. dependency 추가하기: `npm install graphql-yoga`
- `graphql-yoga`는 `Express.js`에 기반을 둔 모든 기능을 갖춘 GQL 서버?
4. `index.js` 수정
  ```javascript
  const { GraphQLServer } = require('graphql-yoga')
  
  // 1
  const typeDefs = `
  type Query {
    info: String!
  }
  `

  // 2
  const resolvers = {
    Query: {
      info: () => `This is the API of a Hackernews Clone`
    }
  }

  // 3
  const server = new GraphQLServer({
    typeDefs,
    resolvers,
  })
  server.start(() => console.log(`Server is running on http://localhost:4000`))
  ```
  - `typeDefs`: 내 GQL의 스키마schema를 정의해줌. 클래스 같은 거일 듯. `!`는 `null`값이 올 수 없음을 뜻한다.
  - `resolvers`: 실제로 스키마를 실행시키는 애. `typeDefs`와 내부 구조는 동일해야 한다.
  - `server`: 두 기능을 번들로 묶어 `GraphQLServer`로 보냄. 이걸 위해 `graphql-yoga` import한거.

5. `node src/index.js` 입력해 서버 실행시켜보고 `localhost:4000` 접속해보면 GraphQL Playground 등장
  - GraphQL Playground: GraphQL API를 GUI로 보여주고, 상호작용할 수 있는 GraphQL IDE이다. GUI로 데이터베이스의 구조를 쉽게 파악하고, 외부 데이터 입력 시 정상작동하는지 테스트하기 편리.
  

  <궁금한 점>
  1. 저희가 스파르타에서 배운 API를 얻기 위해 ajax 사용한 게 바로 REST API인건가요?
  2. 시작하기 중 2번 내용을 cmd창에 입력했더니 
  'touch'은(는) 내부 또는 외부 명령, 실행할 수 있는 프로그램, 또는 배치 파일이 아닙니다.
  라는 에러 떠서 구글링대로 환경변수에 %SystemRoot%\system32;%SystemRoot 추가했는데도 에러가 여전히 나는데 혹시 해결법 아시나요..?ㅜㅜ

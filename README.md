# [TIL] 5-3. 백엔드 - Express : 예외 처리, 미니 프로젝트(회원 API)

# ✅ 핸들러(Handler)
HTTP request 가 날아오면 자동으로 호출되는 메소드를 뜻한다. Node.js 에서는 콜백함수를 핸들러라고 한다.
```
app.get("/channels", (req, res) => {
  var channels = {};
  db.forEach((value, key) => {
    channels[key] = value;
  });
  res.json(channels);
});
```
이 코드에서는 req, res 를 전달하는 익명 콜백함수가 핸들러가 된다.

# ✅ if문
if문을 사용할 때 가장 처음 오는 조건은 긍정문을 쓰는 것이 좋다. 이를 테면 첫 if문의 조건이 false 가 아닌 true 를 반환하는 코드가 좀 더 클린코드 관점에서 좋다는 뜻이다. 그 예로 DELETE 메서드로 개별삭제 기능 코드를 수정해 보도록 하자.
```
if (channel) {
    const channelTitle = channel.channelTitle;
    db.delete(id);
    res.json({
      message: `'${channelTitle}' 채널이 삭제되었습니다.`,
    });
  } else if (channel == undefined) {
    res.json({
      message: `요청하신 ${id} 경로에 존재하는 채널이 없습니다. 경로를 다시 한 번 확인해주세요.`,
    });
  }
```
원래 예외 처리하는 코드가 먼저였고, undefined 가 아니면 삭제가 되도록 if문 순서를 작성했지만, 위처럼 true 를 반환하는 코드가 먼저 나오고, 예외처리 되는 코드를 뒷부분에 작성하는 것이 더 좋다.

# ✅ HTTP 상태 코드를 이용한 예외 처리
HTTP 상태 코드를 사용해서 예외 처리를 해보자. 프론트 상에서는 에러가 뜨지는 않지만 실제로 에러인 예외를 터뜨려 처리하는 일이다. 먼저 예외 처리를 위한 코드를 작성하면서, json array와 find() 함수에 대해 공부해보자.

## 💬 json array
지금까지 Map 을 이용했다면, 이번에는 json 객체를 배열 안에 넣어서 사용할 것이다.
```
const teams = [
  { id: 1, name: "heroes" },
  { id: 2, name: "dinos" },
  { id: 3, name: "lions" },
  { id: 4, name: "giants" },
];

// 전체 조회
app.get("/teams", (req, res) => {
  res.json(teams);
});
```
<img width="623" alt="image" src="https://github.com/user-attachments/assets/5aa20d68-9a03-4713-81ae-33d0dc3846be" /><br />
전체 조회는 배열을 통으로 json 으로 보냈기 때문에, 당연히 배열 안에 json 객체들이 들어있다. 그렇다면 개별 조회도 배열의 인덱스를 꺼내 사용하면 될 것이다. 실제로 프론트엔드에 json을 여러개 보낼 때 배열을 사용한다.
```
// 개별 조회
app.get("/teams/:id", (req, res) => {
  let id = req.params.id;
  let team = teams[id];

  res.json(team);
});
```
url로 들어온 id 값을 꺼내서 변수에 담은 후, 배열의 인덱스에 id 값을 넘겨서 꺼내온 값을 변수에 저장해준다.
<img width="623" alt="image" src="https://github.com/user-attachments/assets/71c119cc-d5ac-4421-b09a-9258599bdd33" /><br />
하지만 배열의 인덱스는 0부터 시작하기 때문에, 경로를 1로 넣으면 두번째 인덱스의 값이 출력된다. 이를 해결하기 위해서 id 값에서 1을 빼주어도 된다.

하지만 우리가 url로 보낸 숫자는 배열의 인덱스 값이라기 보다는, 각 개체가 갖고 있는 id 값일 것이다. 따라서 각 개체 중에 해당하는 id 값을 찾아 전달해주어야 한다. 지난 시간에 공부한 forEach문을 사용할 수 있다.
```
app.get("/teams/:id", (req, res) => {
  let id = req.params.id;
  var findTeam = "";

  teams.forEach((team) => {
    if (team.id == id) {
      findTeam = team;
    }
  });

  res.json(findTeam);
});
```
forEach문의 매개변수는 값, 인덱스, 전체 배열 순으로 받아올 수 있는데, 여기서는 값만 필요하므로 team 이라는 매개변수에 배열의 값들을 하나씩 받아오도록 한다. 받아온 team 이라는 객체가 가지고 온 id 값이 url 로 들어온 id 값과 같으면 res body 에 전달할 json 에 담아 전달하면 된다. if문이 true를 반환할 때, 받아온 team 을 함수 밖에 선언해 둔 빈 변수 findTeam 에 저장해서 이 변수를 res body 에 전달하는 것이다.
<img width="623" alt="image" src="https://github.com/user-attachments/assets/f17f3554-bb2b-42c0-8de4-280c8d07a9c5" /></br />

그런데 개별 조회하는 코드를 단 한 줄로 줄일 수 있다. 이 때 사용하는 함수가 바로 find() 함수이다.

## 💬 find()
```app.get("/teams/:id", (req, res) => {
  let id = req.params.id;
  var findTeam = teams.find((f) => f.id == id);

  res.json(findTeam);
});
```
findTeam 이라는 변수에 teams 라는 배열에서 f => f.id 객체의 id 값이 url로 들어온 id 값과 같은 것을 find() 찾아서 담는다. 즉, teams 배열 안에 있는 객체 중, id 값이 params.id와 같은 객체를 찾겠다는 뜻이다.

## 💬 예외 처리
<img width="622" alt="image" src="https://github.com/user-attachments/assets/66d22936-1fa2-4bdd-9bd8-15858da484db" /><br />
없는 경로를 요청하면 res body에 아무것도 안 뜨는데, 상태 코드가 200 OK 라고 뜬다. 하지만 res body에 아무것도 뜨지 않는 이유는 요청을 성공하지 못 했기 때문에 아무런 데이터를 띄우지 못하는 것이다. 이럴 때 직접 예외를 터뜨려 처리해 주어야 한다.
```
 app.get("/teams/:id", (req, res) => {
  let id = req.params.id;
  var findTeam = teams.find((f) => f.id == id);

  if (findTeam) {
    res.json(findTeam);
  } else {
    res.status(404).send("찾으시는 팀이 존재하지 않습니다.");
  }
});
```
만약 json 을 전달하기 전에, find로 찾는 id가 없으면 findTeam 에 아무것도 안 담길 것이다. 그렇다는 것은 findTeam 이 undefined 라는 뜻이다. 따라서 if문을 사용한다.
또한 예외를 터뜨린다는 것은, http 상태 코드를 임의로 실패하게 한다는 것이다. 이것은 status 메소드를 사용할 수 있다. 여기에 실패 코드인 404를 작성해 주고, 어떤 메세지를 전달할 지 작성하면 된다. 
<img width="626" alt="image" src="https://github.com/user-attachments/assets/88d78e48-8c85-4a93-bff3-47b6075503c3" /><br />
아까와는 다르게 상태코드가 404로 뜨고, 작성한 실패 메세지도 전달이 되는 것을 확인할 수 있다. 그러니까 상태 코드를 임의로 작성하지 않고 그냥 send 로 보내면 200이 뜨지만, status 라는 메소드를 통해 고의로 예외를 터뜨려 처리를 하는 것이다.

# ✅ ``==`` vs ``===``
```
if (1 == "1") {
  console.log("같다");
} else {
  console.log("같지 않다");
}
```
위 코드를 실행하면 콘솔창에 어떤 문구가 찍힐까? 답은 '같다' 이다. 그렇다면 ``==``를 ``===``로 바꾸면? '같지 않다' 가 찍힌다.
``==``와 ``===``의 차이는 '자료형'을 판단하는 차이이다. ``==``는 자료형 상관 없이 오로지 값만 비교한다. 하지만 ``===``은 값 뿐만 아니라 자료형까지 비교하여 연산한다.

```
app.get("/teams/:id", (req, res) => {
  let id = req.params.id;
  var findTeam = teams.find((f) => f.id == id);

  if (findTeam) {
    res.json(findTeam);
  } else {
    res.status(404).send("찾으시는 팀이 존재하지 않습니다.");
  }
});
```
이 코드를 보면, req.params로 받아온 url의 id 값은 사실 문자열이다. 그래서 저번 시간에 parseInt로 정수 변환을 해주었다. ``==`` 는, 자료형이 아닌 값만 비교하기 때문에 해당 코드가 정상적으로 돌아가는 것이다.


# ✅ 예외 고도화
Channel API 설계에서 예외에 대한 코드를 고도화 시켜보자.

## 💬 Map은 undefined가 아니다
원래 코드는 값이 있으면 전달하고, 없으면 빈 값을 전달했다. 예외를 터뜨려서 처리를 해보자.
```
app.get("/channels", (req, res) => {
  var channels = {};

  if (db) {
    db.forEach((value, key) => {
      channels[key] = value;
    });
    res.json(channels);
  } else {
    res.status(404).json({
      message: "조회할 채널이 없습니다.",
    });
  }
});
```
db에 값이 있으면 전달하고, 예외를 터뜨려 404 상태 코드가 뜬다면 조회할 채널이 없다는 메세지를 전달한다. 그런데 해당 코드를 포스트맨에서 get 요청하면, 에러 메세지가 아니라 빈 ``{}``가 뜬다. 왜일까?
채널을 전체 삭제한 뒤 콘솔창에 db를 찍어보면, ``Map(0)``이 뜬다. 이건 db가 아예 존재하지 않는 게 아니라, db가 있긴 있는데 안에 들어있는 요소 개수가 0일 뿐이다. 때문에 항상 true를 반환한다. 따라서 db의 객체 존재 여부가 아니라, db의 size로 비교하는 코드를 써야 한다.
<img width="618" alt="image" src="https://github.com/user-attachments/assets/3ecc059c-8587-4a8c-8819-0cb984cfd55f" /><br />
예외 처리에 맞는 에러 메세지가 잘 전달되는 것을 확인할 수 있다.

## 💬 POST에 대한 예외
반대로 등록하는 메서드인 POST의 예외는 res body에 전달되는 값이 아무것도 없다면? 이라고 생각할 수 있다. 값을 안 넣고 요청하면 어떻게 되는지 확인해보자.
<img width="621" alt="image" src="https://github.com/user-attachments/assets/817cbcb4-3a88-456e-a86a-f092bb96eef2" /><br />
들어가야 할 데이터가 들어가지 않았는데 200 OK가 뜨지만, 이것은 예외 처리를 해줘야 하는 상황이다.
```
app.use(express.json());
app.post("/channels", (req, res) => {
  const channelTitle = req.body.channelTitle;
  if (channelTitle) {
    db.set(id++, req.body);
    res.status(201).json({
      message: `신규 채널 '${
        db.get(id - 1).channelTitle
      }' 이(가) 개설되었습니다.`,
    });
  } else {
    res.status(400).json({
      message: "채널명은 필수입니다.",
    });
  }
});
```
먼저 set하기 전에, req body 값에 channelTitle이 있는지 먼저 확인을 해줘야 한다. 있으면 정상적으로 등록해주면 된다. 없다면 예외를 터뜨려 처리해준다. 그런데 404는 서버에서 찾아봤는데 전달해 줄 리소스 페이지가 없다는 것이었다. 이 때는 400, '서버가 요청의 구문을 인식하지 못함' 이라는 뜻을 가진 상태 코드를 사용해 줄 수 있다. 즉 400은, 요청한 연산(처리)을 할 때 필요한 데이터(req)가 덜 왔을 때 뿌려지는 상태 코드이다.
<img width="623" alt="image" src="https://github.com/user-attachments/assets/092dcaca-51f2-493a-a94f-9f246e493566" /><br />

또 상태 코드에서, 200은 조회,수정,삭제 성공 시의 상태 코드이다. 그런데 포스트맨에서 POST 요청을 하면 200이 뜬다. 등록 성공의 상태 코드는 201이기 때문에, 이 부분의 상태 코드도 201로 고쳐줄 수 있다.
<img width="619" alt="image" src="https://github.com/user-attachments/assets/d1243ca6-9107-4105-8760-9232c7fddc5f" /><br />

# ✅ 미니 프로젝트
지금까지 배웠던 것들로 미니 프로젝트를 진행해보자. 진짜로 동영상 사이트를 운영한다고 가정하고, 어떤 API들이 필요한지부터 정할 수 있다.

## 💬 회원 API 설계
- 회원
  - 로그인 : POST /login
    - req : body(id, pw)
    - res : `${name}님, 환영합니다.` 👉🏻 메인 페이지
  - 회원가입 : POST /join
    - req : body(id, pw, name)
    - res : `${name}님, 가입을 축하합니다.` 👉🏻 로그인 페이지
  - 회원 정보 (개별)조회 : GET /members/:id
    - req : url(id) 
    - res : id, name
  - 회원 (개별)탈퇴 : DELETE /members/:id
    - req : url(id)
    - res : `${name}님, 또 만나기를 바랍니다.` (OR 메인 페이지)

### 로그인 페이지
```
- id 입력란
- pw 입력란
- 로그인 버튼
```
이 로그인 페이지를 완성하는 데 필요한 API가 없지만, 로그인 버튼을 누르는 것은 **id와 pw를 받아서 로그인을 시켜줄 API**가 필요하다.

### 회원가입 페이지
```
- id 입력란
- pw 입력란
- 이름 입력란
- 회원가입 버튼
```
이 페이지도 마찬가지로 완성하는 데에는 API가 필요 없지만, 회원가입 버튼을 눌렀을 때 **입력한 id, pw, 이름을 받아서 등록해줄 API**가 필요하다.

### 회원탈퇴 - 마이 페이지
```
- id 데이터
- 이름 데이터
- 회원탈퇴 버튼
```
화면을 생성할 때는 **회원정보를 조회하는 API**가 필요하고, 회원탈퇴 버튼을 눌렀을 때 **id, 이름을 받아서 탈퇴를 시켜줄 API**가 필요하다.

### 회원 API 설계 결과 : https://www.notion.so/API-19f888d4ff738098811fffe623be9366?pvs=4

## 💬 중복되는 URL을 깔끔하게
URL에 따라 메서드가 분리되는 것을 '라우팅 한다'고 표현한다. 회원 개별 조회나 회원 개별 탈퇴 API는 요청 메서드는 다르지만 URL은 같다. 이런 경우 중복되는 부분을 합쳐주는 방법도 존재한다.
app에는 route 라는 메서드가 있다.
```
app.route("/users/:id").get().delete()
```
"/users/:id" 라는 url이 날아왔을 때, 그 중에 메서드가 get일 때와 delete일 때를 한 번에 작성해 줄 수 있다. 그렇다면 get과 delete 메서드에는 각각 따로 작성해주었을 때의 콜백함수를 넣어주면 된다.
```
app
  .route("/members/:id")
  .get((req, res) => {
    let { id } = req.params;
    id = parseInt(id);

    const member = members.get(id);
    if (member) {
      res.json({
        userId: member.userId,
        name: member.userName,
      });
    } else {
      res.status(404).json({
        message: "사용자를 찾을 수 없습니다.",
      });
    }
  })
  .delete((req, res) => {
    let { id } = req.params;
    id = parseInt(id);

    const member = members.get(id);
    if (member) {
      members.delete(id);
      res.status(200).json({
        message: `${member.userName}님, 또 만날 수 있기를 바랍니다.`,
      });
    } else {
      res.status(400).json({
        message: "존재하지 않는 회원입니다.",
      });
    }
  });
```

## 💬 채널 API 설계
- 채널  <- 회원은 계정 1개당 채널 100개 생성 가능
  - 채널 생성
  - 채널 수정
  - 채널 삭제

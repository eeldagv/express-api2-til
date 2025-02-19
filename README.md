# [TIL] 5-3. 백엔드 -

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
아까와는 다르게 상태코드가 404로 뜨고, 작성한 실패 메세지도 전달이 되는 것을 확인할 수 있다.


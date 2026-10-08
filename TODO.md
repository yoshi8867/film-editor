# 할 일 — 배포와 파티 맺기

> **다른 PC 에서 세션을 열면 이 파일부터 읽는다.**
>
> 이 레포(`film-editor`)는 **공개**다. 화면 한 장짜리만 올라간다.
> 코드는 비공개 레포 **`yoshi8867/treasure-hunter`** 에 있다.
> 여기에는 **접속 문자열도, 비밀번호도, 보물 QR 도 적지 않는다.**

지금까지는 **한 PC 안에서** iframe 으로 여럿인 척했다. 폰 넷이 따로 들어와
한 파티가 되려면 **셋을 같이 아는 무언가**가 있어야 한다 — 누가 들어왔나 ·
언제 시작했나 · 맞았나. 정적 페이지끼리는 서로를 모른다. 그걸 세우는 일이다.

---

## 0. 세션 열면 먼저

- [ ] `git clone https://github.com/yoshi8867/treasure-hunter.git yeouiju`
      (이미 있으면 `git pull`)
- [ ] `CLAUDE.md` 를 읽는다 — 지킬 규칙이 다 거기 있다
- [ ] **`TODO LIST/배포.md`** 를 읽는다 — 이 파일의 자세한 판이다.
      까닭까지 적혀 있으니 결정을 다시 하지 않아도 된다
- [ ] `TODO LIST/목록.md` — 게임 지도 (열다섯 판 · 적어만 둔 넷 · 보류 셋)
- [ ] `pip install -r requirements.txt` · `python server/app.py` 가 뜨는지 본다
      (**`PYTHONIOENCODING=utf-8` 을 걸고 띄운다.** 안 걸면 cp949 에서 죽는다)

---

## 1. 정한 것 — 다시 고르지 않는다

| 무엇 | 어디 | 왜 |
| --- | --- | --- |
| 화면 | 깃헙 페이지스 (`yoshi8867.github.io/…`) | 공짜 · 안 잠듦 · **주소가 안 바뀐다** |
| API | Render 무료 | 판정과 시각은 여기만 안다 |
| 곳간 | Neon 무료 (Postgres) | 파티 · 도토리 · 기록 |
| 깨우기 | 크론이 `/health` 를 친다 | Render 는 15분 놀면 잠든다 |

**화면을 페이지스에 두는 까닭은 보안이 아니라 종이다.** QR 은 주소를 통째로
품고 벽에 붙는다(보물찾기만 열다섯 장). API 주소를 QR 에 넣으면 호스팅을
옮길 때마다 **붙인 걸 다 떼고 다시 찍어야 한다.** 페이지스를 가리키게 두면
옮길 때 고칠 곳은 `server/static/ys.js` 의 `API` **한 줄**뿐이다.

---

## 2. 할 일 차례

### 0번 — 끝났다

- [x] `/health` 를 낸다. **곳간을 안 건드린다** (깨우기 ping 때문에 Neon 을
      내내 돌릴 까닭이 없다)
- [x] CORS 허용 목록 — `ALLOW_ORIGIN` 환경변수, 기본값은 페이지스 한 곳
- [x] `ys.js` 의 `API` 한 줄 + 화면이 뜨자마자 `/health` 한 번 치기

지금 `API = ""` 라 **아무것도 안 바뀐 상태**다. 서버가 화면까지 주는 지금
방식도, 혼자 서는 한 장도 그대로 돈다. Render 주소가 나오면 그 줄에 적는다.

### 1번 — `store.py` 를 Neon 으로 갈아 끼운다 ← **여기부터**

**여기가 서야 나머지 다섯이 다 얹힌다.** 지금 `server/store.py` 는 메모리에만
담아서 서버를 끄면 사라지고, Render 는 아무 때나 껐다 켠다.

- [ ] `server/schema.sql` 의 얼개를 Neon 에 올린다
      (`party` · `member` · `task` · `attempt` · `acorn_ledger` · `party_acorns`)
- [ ] `store.py` 안쪽만 Postgres 로 바꾼다. **바깥에서 부르는 함수 모양은
      그대로 둔다** — `party` · `seen` · `crew` · `join` · `task` · `down` ·
      `up` · `snapshot` · `reset`. 이 이름들이 열두 게임에서 다 불린다
- [ ] `psycopg[binary]` 를 `requirements.txt` 에 더한다
- [ ] 연결을 **풀**로 쓴다. 무료 티어는 연결 수가 적다
- [ ] `DATABASE_URL` 이 없으면 **메모리로 떨어지게** 둔다 — 집에서 시험할 때
      Neon 없이도 돌아야 한다
- [ ] 누름 시각은 지금처럼 `attempt.presses` JSONB 한 칸에 담는다.
      **게이지 값은 저장하지 않는다** — 시각만 있으면 언제든 다시 계산된다

### 2번 — 파티 맺기

화면 하나를 새로 만든다. 글자는 안 쓴다 — QR · 다람쥐 얼굴 · 금빛 버튼
셋으로 다 말이 된다.

```
1  자리마다 붙은 QR        /t/cctv
   처음 잡은 다람쥐가 **대장**. 서버가 파티를 새로 만든다

2  대장 화면에 **초대 QR**  /j/<파티코드>
   다람쥐 얼굴 자리가 2~4개 비어 있다 (.ys-mug.empty — 점선 테)

3  다른 다람쥐가 그 QR 을 잡는다
   빈 굴이 하나씩 얼굴로 찬다. **모두의 화면에서 같이** 찬다

4  대장이 금빛 버튼을 탭
   그때 인원이 굳고, 넷이 같이 시작한다
```

- [ ] `/j/<코드>` 길과 들어가는 API
- [ ] 초대 QR 을 화면에 그린다 (`qr/hunt` 가 이미 QR 을 그리고 있다 — 그 부품을 쓴다)
- [ ] 대장만 보이는 시작 버튼. 누른 뒤에는 인원이 굳는다
- [ ] 폰에 파티 코드를 넣어 둔다(`localStorage`). **두 번째 QR 부터는 초대가
      필요 없다** — 각자 아무 QR 이나 잡으면 제 자리로 바로 들어간다

### 3번 — Render 배포

- [ ] 배포 설정 파일 (`render.yaml` 또는 대시보드에서 손으로).
      `gunicorn` 은 이미 `requirements.txt` 에 있다
- [ ] 환경변수 **`DATABASE_URL`** · **`ALLOW_ORIGIN`** · `PYTHONIOENCODING=utf-8`
- [ ] 뜬 주소를 `ys.js` 의 `API` 한 줄에 적는다

### 4번 — 크론 걸기

Render 무료는 **달에 750시간**이다. 24시간 내내 깨우면 31일짜리 달이
**744시간**이라 남는 게 여섯 시간뿐이고, 무료 서비스를 하나만 더 올려도
터진다.

- [ ] **평일 07:30~18:00 · 10분마다 `/health`** → 월 230시간쯤. 넉넉하다
- [ ] **cron-job.org 나 UptimeRobot** 을 쓴다.
      깃헙 액션 스케줄은 **쓰지 않는다** — 붐비면 10~20분씩 밀려서 15분이면
      잠드는 Render 를 못 지킨다. 레포가 60일 조용하면 스케줄이 꺼지기도 한다

### 5번 — 화면을 페이지스로

지금 굽개 넷(`solo.py` · `layer_solo.py` · `speed_solo.py` · `film_solo.py`)은
**규칙을 페이지 안에 박아서** 혼자 서게 만든다. 그래서 폰 넷이 못 모인다.

- [ ] 굽개를 고쳐, 규칙을 박는 대신 **`API` 를 바라보는** 한 벌을 더 낸다
      (`fetch` 를 가로채지 않고 그냥 내보내면 된다)
- [ ] **정답이 든 것은 페이지스에 안 올린다.** 지금 `cctv.html` · `film.html`
      은 문제집을 통째로 박고 있다 — 혼자 하는 판이라 괜찮지만, 서버와 같이
      도는 판은 그렇게 굽지 않는다
- [ ] 올리개로 올린다 — `python 올리기.py <파일> <올린이름>`

### 6번 — 나머지 열한 판

`sync` · `add` · `make` · `count` · `word` · `timer` · `spin` · `shape` ·
`speak` · `cards` 를 페이지스로.

- [ ] **`hunt` 은 못 올린다.** 보물 QR 열다섯 장이 그대로 답이라 공개 레포에
      올리면 다 보인다 — **그 판만 Render 가 화면까지 준다**

---

## 3. 사람이 정해야 하는 것 — 아직 안 정했다

- [ ] **최소 인원** — 대장이 시작을 누르면 2인이어도 바로 가나, 몇 마리는 모여야 하나
- [ ] **늦게 온 다람쥐** — 게임이 도는 중에 초대 QR 을 잡으면 어떻게 되나
- [ ] **파티 코드 꼴** — `schema.sql` 에는 「여우비」 같은 짧은 말로 적어 뒀다.
      QR 로만 들어오면 읽기 쉬울 까닭이 없으니 짧은 영숫자여도 된다
- [ ] **파티가 언제 해산하나** — 하루? 손으로? `party.ended_at` 만 비워 뒀다
- [ ] **누구인지 적을 것인가** — 학번? 이름? `docs/논의점.md` 에 미뤄 둔 그것이다

## 4. 사람이 손으로 해야 하는 것 — 클로드가 못 한다

- [ ] **Neon 프로젝트를 만든다.** 접속 문자열은 **채팅에 붙이지 말고**
      `.env`(gitignore 된 곳)나 Render 환경변수에 넣는다
- [ ] **Render 에 레포를 연결한다** (비공개 레포라 권한을 줘야 한다)
- [ ] **cron-job.org 에 등록한다**
- [ ] **솔방울 그림** — `server/static/img/speed/3.png` 이 아직 빈 한지다.
      제미나이로 뽑아서 갈아 끼운다 (256×256 · 검정 실루엣 · 한지 바탕)

---

## 5. 지킬 다섯

1. **CORS 는 `https://yoshi8867.github.io` 하나만.** `*` 안 쓴다
2. **정답이 든 것은 페이지스에 안 올린다**
3. **API 주소는 `ys.js` 의 `API` 한 줄**에만 적는다
4. **화면이 뜨자마자 `/health` 를 한 번 친다** — 파티가 모이는 동안 깬다
5. **API 가 제 판 번호를 알려 주고**, 화면이 제 것과 다르면 스스로
   새로고침한다 (아직 안 만들었다)

### 레포의 규칙에서, 이 일에 걸리는 것

- **고치면 곧바로 알릴 파일** — `ys.css` · `ys.js` · `design.html`.
  부품은 한 벌뿐이라, 이 셋은 **따로 커밋하고 사람에게 말한다**
- **한 줄만 더하는 파일** — `games.py` · `app.py` · `README.md` ·
  `목록.md` · `test_design.py`
- **셸 명령에 한글을 직접 넣지 않는다** (cp949 에서 깨진다).
  한글이 든 커밋 메시지는 파일에 적어 `git commit -F <파일>`
- **커밋하면 곧바로 푸시한다**
- **화면에 글자를 두지 않는다.** 운영자 화면(`/games` · `/design` · `/lab/*`)만 예외
- **색 하나에 뜻 하나** — 금 = 손대야 하는 것 · 녹색 = 켜졌다 · 적색 = 틀렸다

---

## 6. 지금 올라가 있는 것

| 한 장 | 무엇 |
| --- | --- |
| [speed.html](https://yoshi8867.github.io/film-editor/speed.html) | 스피드 슬라이드 **완성** (`?n=3` · `?n=4`) |
| [speed-lab.html](https://yoshi8867.github.io/film-editor/speed-lab.html) | 스피드 슬라이드 규격 실험실 |
| [cctv.html](https://yoshi8867.github.io/film-editor/cctv.html) | CCTV 미로 **완성** (혼자) |
| [layer.html](https://yoshi8867.github.io/film-editor/layer.html) | 필름 나눠 그리기 **완성** (2~4) |
| [film.html](https://yoshi8867.github.io/film-editor/film.html) | 필름 겹치기 (기획을 바꿀 예정이라 보류) |
| [maze.html](https://yoshi8867.github.io/film-editor/maze.html) | CCTV 실험실 |
| [curl.html](https://yoshi8867.github.io/film-editor/curl.html) | 벌집 컬링 — **폐기한 기획.** 되살리려면 `games.py` 의 `hold` 한 줄만 지운다 |
| [book.html](https://yoshi8867.github.io/film-editor/book.html) | 책 바코드 읽기 (게임 아님 · 연장) |
| [index.html](https://yoshi8867.github.io/film-editor/) | 필름 편집기 |

**이 넷만 게임으로 할 수 있다** — `speed`(2~4) · `cctv`(혼자) ·
`layer`(2~4) · `film`(보류). 나머지는 다 연장이다.
**5번이 서면 이 넷도 진짜 폰 넷이서** 하게 바뀐다.

---

## 7. 딴 길에 있는 일 — 지금은 건드리지 않는다

- **책 빌리기 판** — 멈춰 세워 뒀다. 「재개하려는데 뭐부터?」 하고 물으면
  `TODO LIST/book/README.md` 를 꺼낸다
- **필름 겹치기** — 기획 자체를 바꿀 생각이라 손대지 않는다
- **벌집 컬링** — 폐기

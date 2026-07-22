# 버스 도착정보 화면

경기도 버스정보(GBIS) Open API를 이용해 등록한 정류장의 실시간 도착정보를 보여주는
독립 화면입니다. 급식 앱과 동일한 구조(GitHub Pages + Cloudflare Workers)로 만들었습니다.

## 구성
- `worker.js` — Cloudflare Worker. GBIS API 서비스키를 서버 쪽에서만 보관하고,
  브라우저 요청을 대신 GBIS API로 전달(프록시)합니다. **키는 절대 프론트엔드 코드에 넣지 않습니다.**
- `index.html` — 정류장을 검색해서 등록하고, 등록된 정류장 탭을 눌러 30초마다
  자동 갱신되는 도착정보를 보여주는 프론트엔드입니다.

## 배포 방법

### 1. Worker 배포
```bash
npm install -g wrangler   # 이미 설치되어 있다면 생략
wrangler login
cd bus-info

# wrangler.toml이 없다면 아래처럼 최소 설정으로 만드세요:
#   name = "bus-info"
#   main = "worker.js"
#   compatibility_date = "2024-01-01"

wrangler secret put GBIS_SERVICE_KEY
# → 프롬프트가 뜨면 공공데이터포털에서 발급받은 "디코딩된" 서비스키를 붙여넣으세요

wrangler deploy
```
배포가 끝나면 `https://bus-info.<계정이름>.workers.dev` 같은 주소가 나옵니다.

### 2. 프론트엔드 배포
`index.html`을 새 GitHub 저장소(예: `bus-info`)에 올리고 GitHub Pages를 켜면 됩니다.
급식 앱 저장소 설정하신 방식과 동일합니다.

### 3. 최초 설정
배포된 페이지에 접속하면 맨 위에 "Worker 주소 설정" 칸이 보입니다.
1단계에서 받은 Worker 주소를 입력하고 저장하면, 이후에는 브라우저에 저장되어
다시 입력할 필요가 없습니다.

### 4. 정류장 등록
검색창에 정류장 이름(예: 성빈센트병원)을 입력하고 검색 → 결과에서 "추가"를 누르면
탭에 등록됩니다. 자주 쓰실 4곳(성빈센트병원, 한국전력경기사업본부, 한라시그마팰리스,
동수원사거리)은 빠른 검색 버튼으로 미리 넣어뒀습니다.

⚠️ 정류장 번호판에 적힌 4~5자리 번호(예: 03074)는 GBIS API가 쓰는 `stationId`와
다릅니다. 반드시 이름으로 검색해서 나온 결과 중 맞는 정류장을 선택해 추가하세요.
검색 결과에 함께 표시되는 ID가 실제 `stationId`입니다.

## 참고사항 (중요)
GBIS API의 실제 JSON 응답 필드명은 문서마다 표기가 조금씩 달라(`predictTime1` /
`PREDICT_TIME1` 등) 이 코드에서는 여러 표기를 함께 검사하도록 만들었습니다.
다만 제가 실제 API를 직접 호출해서 응답을 확인할 수 있는 환경이 아니라서,
실제 배포 후 도착정보가 이상하게 나오면:
1. 도착정보 카드 하단의 "원본 응답 보기"를 눌러 실제 JSON 구조를 확인하고
2. 그 내용을 저에게 붙여넣어 주시면 `renderBusItem` / `extractList` 함수의
   필드명 매핑을 정확히 맞춰드릴게요.

## 급식 앱 연동
급식 앱 메인 화면에 아래처럼 링크만 추가하면 됩니다:
```html
<a href="https://<사용자명>.github.io/bus-info/">🚌 버스 도착정보</a>
```

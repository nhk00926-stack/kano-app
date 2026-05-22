# 마이리얼트립 AI 챗봇 — Kano 모형 실시간 설문 · 설치·배포 안내

학생들이 휴대폰(QR/링크)으로 11개 기능에 대한 긍정·부정 응답을 제출하면, 교수님 화면(`dashboard.html`)에 **실시간으로** 분류·집계·시각화되는 웹앱입니다. 데이터는 무료 Supabase에 저장됩니다. (SERVQUAL 실습과 동일한 구조)

구성 파일은 4개입니다.
- `index.html` — 학생 응답 페이지 (QR로 배포)
- `dashboard.html` — 교수용 실시간 대시보드 (강의 화면)
- `config.js` — Supabase 주소·키, 세션 코드, 11개 기능·Kano 평가표 (★ 여기만 수정)
- `qr.html` — 학생용 링크의 QR코드 생성 페이지

---

## 1단계 · Supabase 프로젝트 만들기 (무료, 약 5분)

1. https://supabase.com 로그인 → **New project** (Region은 `Northeast Asia (Seoul)` 권장).
2. 좌측 **SQL Editor → New query**에 아래를 붙여넣고 **Run**.

```sql
create table kano_responses (
  id bigint generated always as identity primary key,
  created_at timestamptz default now(),
  session text not null,
  pos jsonb not null,   -- 기능별 긍정질문 응답 {f1:0, f2:1, ...} (0~4)
  neg jsonb not null    -- 기능별 부정질문 응답
);

alter table kano_responses enable row level security;

create policy "anyone can insert" on kano_responses
  for insert to anon with check (true);
create policy "anyone can select" on kano_responses
  for select to anon using (true);
create policy "anyone can delete" on kano_responses
  for delete to anon using (true);

alter publication supabase_realtime add table kano_responses;
```

3. **Project Settings → API**에서 **Project URL**과 **anon public** 키를 복사.

> 응답값 인코딩: 0=마음에 든다, 1=당연하다, 2=별 느낌 없다, 3=어쩔 수 없다, 4=마음에 들지 않는다

---

## 2단계 · config.js 입력

```js
const SUPABASE_URL  = "https://abcd1234.supabase.co";  // 본인 Project URL
const SUPABASE_ANON = "eyJhbGciOi...";                 // 본인 anon public 키
const SESSION_CODE  = "2026-1";  // 강의/회차 구분용
```

---

## 3단계 · Vercel 배포

이 폴더(`kano-app`)를 통째로 배포합니다 (빌드 설정 없음, 정적 파일).

- **GitHub + Vercel**: 폴더를 저장소로 push → Vercel에서 Import → Framework `Other` → Deploy
- **CLI**: `cd kano-app && vercel --prod`

배포 후 주소:
- 학생용: `https://(프로젝트).vercel.app/`
- 교수용: `https://(프로젝트).vercel.app/dashboard.html`

---

## 4단계 · QR 만들기

배포된 사이트에서 `/qr.html`을 열고 학생용 주소를 입력하면 QR이 생성됩니다.

---

## 수업 진행 순서

1. 강의 화면에 **dashboard.html** 띄우기
2. 학생들에게 **QR/링크** 안내 → 각 기능마다 "있을 때"·"없을 때" 응답 후 제출
3. 제출될 때마다 대시보드의 분류표·산점도가 **자동 갱신**
4. 산점도 사분면(매력적/일원적/당연적/무관심)과 분류표로 **품질유형 해석**
5. 만족계수·불만족계수로 **기능 개발 우선순위** 논의

## 결과 읽는 법 (자료 8~10단계 연계)

- **당연적**(우하단): 없으면 불만 큼 → 반드시 안정 제공 (예: 환불 안내, 상담원 연결)
- **일원적**(우상단): 잘할수록 만족↑ → 성능·정확도 개선 (예: 비교 추천, 응답 속도)
- **매력적**(좌상단): 있으면 감동 → 차별화 요소 (예: 맞춤 일정, 개인화 추천)
- **무관심**(좌하단): 영향 작음 → 후순위
- **역품질**: 제공할수록 불만 → 제거하거나 선택형 (예: 과도한 추천 메시지)

> 계수 공식: 만족계수=(A+O)/(A+O+M+I), 불만족계수=(O+M)/(A+O+M+I)를 음수로 표기. 분모에서 역품질(R)·의심(Q)은 제외합니다.

## 점검

- 미리 보려면 대시보드의 **"예시 데이터 30명 추가"** 사용.
- 수업 후 **"이 세션 응답 전체 삭제"**로 비움. 회차 변경은 `SESSION_CODE`만 수정.

# REVOLV — Phase 3 글로벌 리더보드 설계 (T-29)
> 작성: 2026-07-11 | PM/개발 | 상태: **설계 확정 완료 — 구현 착수 가능**
> ✅ **부정방지 방침 확정(7/11)**: 콤보 무제한 구조상 고정 비율 상수는 부적합하다고 판단, 채택하지 않음. MVP는 절대 상한(score cap) + RPC 전용 쓰기만 적용, 정교화는 실제 리더보드 데이터 축적 후(Phase 3.5) 진행. (6번 섹션 참조)
> ✅ **저장 정책 확정(7/11)**: 게스트당 최고 기록만(A안)
> ✅ **Supabase 소유 계정 확정(7/11)**: 기존 애드센스 승인 개인 계정(GA4/AdMob과 동일)
> ✅ **공개 오픈 게이팅 확정(7/11)**: 백엔드 완성 즉시 공개(A안) — 나머지 트리거 조건 대기 없음

확정된 설계 결정 두 가지를 기준으로 Supabase 백엔드 스키마와 프론트엔드 연동을 설계합니다.

- **닉네임**: 1회 입력 후 localStorage 저장 (게스트 우선, Spherun식 마찰 최소화)
- **노출 범위**: 상위 100 + 내 순위 별도 표시

---

## 1. 아키텍처 개요

REVOLV는 단일 HTML 파일 + 바닐라 JS 구조를 유지해야 하므로, Supabase는 **SDK 없이 REST 엔드포인트(fetch)** 로만 연동합니다. 외부 스크립트 의존을 늘리지 않는 것이 이 프로젝트의 핵심 원칙과 맞습니다.

```
[브라우저 게임]  --(fetch, anon key)-->  [Supabase REST API]  -->  [Postgres + RLS]
     |                                                                    |
  게임오버 시 점수 제출 (INSERT)                              row-level security로
  리더보드 열람 시 상위 100 조회 (SELECT)                     읽기=전체허용/쓰기=검증
```

- **인증 없음**: 게스트 플레이가 기본이므로 Supabase Auth를 쓰지 않습니다. anon key만 사용.
- **닉네임 = 표시용 식별자**: 계정이 아니므로 중복 허용. 대신 클라이언트가 발급한 `player_id`(UUID, localStorage 저장)로 "내 기록"을 추적합니다.

---

## 2. 데이터베이스 스키마

### 테이블: `scores`

| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | `bigint` (identity) | PK |
| `player_id` | `uuid` | 클라이언트 발급, "내 순위" 추적용 (localStorage 저장) |
| `nickname` | `text` | 표시용 닉네임 (2~12자) |
| `score` | `integer` | 최고 점수 |
| `play_time` | `integer` | 해당 기록의 플레이 시간(초), 부정 방지 참고용 |
| `platform` | `text` | 'mobile' / 'pc' (분석 겸용) |
| `created_at` | `timestamptz` | 서버 시각 (default now()) |

### 설계 노트
- **"내 최고 기록만 유지" vs "모든 판 기록"**: 리더보드는 유저당 1행(최고점)만 노출하는 게 깔끔합니다. 두 가지 방법:
  - (A) `player_id`에 UNIQUE 제약 + upsert (기존보다 높을 때만 갱신) — **✅ 확정 채택(7/11)**
  - (B) 모든 판을 INSERT하고 조회 시 `DISTINCT ON (player_id)` — 데이터 누적 빠름 (미채택)
  - → 사유: 행 수가 유저 수만큼만 늘어나 무료 티어에 유리, 리더보드 쿼리 단순, "상위100+내순위" UX와 부합. 히스토리(성장그래프) 기능은 현재 로드맵에 없어 (B)의 이점이 불필요.

---

## 3. SQL — Supabase 대시보드 SQL Editor에 그대로 실행

```sql
-- 1) 테이블 생성
create table if not exists public.scores (
  id          bigint generated always as identity primary key,
  player_id   uuid not null unique,
  nickname    text not null check (char_length(nickname) between 1 and 12),
  score       integer not null check (score >= 0 and score <= 10000000),
  play_time   integer not null default 0 check (play_time >= 0),
  platform    text default 'unknown',
  created_at  timestamptz not null default now(),
  updated_at  timestamptz not null default now()
);

-- 2) 리더보드 조회 성능용 인덱스 (점수 내림차순)
create index if not exists scores_score_desc_idx on public.scores (score desc);

-- 3) RLS 활성화
alter table public.scores enable row level security;

-- 4) 읽기 정책: 누구나 리더보드 조회 가능
create policy "public read"
  on public.scores for select
  using (true);

-- 4-1) Data API(REST) 노출을 위한 명시적 GRANT
-- ⚠️ 2026/4월 이후 신규 Supabase 프로젝트는 "테이블 자동 노출"이 기본 꺼짐 상태로 바뀜.
--    RLS만으로는 부족하고, 아래처럼 명시적으로 권한을 줘야 anon 키로 조회가 가능함.
--    (프로젝트 생성 시 "Automatically expose new tables and functions" 옵션을 켰든 껐든,
--     이 GRANT문을 실행해두면 항상 정상 작동합니다.)
grant select on public.scores to anon;

-- 5) 최고점 upsert 함수 (기존보다 높을 때만 갱신, 원자적 처리)
-- ✅ MVP 방침(7/11 확정): 콤보가 무제한이라 점수가 시간에 비례하지 않고
--    장기 플레이일수록 콤보 누적으로 점수가 비선형(제곱에 가깝게) 증가함.
--    고정된 "초당 최대점수" 상수는 정확한 기준이 될 수 없다고 판단해 채택하지 않음.
--    → MVP는 절대 상한(score cap)만 적용. 정교한 부정방지는 실제 리더보드 데이터가
--      쌓인 뒤(Phase 3.5)에 관측된 상위권 분포를 근거로 재설계.
create or replace function public.submit_score(
  p_player_id uuid,
  p_nickname  text,
  p_score     integer,
  p_play_time integer,
  p_platform  text
) returns void
language plpgsql
security definer
set search_path = public
as $$
begin
  -- 서버측 방어: 닉네임 길이/점수 범위 재검증
  if char_length(p_nickname) < 1 or char_length(p_nickname) > 12 then
    raise exception 'invalid nickname';
  end if;
  if p_score < 0 or p_score > 10000000 then
    raise exception 'invalid score';
  end if;
  if p_play_time < 0 then
    raise exception 'invalid play_time';
  end if;

  insert into public.scores as s (player_id, nickname, score, play_time, platform)
  values (p_player_id, p_nickname, p_score, p_play_time, p_platform)
  on conflict (player_id) do update
    set score      = greatest(s.score, excluded.score),
        nickname   = excluded.nickname,
        play_time  = case when excluded.score > s.score then excluded.play_time else s.play_time end,
        platform   = excluded.platform,
        updated_at = now()
    where excluded.score > s.score;  -- 신기록일 때만 갱신
end;
$$;

-- 6) 함수 실행 권한을 anon(익명)에게 부여
grant execute on function public.submit_score(uuid, text, integer, integer, text) to anon;

-- 7) 내 순위 조회 함수 (상위 100 밖이어도 내 순위를 알 수 있게)
create or replace function public.my_rank(p_player_id uuid)
returns table(rank bigint, score integer, nickname text)
language sql
stable
as $$
  select r.rank, r.score, r.nickname
  from (
    select player_id, score, nickname,
           rank() over (order by score desc) as rank
    from public.scores
  ) r
  where r.player_id = p_player_id;
$$;

grant execute on function public.my_rank(uuid) to anon;
```

### 왜 RPC 함수(submit_score)를 쓰나
- 클라이언트가 `scores` 테이블에 직접 INSERT 권한을 갖게 하면 임의 점수 주입이 쉽습니다.
- INSERT 정책을 열지 않고 `security definer` 함수로만 쓰기를 허용하면, **검증 로직(점수 상한, 닉네임 길이)을 서버에서 강제**할 수 있습니다.
- upsert의 "신기록일 때만 갱신"도 DB 트랜잭션 안에서 원자적으로 처리되어 경쟁 조건이 없습니다.

---

## 4. 프론트엔드 연동 (fetch, SDK 없음)

### 4-1. 상수 (index.html 상단 스크립트에 추가)
```js
const SB_URL  = 'https://YOUR-PROJECT.supabase.co';
const SB_ANON = 'YOUR-ANON-PUBLIC-KEY';  // anon key는 공개돼도 되는 키(RLS로 보호)
```
> anon key는 클라이언트 노출이 전제된 공개 키입니다. RLS와 RPC 검증이 실제 보안 경계이므로 여기 하드코딩해도 됩니다. **service_role 키는 절대 클라이언트에 넣지 않습니다.**

### 4-2. player_id / nickname 관리
```js
function getPlayerId(){
  let id = localStorage.getItem('revolv_pid');
  if (!id){ id = crypto.randomUUID(); localStorage.setItem('revolv_pid', id); }
  return id;
}
function getNickname(){ return localStorage.getItem('revolv_nick') || ''; }
function setNickname(n){ localStorage.setItem('revolv_nick', n); }
```

### 4-3. 점수 제출 (게임오버 시)
```js
async function submitScore(score, playTime){
  const nick = getNickname();
  if (!nick) return;              // 닉네임 없으면 제출 스킵 (등록 유도 UI에서 처리)
  try {
    await fetch(SB_URL + '/rest/v1/rpc/submit_score', {
      method: 'POST',
      headers: {
        'apikey': SB_ANON,
        'Authorization': 'Bearer ' + SB_ANON,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        p_player_id: getPlayerId(),
        p_nickname:  nick,
        p_score:     score,
        p_play_time: playTime,
        p_platform:  isMobile ? 'mobile' : 'pc',
      }),
    });
  } catch(e){ /* 오프라인 등은 조용히 무시 */ }
}
```

### 4-4. 리더보드 조회 (상위 100 + 내 순위)
```js
async function fetchLeaderboard(){
  const top = await fetch(
    SB_URL + '/rest/v1/scores?select=nickname,score&order=score.desc&limit=100',
    { headers: { 'apikey': SB_ANON, 'Authorization': 'Bearer ' + SB_ANON } }
  ).then(r => r.json());

  const mine = await fetch(
    SB_URL + '/rest/v1/rpc/my_rank',
    { method: 'POST',
      headers: { 'apikey': SB_ANON, 'Authorization': 'Bearer ' + SB_ANON, 'Content-Type': 'application/json' },
      body: JSON.stringify({ p_player_id: getPlayerId() }) }
  ).then(r => r.json());

  return { top, mine: mine[0] || null };
}
```

---

## 5. UI 흐름

1. **첫 신기록 달성 시** 게임오버 화면에 닉네임 입력 필드 노출 → 입력하면 localStorage 저장 + 제출
2. **이후 게임오버** 마다 저장된 닉네임으로 자동 제출 (신기록일 때만 서버가 갱신)
3. **홈 화면에 리더보드 버튼** 추가 → 상위 100 리스트 + 최하단에 "내 순위: 142위 / 3,200점" 고정 표시
4. 닉네임은 게임오버 화면 또는 리더보드에서 언제든 변경 가능

### itch.io 채널 주의
- GA4와 마찬가지로 itch iframe 샌드박스가 외부 fetch(cross-origin)를 어떻게 처리하는지 확인 필요.
- Supabase는 CORS 허용 도메인 설정이 되어 있어 GA4(쿠키/리퍼러 기반)와는 상황이 다를 수 있으나, itch 샌드박스에서 실제 동작하는지 **배포 전 테스트 필수**.
- 최악의 경우 itch에서는 리더보드가 "읽기 전용" 또는 비활성화될 수 있음 — Netlify가 공식 채널이므로 치명적이지 않음.

---

## 6. 부정 방지 (MVP: 절대 상한만 / 정교화는 Phase 3.5로 연기)

**✅ 결정 확정(7/11)**: "점수 대비 플레이시간" 같은 고정 상수 기반 검증은 채택하지 않기로 확정. 대신 절대 상한 + RPC 전용 쓰기 구조만으로 MVP를 시작하고, 실제 리더보드 데이터가 쌓인 뒤 정교화한다.

**왜 고정 비율 상수를 쓰지 않기로 했나**: REVOLV의 콤보(`S.combo`)는 4초 안에 계속 스파크를 주우면 리셋 없이 무한히 누적됩니다. 점수 획득량이 `10 × 콤보`(피버 중 ×2)이므로, 오래·잘 플레이할수록 점수가 시간에 비례하는 게 아니라 콤보 누적 효과로 거의 제곱에 가깝게 늘어납니다. 즉 "1초당 최대 몇 점"이라는 고정 상수 하나로는 짧은 판과 긴 판 모두에 맞는 정확한 기준을 만들 수 없습니다 — 너무 낮게 잡으면 실력자의 정당한 장기 플레이가 오탐으로 걸리고, 너무 높게 잡으면 부정 방지 효과가 사실상 없어집니다. 지금 데이터(GA4 소규모 표본)로 억지로 상수를 정하면 잘못된 확신만 갖게 될 뿐이라 판단해 채택하지 않았습니다.

**MVP에서 적용하는 것**:
- **점수 절대 상한** (`score <= 10000000`): 명백한 조작값(예: 999999999) 즉시 차단
- **RPC 전용 쓰기**: `scores` 테이블에 직접 INSERT 권한을 주지 않고, `submit_score` 함수로만 쓰기 허용 (3번 섹션) — "누구나 아무 값이나 방명록에 써넣는" 최악의 시나리오는 이것만으로도 차단됨
- service_role 키 클라이언트 미노출, RLS로 직접 INSERT/UPDATE/DELETE 차단

**Phase 3.5로 미룬 것**:
- 리더보드가 실제로 켜지고 상위권 점수 분포가 쌓이면, 그 실측 데이터를 근거로 이상치 탐지 기준을 다시 설계
- 검토했으나 이번엔 채택하지 않은 대안: (a) 게임 로직에서 콤보 자체에 상한을 걸어 점수 증가를 선형으로 만들기 — 백엔드 범위를 넘어서는 게임 밸런싱 변경이라 별도 논의 필요, (b) 구간별(비선형) 상한 — 지금 단계엔 과한 엔지니어링

---

## 7. 계정 소유 확정 (7/11)
- **Supabase 프로젝트는 기존 애드센스 승인 개인 계정(GA4/AdMob 소유 계정)으로 생성** — 공개 채널(playrevolv)과 자산 소유 계정 분리 원칙 적용
- 프로젝트 생성 시 이름 예시: `revolv-leaderboard` 또는 `playrevolv-prod`

---

## 8. 다음 실행 순서

1. **[사용자]** Supabase 프로젝트 생성 (기존 애드센스 승인 계정으로 소유 — 확정됨)
2. **[사용자]** 위 SQL을 SQL Editor에 실행
3. **[사용자]** Project Settings → API에서 `URL`과 `anon public key` 복사해 전달
4. **[PM/개발]** index.html에 연동 코드 + 닉네임 입력 UI + 리더보드 화면 구현
5. **[PM/개발]** 문법 검증 + 시나리오 시뮬레이션 후 완성본 전달
6. **[사용자]** push + Netlify 배포, itch에서 fetch 동작 테스트
7. **공개 오픈 시점**: ✅ 확정(7/11) — 백엔드 완성 즉시 공개(A안). Phase 3 트리거(3개 중 1개 충족) 성립 근거로, 나머지 2조건 충족을 기다리지 않고 리더보드 UI를 유저에게 바로 노출.

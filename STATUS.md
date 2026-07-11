# REVOLV 프로젝트 현황판 (STATUS.md)
> 최종 갱신: 2026-07-09 | 갱신 주체: PM (사용자 유저 피드백 반영 + 홍보팀 7/9 저녁 보고 반영)
> 규칙: 모든 작업 상태는 이 파일이 기준. "완료" = 코드 작성 + push + 배포 확인까지 끝난 상태.

## 로드맵 위치
- Phase 1 (웹 배포) ✅ 완료
- **Phase 2 (검증 & 폴리시) ← 현재**
- Phase 3 (글로벌 랭킹) — 트리거 대기: 주간 game_start 150+ / 주간 재방문 10+ / 리더보드 피드백 3건 중 1개 충족 시. Measurement Protocol 재검토 예정. **참고 사례: Spherun(리더보드+게스트→계정 전환 UX)**
- Phase 4 (안드로이드 앱 + AdMob) — Phase 3 이후
- Phase 5 (iOS) — 이후

## ✅ 완료 (검증됨)
| ID | 작업 | 비고 |
|---|---|---|
| T-01~T-16, T-23 | (이전 갱신 참조) | 웹 배포·GA4·SEO·itch 런칭·AI Disclosure 등 전부 완료 |
| T-24 | 유저 피드백 3건 반영 | ①가로화면 HUD 재배치 ②일시정지 버튼(P/Esc, 탭전환 자동정지) ③튜토리얼/일시정지 재개 시 3-2-1 카운트다운 — 배포 완료 |
| T-25 | 가로모드 스킬 버튼 반응형 크기 | clamp(54~72px, 14vh) 적용, 기기별 검증 완료 |
| T-26 | Devlog #2 게시 | "Player feedback round 1" — 피드백 반영 스토리, PM 작성 |
| T-19 | Instagram 4번째 피드 | 챌린지형("1만점 슈퍼노바 인증 유도") — 조회→관여 전환 시도 |

## 🔶 진행중
| ID | 작업 | 담당 | 비고 |
|---|---|---|---|
| T-18 | Reddit 카르마 빌드업 | 홍보팀 | Spherun 등 실플레이 피드백 댓글 진행중, 20~30 도달 시 r/WebGames 게시(문구 준비완료) |
| 관찰 | T-19 챌린지 반응 대조 | 홍보팀 | 댓글/DM 점수인증, itch referrer·Plays 증감 — 내일 확인 |

## 📌 경쟁 벤치마킹
- **Spherun** (3D 멀티 브라우저 게임): 리더보드·게스트→계정 전환 UX 보유, **단 데스크톱 전용(모바일 미지원)**
- → REVOLV의 "모바일+PC 동시지원, 원버튼" 포지셔닝이 상대적 강점 — r/WebGames 게시 및 향후 피드 소재에 강조 근거로 활용
- → Phase 3 랭킹 UX 설계 시 Spherun 참고 사례로 활용 예정

## 채널별 지표 체계 (T-20)
| 채널 | 추적 방식 |
|---|---|
| netlify / GitHub | GA4 (정밀) |
| itch.io | itch 자체 대시보드 (Views/Plays/Ratings/Referrer) |

### itch 채널 baseline (7/8 기준) — 다음 갱신: 챌린지 피드 효과 대조 후
- Views 17 / Browser Plays ~5 / Ratings 0 / Referrer: FB 2, IG 1, itch profile 1

## 📋 예정
| ID | 작업 | 시점 |
|---|---|---|
| T-18 | r/WebGames 게시 (Spherun 대비 강점 앵글 반영) | 카르마 20~30 도달 시 |
| T-19-2 | Instagram 5번째 피드 | 2~3일 후, 챌린지 반응 좋으면 "이번 주 최고점" 연재형 검토 |
| T-27 | Long-form devlog (절차생성 BGM 원리 / 난이도 곡선 삼질기) | 소재 보유, 업데이트 확정 시 게시 |
| T-20 | 채널/날짜/링크 기록 상시 가동 | 상시 |

## 🚫 종결/보류
| ID | 작업 | 사유 |
|---|---|---|
| T-17 | 디시인사이드 개발 비하인드 글 | 보류(재검토 조건부) |
| — | Measurement Protocol | 보류, Phase 3 재검토 |
| T-21, T-22 | netlify.toml / 커스텀 도메인 | 각각 불필요 종결 / Phase 4 재검토 |

## 관찰 지표
- netlify/GitHub: 일일 game_start / 재방문 / score 분포 (GA4)
- itch: Views / Plays / Ratings / Referrer (대시보드)
- 리더보드 언급 피드백: **0/3** (챌린지 피드 반응으로 변동 가능성 주시)

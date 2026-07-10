# REVOLV 프로젝트 현황판 (STATUS.md)
> 최종 갱신: 2026-07-08 | 갱신 주체: PM (홍보팀 7/8 저녁 보고 반영)
> 규칙: 모든 작업 상태는 이 파일이 기준. "완료" = 코드 작성 + push + 배포 확인까지 끝난 상태.

## 로드맵 위치
- Phase 1 (웹 배포) ✅ 완료
- **Phase 2 (검증 & 폴리시) ← 현재**
- Phase 3 (글로벌 랭킹) — 트리거 대기: 주간 game_start 150+ / 주간 재방문 10+ / 리더보드 피드백 3건 중 1개 충족 시. **착수 시 Measurement Protocol(itch 서버사이드 추적) 재검토 예정**
- Phase 4 (안드로이드 앱 + AdMob) — Phase 3 이후
- Phase 5 (iOS) — 이후

## ✅ 완료 (검증됨)
| ID | 작업 | 비고 |
|---|---|---|
| T-01 | 게임 전 기능 개발 | 스킬/이벤트/튜토리얼/i18n/BGM/iOS뷰포트 |
| T-02 | GitHub 저장소 + Pages | hangoverun.github.io/revolv |
| T-03 | Netlify 배포 | playrevolv.netlify.app (공식 URL) |
| T-04 | itch.io 런칭 | playrevolv.itch.io/revolv + devlog 1건 |
| T-05 | Instagram 개설 + 피드 3건 | bio 링크 = itch, referrer 유입 실측 확인(7/8) |
| T-06 | 피드백 채널 노출 | 게임오버/도움말에 Instagram·이메일 |
| T-07 | privacy.html 배포 | 게임 내 링크 연결 |
| T-08 | GA4 + 커스텀 이벤트 + platform | 커스텀 정의 5/5, **netlify/GitHub 채널만 유효** (itch는 T-14 참조) |
| T-09 | SEO 메타/OG커버/sitemap/robots 배포 | 구글 서치콘솔 sitemap 제출 "성공" |
| T-10 | sitemap `<script/>` 이슈 | 브라우저 렌더링 산물로 판명 → 조치 불필요 종결 |
| T-11 | 3채널 버전 통일 | GitHub=netlify=itch 동일 빌드 |
| T-12 | Naver 소유확인 | 완료 |
| T-13 | Naver sitemap 제출 | 완료 (7/8) — Google·Naver 양쪽 SEO 세팅 마무리 |
| T-15 | AI Disclosure 표기 검토 | 회신 완료 — 절차생성 BGM은 Sound 항목 비대상, 나머지는 정직 표기 원칙 유지 |
| T-16 | 커버 GIF 장면 확인 | 회신 완료 — 코어 루프(2~4초) 위주, 게임오버/메뉴는 부적합 판정 |
| T-23 | 개발자 기기 GA4 제외 (`?dev`) | netlify/GitHub 배포 완료·유효. **itch는 구조상 적용 무의미**(T-14로 확정, 추가조치 불필요) |

## 🔶 진행중
| ID | 작업 | 담당 | 다음 행동 |
|---|---|---|---|
| (없음 — 배포 대기 항목 전부 해소) | | | |

## 📌 T-14 결론 (itch GA4 추적 — 구조적 한계 확정)
- itch는 게임을 서드파티 CDN iframe(샌드박스)으로 서빙 → 쿠키/저장소 차단으로 GA4 gtag 전송 불가. **REVOLV 코드 문제 아님, itch 임베드 구조의 근본 한계**
- **대체 측정 체계 확정(PM 승인)**: itch 채널은 GA4 대신 **itch 자체 대시보드**(Views / Browser Plays / Ratings / Referrers)로 측정
- 완전 해결책(Measurement Protocol, 서버사이드 전송)은 **Phase 3 착수 시 재검토** — 그때 랭킹용 서버(Supabase)를 두게 되므로 같은 김에 검토

## 채널별 지표 체계 (T-20)
| 채널 | 추적 방식 | 상태 |
|---|---|---|
| netlify / GitHub | GA4 (platform 파라미터, 커스텀 이벤트 전부) | 정밀 측정 가능 |
| itch.io | itch 자체 대시보드 (Views/Plays/Ratings/Referrer) | GA4 미집계, 별도 확인 필수 |

### itch 채널 baseline (7/8 기준, 최근 30일)
- Views: 17 / Browser Plays: 약 5 (7/7 런칭 이후) / Ratings: 0
- Referrer: Facebook 2, Instagram 1, itch 프로필 1 → **Instagram bio→itch 유입 실측 확인됨**
- 참고: Views 17 대비 Plays 5 (Run game 이탈 관찰, 표본 소 — 재점검 필요)
- ⚠️ itch Plays와 GA4 game_start는 집계 기준이 달라 **직접 비교/합산 금지**, 채널별 개별 추이로만 판단

## 📋 예정 (홍보팀 파이프라인)
| ID | 작업 | 시점 |
|---|---|---|
| T-18 | Reddit 카르마 빌드업(일 2~3 댓글) → 카르마 20~30 도달 시 r/WebGames 게시 | 진행중 |
| T-19 | Instagram 4번째 피드 | 내일~모레 (3번째로부터 2~3일 간격) |
| T-20 | 채널/날짜/링크 기록 → platform 매칭 (itch는 예외 처리, 대시보드로 별도 확인) | 상시 |
| itch 대시보드 정기 확인 | Browser Plays·Ratings·Referrer 추이 관찰 | 상시 |

## 🚫 종결/보류
| ID | 작업 | 사유 |
|---|---|---|
| T-21 | netlify.toml 적용 | T-10 판명으로 불필요 → 미적용 종결 |
| T-22 | 커스텀 도메인 구매 | Phase 4 근처에서 재검토 |
| T-17 | 디시인사이드 개발 비하인드 글 | **보류로 전환**(완전 제외 아님) — 리스크 대비 실익 낮음 판단, 게임 완성도·반응 개선 시 재검토 조건부. 해당 리소스는 Reddit/Instagram에 집중 |
| — | Measurement Protocol (itch 서버사이드 추적) | 검증기 단계 과투자 판단, 보류. **Phase 3 착수 시 재검토** |

## 관찰 지표 (매일)
- netlify/GitHub: 일일 game_start / 재방문 사용자 / score 분포 (GA4)
- itch: Views / Browser Plays / Ratings / Referrer (itch 대시보드, 별도 확인)
- 리더보드 언급 피드백 카운트: **0/3**

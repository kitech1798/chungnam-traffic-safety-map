# 충남 교통안전 개선 지도 — 어르신 보행부터

⚠ 알려진 한계: 전국 버스정류장 파일(15067528)에 계룡시 등재분이 없어 계룡 현장은 정류장 결합 0 · 교통카드는 2025.8 1개월 합성자료(상대 비교용) · 정류장명–좌표 매칭은 동명 오매칭 가능

라이브: https://kitech1798.github.io/chungnam-traffic-safety-map/ (GitHub Pages, `build\deploy_pages.ps1`) · 검증 `build\verify_live.py` · 민감도 `build\sensitivity.py`

충남 15개 시군의 사고다발지역 7유형(2021~2024, 원본 549건 → 현장 354곳 병합) × 정류장 카드 승하차(경로카드 비중) × 시군별 고령보행 통계(2022~2024)를 결합해, **시군별 우선 개선 후보(강건 후보군 = 세 가중식 모두 상위 50, 조치 예시 포함)와 대응 유형(세 축: A 탐지 공백 / B 중증·반복 현장 물량 / C 고령보행 피해율)별 개선 필요사항**을 보여주는 정적 웹 화면. 2026 충청남도 데이터 분석 아이디어 공모전 출품 산출물.

## 화면 5탭
| 탭 | 내용 |
|---|---|
| 지도 | 시군 색 지표 6종 · 현장 354 점(크기=점수) · 보행노인 다발지·링크 위험지역·경로카드 상위 정류장 층 · 시군 클릭 카드(대응 유형·우선 후보·필요사항) · 현장 클릭 팝업(조치) |
| 우선 개선 후보 | 시군·유형·사망·정류장 필터, 점수(12·3·1 가중) 순, 조치 예시·난이도, CSV |
| 권역별 개선 | 15시군 카드: 유형·규칙 근거·개선 필요사항·우선 지점 3·사각 시군 정류장 후보·후속 확인 5항목 체크·메모(브라우저 로컬) |
| 통계 | 발견 3건 · 연도별 2020~2024 · 15시군 전체 표 · 산식 |
| 목록·출력 | 시군 CSV · 현장 전체 CSV · 인쇄 보고서 |

## 데이터(전량 공개)
- 사고다발지역 7유형·링크 위험지역 — 한국도로교통공단 TAAS: koroad opendata(oldman·child·drunk·pedstrians, sido 44/gugun 3자리) + data.go.kr B552061(lg·bicycle·violation·risklink)
- 지자체별 대상 교통사고 통계 — data.go.kr 15056770 (`B552061/lgStat`, **siDo=1600**, guGun 1602~1624 — 행안부 44 코드 아님)
- 정류장별 대중교통 이용량(교통카드 합성데이터) — `apis.data.go.kr/1613000/TripVolumebyStop`(2025.8, 16구군 21.8만 행) · 정류장명 `BusStop/getBusStop`(파라미터 `opr_ymd`) → 전국 버스정류장 위치(15067528)와 이름 조인(4,449곳)
- 행정안전부 행정동별 연령별 인구 — 15097972 (2026-05-31, 65세 이상 → 인구당 지표 **참고치**)
- 행정동 경계 — 통계청 SGIS 원출처, GitHub vuski/admdongkor(MIT) 재배포본(표시용)

## 재현
```powershell
python -X utf8 acquire/fetch_lgstat.py                     # 시군 통계
python -X utf8 acquire/fetch_taas_all.py                   # 다발지역 7유형+위험지역
python -X utf8 acquire/fetch_transit_chungnam.py --ym 202508   # 교통카드 + 정류장 좌표 (--skip-trips 로 좌표만)
python -X utf8 acquire/get_boundary_sgg.py                 # 시군 경계
python -X utf8 build/build_data.py                         # app_data.json (시군 통계·다발지 대조)
python -X utf8 build/build_sites.py                        # sites.json (현장 병합·정류장 결합·조치 예시·대응 유형)
python -X utf8 build/build_selfcontained.py                # dist/index.html (Leaflet·데이터 인라인, BOM)
python -X utf8 -m pytest tests -q                          # 18개 검사
python -X utf8 build/capture_shots.py                      # build/shots/*.png
python -X utf8 build/summary_for_proposal.py               # 기획서 숫자 출력(수기 입력 방지)
```
- 인증키 `acquire/.key*`(gitignore). 원자료 대용량 2종(정류장 전국·교통카드)은 gitignore.
- 병합 규칙·EPDO·조치 유형은 천안 C-TIPS 회차에서 실측으로 확정한 규칙을 **재구현**(천안 파일은 읽기만).

## 하지 않는 것
사고 예측 · 다발지역 밖 사망 위치 특정 · 인과·정책 효과 % · 시군 종합 순위·안전등급 · 소관 부서 지정

## 배포
`dist/index.html` 한 파일. GitHub Pages(새 public repo, index.html로 복사) 또는 USB·메일 전달. 배경지도(Esri)는 인터넷 필요, 표·수치·경계는 오프라인 동작.

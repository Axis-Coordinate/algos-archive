# Perpetual Arbitrage 전략 분석 (algos.org #6 + #7)

Source HTML: [`html/perpetual-arbitrage-analysis.html`](../html/perpetual-arbitrage-analysis.html)

# Perpetual Arbitrage 전략 분석 (algos.org #6 + #7)

> 출처: Small Trader Alpha #6 (2024-07-09) + #7 (2024-10-01)  
> 분석일: 2026-05-20  
> 대상: Axis/Statra 크로스 익스체인지 아비트라지 팀

---

## 크롤링 상태

| 글 | 상태 | 누락된 섹션 |
| --- | --- | --- |
| #6 (Perpetual Arbitrage) | **부분 크롤링** (~60%) | Checking For Opportunities, Taker Backtest, **전체 Part 3 (Advanced Trading)** |
| #7 (Perpetual Arbitrage Pt. 2) | **거의 미크롤링** (~5%) | Rust 프로덕션 코드 전체, 고급 기법, Appendix |

**#7에서 누락된 핵심 콘텐츠**: Rust perpetual arbitrage scanner 전체 코드, barter-rs 연동, 프로덕션 구현 상세. 원본 URL에서 재크롤링 필요.

---

## Part 1: 기본 개념 (#6에서 추출)

### 1.1 Perpetual Arbitrage 개요

**핵심**: 동일 자산의 크로스 익스체인지 무기한 선물(perpetual) 간 스프레드 수렴에 베팅하는 **통계적 아비트라지(statistical arbitrage)**. 순수 아비트라지가 아님.

- 페어 트레이딩과 유사: 진입 임계값 + 청산 임계값 (보통 평균)
- 기계적 힘(mechanical forces)이 자산을 다시 수렴시킴 (순수 통계적 트레이드와 다른 점)
- 예시: SOL/USD OKX vs SOL/USDC Binance perp

### 1.2 가격 차이의 3가지 원인

| 원인 | 설명 | 대응 |
| --- | --- | --- |
| **스테이블코인 차이** | USDC/USDT/USD 가격 차이 | 스테이블코인 가격 추적/정규화 |
| **구조적 차이** | 거래소별 고유 플로우, 인덱스 구성 방식 | 경험적 분석 필요 |
| **계약 차이** | 펀딩 주기, 마진 타입, 결제 방식 | 수동 조사 + 정규화 |

### 1.3 스테이블코인 베이시스 정규화

**필수 사항:**  
- 거래소별 스테이블코인 시장 가격으로 정규화 (글로벌 가격 X, 거래소 로컬 가격 O)  
- 디페깅 시나리오 대비 시스템 off 스위치 필수  
- USDC 디페깅 사례: Coinbase에서 $0.93까지 하락, 다른 거래소는 몇 센트만 하락  
- KuCoin도 다른 거래소 대비 큰 폭 디페깅

**주의사항:**  
- 일부 거래소는 -USD 표기지만 실제로는 USDC/USDT 결제  
- 같은 거래소에 USDC/USDT/USD perp가 모두 존재할 수 있음  
- USD perp가 coin-margined인 경우 있음 (다른 펀딩 인터벌 가능)  
- 동일 결제/펀딩 사양인 경우에만 교차 비율로 스테이블코인 가격 추정 가능

### 1.4 계약 정규화 (Contract Normalization)

Binance perp 가격을 다른 거래소(Bybit, OKX, Hyperliquid 등)로 변환하는 문제.

**단순 접근법의 한계:**  
- Binance perp 가격 직접 사용 -> 특히 DEX처럼 짧은 펀딩 주기(시간 단위)에서 부정확  
- perp 가격은 "하향 계단" 패턴으로 스팟에 수렴 -> 펀딩 결제 시 점프  
- 스팟 가격만 사용 -> 불균형 포지션 (언더퍼포밍 코인 롱, 급등 코인 숏)

**최선의 접근법:**

```
1. Binance perp 가격 가져오기
2. 미수 펀딩 결제(accrued but unpaid funding) 차감
3. 금리 차이(rate differences) 조정
4. 대상 거래소의 미수 쿠폰(accrued coupon) 차감
5. 펀딩 결제 재계산
```

**정규화의 어려움:**  
1. 거래소마다 다른 인덱스 -> 변동성 큰 시장에서 괴리 발생  
2. 펀딩 결제 상한(caps) 거래소마다 다름  
3. 펀딩 레이트 빈번하게 변경 -> 비선형 공식 때문에 변환에 영향  
4. 실무 구현 이슈: 미수 펀딩 결정, 금리 차이 조정

**신뢰도 스코어(Confidence Score) 모델:**  
- 두 perp의 유사성이 높으면 -> 신뢰도 높음 -> 다이버전스 시 더 큰 사이징  
- 포함해야 할 컴포넌트: **펀딩 인터벌, 인덱스 가격 공식, 펀딩 상한, 발생 공식(accrual formula)**

**베이시스 변동성(Basis Volatility):**  
- 인덱스가 전체 시장 볼륨의 큰 비중을 커버할수록 안정적  
- 예: Bybit+OKX+Binance 인덱스 -> 매우 안정 / 자체 스팟 마켓만 사용하는 소형 거래소 -> 불안정

### 1.5 진입/청산 타이밍

**기본 파라미터:**

| 파라미터 | 값 | 설명 |
| --- | --- | --- |
| 이동평균 | SMA 또는 EMA | lookback period 설정 |
| 진입 임계값 | 2 std dev | 스프레드의 z-score 기준 |
| 청산 임계값 | 평균 (z=0) 또는 그 이전 | 3번째 파라미터로 exit z-score 설정 가능 |

**SMA <-> EMA 변환:**  
- EMA의 반감기(half-life)와 SMA의 lookback period를 동등하게 환산 가능

**Mean Reversion 동학:**  
- 기대 평균 회귀 E[MR(t1, t2)]는 절대 z-score에 비례 (대략 선형, 실제로는 지수적 또는 플래토 가능)  
- 스프레드 변동성은 대략 상수로 가정 가능  
- **핵심 인사이트**: 스프레드가 회귀할수록 엣지 감소, 변동성은 동일 -> 리스크-리워드 악화  
- 평균 추정 오류(mean estimation error) 위험까지 고려하면 -> **평균까지 완전히 보유하는 것보다 약간 이전에 청산이 합리적**

**비용 민감도:**  
- 이 전략은 수수료/거래비용 대비 경쟁이 매우 치열  
- "몇 bps가 세상을 좌우" -> 거래비용 후 수익이 몇 bps 수준

---

## Part 2: 구현 (#6에서 추출)

### 2.1 파이프라인

```
1. Data Scraping       (.ipynb)
2. Data Pre-Processing (.ipynb)
3. Analysis            (.ipynb) -> 기본 기회 탐색
4. Analysis Cont.      (.ipynb) -> 백테스트
5. Expanded Analysis   (.ipynb) -> 레이턴시/거래비용 변동 분석
```

각 단계를 별도 .ipynb로 분리. 여러 연구자가 검증해야 하는 전략이면 .py 파일로 툴링 빌드.

### 2.2 데이터 소싱

**Tardis.dev** 사용 (히스토리컬 데이터).

**거래소 목록 및 티커 포맷:**

| 거래소 | Tardis 이름 | 티커 접미사 |
| --- | --- | --- |
| OKX | `okex-swap` | `-USDT-SWAP` |
| Bybit | `bybit` | `USDT` |
| dYdX | `dydx` | `-USD` |
| Gate.io | `gate-io-futures` | `_USDT` |
| Binance | `binance-futures` | `USDT` |

**데이터 타입:** `quotes`, `trades`, `book_snapshot_25`

**실시간 대안:** Rust로 WebSocket 피드 구독 -> 기회 스캔 -> 데이터셋 로깅

### 2.3 데이터 전처리

**TAQ (Trades & Quotes) 병합:**  
- 거래소별 TAQ 병합 (limit order 최적화 분석용)  
- 크로스 익스체인지 quotes 병합 (기회 탐색 + 백테스트용)  
- `pd.merge_asof`로 타임스탬프 기준 nearest merge  
- 출력: Parquet 포맷

**Quote 데이터 처리 규칙:**  
- Quote 값은 fill-forward 가능 (다음 quote까지 유효)  
- Trade 값은 fill-forward 불가  
- NaN 체크: trade 컬럼이 NaN이면 quote 업데이트, 아니면 trade 업데이트  
- Binance 실시간 업데이트 ~30ms 간격 -> sub-100ms/sub-30ms 주기에서 주의

**심볼 필터링:**  
- 최소 2개 이상의 거래소에 동시 상장된 심볼만 유효  
- 시가총액 기준 필터링 가능

### 2.4 [누락] Checking For Opportunities

크롤링 잘림. 단일 자산(LINK) 2개 거래소 간 아비트라지 스캔이 시작되는 시점에서 절단.

### 2.5 [누락] Taker Backtest

크롤링 잘림. 기본 taker 버전 백테스트 코드 전체 누락.

---

## Part 3: Advanced Trading (#6에서 목차만 존재, 본문 누락)

**#6 목차에 나열된 고급 주제들 (본문 미크롤링):**

| 섹션 | 설명 (목차 기반 추정) |
| --- | --- |
| Maker/Taker Execution | 메이커 주문으로 비용 절감 전략 |
| Volume & Lead-Lag | 거래량 기반 리드-래그 분석 |
| Latency & Message Ordering | 레이턴시 최적화, 메시지 순서 처리 |
| Mean Estimate Free Reversion | 평균 추정 없는 평균 회귀 기법 |
| Global Lead Lag | 글로벌 리드-래그 관계 |

---

## Part 4: Rust 프로덕션 코드 (#7 - 거의 전체 누락)

**#7에서 확인된 정보:**  
- Rust로 perpetual arbitrage scanner **전체 코드** 제공 (Appendix에 copy-paste 가능한 형태)  
- **barter-rs** 오픈소스 프로젝트 활용 (Rust용 CCXT 유사 라이브러리)  
- 셀프 도큐멘팅 코드 (주석 + 네이밍으로 설명)  
- 프로덕션 배포 가능 수준

**재크롤링 필요**: https://www.algos.org/p/small-trader-alpha-7-perpetual-arbitrage

---

## PNL 차트 분석

첨부된 PNL 이미지 (1코인, 2거래소, top-of-book 기준):

- **Absolute PnL**: 계단식 상승, 일관된 양의 수익 (~0.20 달러 규모, 기간 내)
- **Mid-price Spread**: 1.0 중심으로 oscillation, 가끔 1.01까지 스파이크
- 스프레드 비율이 1.0 = 두 perp 가격 동일, 1.0 초과 = 첫 번째 거래소가 더 비쌈
- "shape에 집중, 스케일링은 trivial" -> 코인 수/거래소 수 확장으로 선형 스케일링

---

## Axis/Statra 적용 시 고려사항

### 직접 관련 영역

| Axis 현재 전략 | 이 글과의 연관성 |
| --- | --- |
| Basis trading | Perp arb의 특수 케이스. 동일 개념, 다른 쌍 |
| Funding rate capture | 계약 정규화 시 펀딩 레이트가 핵심 요소 |
| Calendar spreads | 만기 기반 vs 무기한 - 유사한 mean reversion 동학 |
| Cash-and-carry | Spot-perp 버전과 perp-perp 버전의 차이 이해 필요 |

### 멀티 익스체인지 구현 체크리스트

```
1. 스테이블코인 베이시스 정규화
   - [ ] 거래소별 USDT/USDC/USD 가격 실시간 추적
   - [ ] 디페깅 감지 및 자동 시스템 정지 로직
   - [ ] Binance, OKX, Bybit, Coinone, Bitget, Kraken, Gate.io 각각 매핑

2. 계약 정규화
   - [ ] 거래소별 펀딩 인터벌 매핑 (8h vs 4h vs 1h)
   - [ ] 펀딩 레이트 상한(cap) 매핑
   - [ ] 인덱스 가격 공식 분석
   - [ ] 미수 펀딩 계산 로직
   - [ ] 신뢰도 스코어 모델 구축

3. 스프레드 타이밍
   - [ ] EMA/SMA lookback 최적화
   - [ ] 진입 z-score (기본 2.0) 튜닝
   - [ ] 청산 z-score (평균 이전) 튜닝
   - [ ] 거래비용 차감 후 순수익 시뮬레이션

4. 인프라
   - [ ] Rust WebSocket 피드 (barter-rs 검토)
   - [ ] 크로스 익스체인지 레이턴시 최적화
   - [ ] 메시지 순서 처리
   - [ ] 리드-래그 분석 파이프라인
```

### 수익 기대치

- 단일 코인/2거래소/top-of-book: PnL 차트 기준 소폭이지만 일관된 수익
- 거래비용 후 "몇 bps" 수준 -> **수수료 최적화가 생존 조건**
- 메이커 실행으로 비용 절감 필수
- 스케일링: 코인 수 x 거래소 쌍 수로 선형 확장 가능

---

## 핵심 수식/파라미터 요약

| 항목 | 값/공식 |
| --- | --- |
| 스프레드 비율 | `price_A / price_B` |
| 스프레드 차이 | `price_A - price_B` (실제 트레이딩 가능) |
| 진입 조건 | `abs(z_score) >= 2.0 * std_dev` |
| 청산 조건 | `z_score -> 0` (또는 exit\_z\_score 파라미터) |
| Mid price | `(ask_price + bid_price) / 2.0` |
| Mean reversion edge | `E[MR(t1,t2)] ~ proportional to abs(z_score)` |
| Edge/Vol ratio | 회귀 진행 시 악화 -> 조기 청산 근거 |
| 데이터 업데이트 간격 | Binance ~30ms |
| 최소 거래소 수 | 자산당 >= 2개 |

---

## 추가 읽기 권장 (같은 시리즈)

| 글 | 파일 |
| --- | --- |
| Small Trader Alpha #4: Funding Arbitrage | `2024-01-30-small-trader-alpha-4-funding-arbitrage.md` |
| How to Level Up Your Arb Game | `2024-04-07-how-to-level-up-your-arb-game.md` |
| Ultimate Crypto Arbitrage Guide | `2025-06-20-ultimate-crypto-arbitrage-guide.md` |
| Finding Arbitrage Opportunities | `2025-09-22-finding-arbitrage-opportunities.md` |
| Ultimate Crypto Latency Guide | `2026-02-08-ultimate-crypto-latency-guide.md` |

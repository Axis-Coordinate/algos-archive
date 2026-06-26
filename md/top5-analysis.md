# Top 5 글 상세 분석 (Axis 관점)

Source HTML: [`html/top5-analysis.html`](../html/top5-analysis.html)

# Top 5 글 상세 분석 (Axis 관점)

> 분석일: 2026-05-20  
> 상태: 7개 파일 중 1개만 완전 크롤링. 나머지 6개는 재크롤링 필요.  
> ⚠️ 재크롤링 후 이 문서 업데이트 필수

---

## 1. Ultimate Crypto Latency Guide (2026-02-08)

**수집률: ~30%**

### 확인된 기법 3가지

| 기법 | 설명 | Axis 적용 |
| --- | --- | --- |
| WS Rotate | 거래소당 WS 연결 3-5개 로테이션, P50 15-25% 개선 | gRPC 드라이버에 즉시 적용 |
| FIX Feed | FIX 프로토콜 직접 연결 (WS 대비 저레이턴시) | Binance/OKX FIX 지원 확인 |
| Machine Gun Orders | 동시 다발 주문으로 체결 성공률 2-3배 | Taker-Taker 동시 체결에 적용 |

### 외부 리서치 보충 수치

| 항목 | 수치 |
| --- | --- |
| <50ms 체결 성공률 | 82% |
| 100-150ms | 48% |
| >150ms | 31% |
| 아비트라지 스프레드 수명 | 200-800ms |
| TCP 커널 튜닝 효과 | 5-12ms 감소 |
| orjson 파싱 전환 | 0.8ms 감소 |

### 즉시 적용 가능 항목

```
# TCP 커널 튜닝 (sysctl.conf)
net.ipv4.tcp_nodelay = 1
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
```

- TCP\_NODELAY + WS 압축 비활성화 → 2-5ms 감소
- 커넥션 풀링 → 15ms 감소
- Tokyo/Singapore 분리 배치 → 20-50ms 감소

---

## 2. A Real HFT/MFT Alpha (2026-04-06)

**수집률: ~20%**

### 확인된 정보

| 항목 | 값 |
| --- | --- |
| 알파 개수 | 2개 (proprietary orderbook alphas) |
| 주파수 | 1h 리밸런싱 |
| 구성 | Cross-sectional z-score portfolio |
| Sortino | >3.0 (합산) |
| Sharpe | >2.0 (개별) |
| 백테스트 | 2021-10 ~ 2025-12 (약 4년) |
| 공개 문헌 | "no public literature" |

### Orderbook Alpha 후보 (같은 저자 다른 글에서 추론)

```
# Order Flow Imbalance (가장 유력)
OFI = (buy_vol - sell_vol) / total_vol
z_score = ts_zscore(OFI, 20)

# Book Depth Imbalance
BDI = bid_depth / (bid_depth + ask_depth)

# Spread Dynamics
spread_delta = delta(ask_price - bid_price, 1)

# 포트폴리오 구성
# 상위 10% long, 하위 10% short, 변동성 역수 가중
```

### Axis 적용

- ClickHouse `hot.price_bbo_minutely_snapshot_v1`로 BDI/OFI 계산 가능
- 기존 EMAMA 시그널과 상관관계 분석 (직교성 확인)
- Cross-sectional z-score L/S 파이프라인 구축

---

## 3. Hide N' Seek Pt.1 & Pt.2 (2023-11/12)

**수집률: Pt.1 ~40%, Pt.2 ~15%**

### Pt.1 — 탐지 회피 8대 기법

| # | 기법 | 핵심 | Axis 적용 |
| --- | --- | --- | --- |
| 1 | Randomization | 주문 사이즈+빈도 2축 랜덤화 | delta-revert 패턴 다양화 |
| 2 | When Detection Doesn't Matter | 충분한 유동성이면 sweep | 고유동성 코인은 aggressive |
| 3 | Aggression | Make vs Take 결정 프레임워크 | Balancer maker/taker 비율 |
| 4 | Retail Flow | 리테일 패턴 모방 | round number 사이즈 등 |
| 5 | Matching Size Distributions | 시장 분포 유지 | 주문 사이즈 분포 모니터링 |
| 6 | Hiding In High Volume | 고거래량 시간대 집중 | 실행 시간대 최적화 |
| 7 | Wash Flow | (참고용) | — |
| 8 | Adversarial Games | 탐지 모델 역설계 | 자체 탐지 테스트 |

### Pt.2 — 탐지 4대 기법

| # | 기법 | 방법 |
| --- | --- | --- |
| 1 | Recurring Frequencies | 시간 간격 autocorrelation |
| 2 | Binning | 사이즈 구간별 Chi-squared/KS test |
| 3 | Fourier Transform | PSD로 숨겨진 주기성 탐지 |
| 4 | Global Orderbook Method | 멀티 거래소 오더북 상관 분석 |

### 예상 효과

- 즉시 적용 (1-2주): 3-5bps 절감
- 중기 (1-2월): 5-10bps 절감
- 현재 delta-revert 45-49bps 기준

---

## 4. Advanced Market Making (2025-08-02)

**수집률: ~15%**

### 확인된 포인트

- 소규모 거래소 MM이 현실적 타겟
- Binance급은 최고 latency tech + 대규모 팀 필요
- MM 시스템 컴포넌트별 분해 설명 구조
- NBBO flutter 차트 (시간대별 BBO 변동 percentile)
- **본문 미확보로 상세 분석 불가 → 재크롤링 최우선**

---

## 5. Small Trader Alpha #6 & #7: Perpetual Arbitrage (2024-07/10)

**수집률: #6 ~60%, #7 ~10%**

### #6 — 전략 구조

- 크로스 거래소 **perp-perp** 통계적 아비트라지
- 스프레드 z-score: **진입 2σ / 청산 mean**
- 파라미터: `lookback`, `entry_z`, `exit_z`

### 핵심 — 정규화 (가장 실무적)

| 정규화 | 설명 |
| --- | --- |
| 스테이블코인 베이시스 | 거래소별 USDT/USDC/USD 가격으로 정규화. 글로벌 가격 사용 금지 |
| 계약 정규화 | 펀딩 인터벌, 금리, 캡, 인덱스 공식 차이 반영 |
| 신뢰도 스코어 | perp 쌍 유사성 점수화 → 사이징에 반영 |

### 리스크 인사이트

- 스프레드 회귀 시 엣지↓ but 변동성 동일 → 리스크-리워드 악화
- **완전 mean reversion 전 조기 청산이 합리적**
- 거래비용 후 "몇 bps" → 비용 최적화 = 생존 조건

### USDC 디페깅 사례

> Coinbase $0.93, 다른 거래소는 몇 센트만 하락 → 거래소별 정규화 안 하면 대규모 손실

### #7 (인트로만)

- Rust 프로덕션 코드 전체 제공 (barter-rs 기반)
- 재크롤링 필수

---

## 재크롤링 현황

| 파일 | 크기 | 상태 |
| --- | --- | --- |
| ultimate-crypto-latency-guide | 1.9KB | ❌ 불완전 |
| a-real-hftmft-alpha | 0.7KB | ❌ 불완전 |
| hide-n-seek-pt1 | 3.5KB | ❌ 불완전 |
| hide-n-seek-pt2 | 1.5KB | ❌ 불완전 |
| advanced-market-making | 1.5KB | ❌ 불완전 |
| perpetual-arbitrage #6 | 32KB | ✅ 거의 완전 |
| perpetual-arbitrage #7 | 1.8KB | ❌ 불완전 |

**전체 76개 중 41개(54%) 불완전.** 새 쿠키로 재크롤링 필요.

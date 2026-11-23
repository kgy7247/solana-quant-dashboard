# Solana Quant — Live Dashboard

솔라나 퀀트 봇의 **공개 지갑** 하나를 실시간으로 보여주는 정적 대시보드입니다.
빌드 과정 없이 `index.html` + `data/` 만으로 GitHub Pages에서 동작합니다.

## 구성

| 경로 | 내용 |
| --- | --- |
| `index.html` | 대시보드 (다크모드 · 반응형 · 15초 자동 갱신) |
| `data/trades_state.json` | 봇 매매 상태 (보유 포지션 · 히스토리) |
| `data/quant_config.json` | 현재 전략 파라미터 |
| `data/evolution.txt` | 1시간 자가진화 로그 (한 줄 = 한 이벤트) |
| `data/wallet_snapshot.json` | 봇이 남기는 잔액·시세 스냅샷 (브라우저가 RPC를 못 부를 때 대체 표시) |

## 실시간 데이터 소스

- **잔액** — Solana Mainnet RPC `getBalance`, `getTokenAccountsByOwner`
  (SPL Token + Token-2022 두 프로그램 모두 조회)
- **가격** — DexScreener 배치 API(유동성 최대 페어) → Jupiter lite price v3 보완
- **매매/전략** — 같은 레포의 `data/*.json`, `data/evolution.txt` (30초마다 재조회)

브라우저에서 직접 호출하므로 서버·키가 필요 없습니다.

## 화면

- 상단 헤더에 **SOL / USD / KRW 통화 토글** (선택은 브라우저에 저장, 실시간 USD→KRW 환율은 1회 조회 후 적용)
- 총 평가액 · SOL 잔액 · 토큰 평가액 · 누적 손익(실현 + 미실현) — 선택 통화로 표시
- 누적 손익 추이 라인 차트, 자산 구성 도넛, 거래별 실현 손익 바 차트 (모두 선택 통화 축)
- 현재 전략 파라미터 카드 (버전 · min_score · 슬롯 · 트레일링 · 손절 · 필터)
- 보유 포지션 표 (수량 · 현재가 · 진입가 · **코인 수익률(SOL 기준)** · **총 평가액 변동(USD/KRW)** · Solscan)
- 매매 히스토리 표 (투입/회수 SOL(스왑) · 손익(SOL) · 손익 %(스왑 기준) · 트랜잭션 링크)
- 자가진화 로그 타임라인

## 손익 회계 기준 (2026-11-23 통일)

모든 매매 손익은 **순수 스왑 SOL** 기준입니다.

- `entry_sol` = 스왑에 실제로 넣은 SOL, `out_sol` = 스왑에서 받은 SOL → `pnl = out_sol − entry_sol`, `pnl% = pnl / entry_sol`
  (예: PUMP 0.3966 → 0.415495 SOL = **+4.76%**)
- **ATA 토큰 계정 렌트 예치금**(~0.0015~0.002 SOL, 계정 소각 시 회수)과 **tx 수수료**는 손익에 섞지 않고
  `ata_rent_sol` / `tx_fee_sol` 로 따로 집계해 화면 하단에 별도 표기합니다.
- 보유 포지션은 두 지표를 분리합니다.
  - **코인 수익률 (SOL)** — 코인 가격만의 변동(진입가 SOL 대비). SOL 시세 변동 제외
  - **총 평가액 변동 (USD/KRW)** — 코인 가격 + SOL 시세 변동을 모두 반영한 USD/KRW 평가액 변동률
- `data/wallet_snapshot.json`의 `positions_meta`가 진입 시점 가격(`entry_price_sol`, `sol_usd_entry`, `entry_price_usd`)과
  ATA 렌트를 내려주므로, 기록이 없는 기존 포지션도 진입 시점 SOL 시세로 USD 변동률을 계산합니다.

## `evolution.txt` 형식

파이프(`|`)로 구분된 한 줄 형식입니다. 대시보드는 앞의 ISO 시각과 `init` / `trade` / `tune`
종류만 읽고 나머지는 그대로 출력하므로, 형식이 조금 달라도 깨지지 않습니다.

```
2026-11-19T15:38Z | v3 | trade | mihao BUY 0.58SOL score=100/100 | EMA9>21 · BB_Breakout · Donchian20_High
```

봇이 `data/evolution.txt` 끝에 한 줄씩 append 하고 커밋하면 대시보드 타임라인에 그대로 반영됩니다.

## 로컬 확인

```bash
python3 -m http.server 8000
# http://localhost:8000
```

> 이 대시보드는 봇의 **공개 지갑 주소**만 사용합니다. 개인 키·개인 정보는 포함하지 않습니다.

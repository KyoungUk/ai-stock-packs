---
name: weekly-pack-reader
description: KyoungUk/ai-stock-packs 저장소의 주간 후보 팩을 읽고 정해진 형식으로 판단 결과를 출력한다.
---

# 주간 후보 팩 읽기 규약

이 스킬은 매주 `ai-stock-packs` 저장소에 새로 올라오는 후보 팩(`packs/YYYY-MM-DD.json`)을 읽어
주말 종목선정 논의를 진행하는 챗 세션이 따라야 할 규약이다.

**원칙: 계산은 홈PC, 판단은 챗.** 팩 안의 수치(매물대·저항·지지·ATR 등)는 이미 결정적으로
계산되어 있다 — 이 스킬은 그 수치를 읽고 판단만 한다. **매물대를 챗에서 재계산하지 않는다.**

## 1. 팩 URL

```
https://raw.githubusercontent.com/KyoungUk/ai-stock-packs/main/packs/YYYY-MM-DD.json
```
`YYYY-MM-DD`는 팩이 생성된 금요일 날짜(`pack_date` 필드와 동일).

## 2. 읽는 순서

1. **시장** (`market` 필드): `kospi_close`, `kosdaq_close`, `kospi_20d_change_pct`로 전반적 국면 파악
2. **후보별로 순서대로**:
   1. 현재가(`close`) vs 저항 거리(`resistance[].distance_pct`)
   2. 매물대 분포(`volume_profile`) — 현재가 근처 구간의 거래량 비중(`share_pct`)
   3. 호가 잔량(`hoga_close.asks`/`hoga_close.bids`) — 있으면 참고, 없으면(빈 배열) 생략
   4. ATR(`atr14`) — 손절폭·회차가격 간격 산정에 사용

## 3. 출력 형식 고정

후보별로 다음 형식을 그대로 지킨다. 이 형식이 그대로 `register_weekly_pick.py` 인자가 된다.

```
[종목명 종목코드]
저항가: {price}
회차가격: {price1}, {price2}, {price3} (3~4개)
예산: {total_budget}
근거: {한 줄 서술}
```

## 4. 하지 말 것

- 팩에 없는 수치를 추정하지 않는다.
- 펀더멘털을 언급하지 않는다 — 별도로 `company-decoder-korean` 스킬을 통해 다룬다(M08b 구조).
- 점수를 매기지 않는다("A등급", "80점" 등 금지) — 저항가/회차가격/예산/근거만 출력한다.

## 5. 마지막 단계

확정된 값을 아래 형태의 CLI 명령으로 출력한다 — 사용자가 홈PC에서 그대로 실행한다.

```
python scripts/register_weekly_pick.py --symbol 005930 --resistance 75000 --tranches 70000,71000,72000 --budget 1000000 --reason "매물대 상단 돌파 대기"
```

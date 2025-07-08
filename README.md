# 💡 LogicWorks 기반 자판기 회로 구현 프로젝트

## 📌 프로젝트 개요

본 프로젝트는 LogicWorks를 활용하여 자판기의 기능을 회로로 구현한 디지털 시스템 설계 과제입니다. 컴퓨터구성 과목에서 학습한 **ALU, BCD, MUX, Adder, Subtractor** 등의 이론을 바탕으로, 실제 동작하는 자판기 회로를 설계 및 구현하는 것을 목표로 하였습니다.

## 전체 설계
<img width="1381" height="1272" alt="Image" src="https://github.com/user-attachments/assets/96d1e446-e6a1-4dce-8114-4a0075250459" />>

---

## 🎯 주요 기능

| 기능 구분       | 상세 설명 |
|----------------|----------|
| 💰 **입금 기능** | 10원 ~ 50,000원 단위의 금액 입력 가능 |
| 🧃 **음료 선택** | 8종 음료 선택, 가격 300원 ~ 8000원 |
| 💸 **잔액 계산** | BCD Adder/Subtractor를 이용한 실시간 잔액 계산 |
| 🔁 **잔돈 반환** | 잔액을 동전/지폐 단위로 분할하여 반환 |
| 🔋 **음료 재고 관리** | 재고가 0일 경우 음료 구매 불가 처리 |
| ⏱ **점검 시간 기록** | 점검 시각을 Segment에 저장하여 확인 가능 |
| 🔦 **LED 안내** | 구매 가능/불가능 여부에 따라 LED 점등 |

---

## 🛠 사용된 주요 논리 회로

| 구성 요소        | 설명 |
|------------------|------|
| **Money Checker** | 입력 금액을 10~50,000원 단위로 구분 |
| **BCD Adder/Subtractor** | 잔액 계산을 위한 BCD 기반 연산 회로 |
| **MUX & Encoder** | 음료 선택 및 제어 회로 구현 |
| **Carry/Borrow 처리** | 자리 올림 및 자리 내림 처리 로직 |
| **LED Checker** | 잔액 부족 시 LED 비점등 처리 |
| **BCD to 7-Segment** | 디지털 숫자 표시 |
| **Change Logic** | 단위별 잔돈 계산 및 반환 처리 |
| **Counter + FF** | 점검 시각 저장 기능 구현 |
| **Stock Control** | 음료 재고량 관리 및 표시 기능 |

---

## 📐 회로 구조 요약

- **입금 처리부**: 각 단위별 금액을 Money Checker에서 감지하고 BCD Adder로 누적
- **음료 구매부**: MUX로 음료 선택 → 가격과 잔액 비교 → Subtractor → 잔액 갱신
- **LED 표시부**: 구매 가능 여부를 LED로 표시
- **잔돈 반환부**: 현재 잔액에서 높은 단위부터 나누며 각 화폐 단위 출력
- **부가 기능부**: 점검 시각 저장, 재고 상태 저장 및 감소 로직 포함
- **점검 시간 저장 기능**: Counter 및 FF 회로로 마지막 점검 시각 저장
- **음료 재고 관리**: 최대 2개 재고 유지, 0일 경우 구매 불가 처리
 
---

## 🛠 사용된 주요 논리 회로 설명

### 💰 금액 계산 - ADDER 파트

입력된 금액의 종류를 구분하고 각 자릿수에 더함.

<img width="788" height="555" alt="Image" src="https://github.com/user-attachments/assets/ffc7d1e4-4c0b-4c6b-aa2a-97122abf96f0" />

- **MoneyChecker**: 1원, 5원, 10원 등 입력 금액을 판단해 자리값 생성  
- **CARRY**: 자리올림 여부 판단, 상위 자리로 carry 전달  
- **BCD_ADDER**: 자릿수 값과 입력값을 더해 합과 carry 출력

1원 단위부터 100,000원까지 자릿수를 BCD 방식으로 처리

### 💵 MoneyChecker

입력된 돈의 단위를 구분하고 각 자릿수에 맞게 BCD 형태로 출력.

<img width="939" height="845" alt="Image" src="https://github.com/user-attachments/assets/9f7cb5f9-4923-45b7-85dc-3032655efc83" />

- **1digit**: 10원 단위 입력 처리 → BCD 입력값 0001  
- **5digit**: 50원 이상 입력 처리 → BCD 입력값 0101  
- **A0 ~ A3**: 자릿수 선택 (10원, 100원, 1000원, 10000원)

각 자리의 A와 B는 Full Adder로 들어가 덧셈 수행.  
- 첫 Adder에는 외부 carry 없음 → C0 = 0  
- 이후 Carry는 다음 자리로 전달  
- 최종 Carry는 `CARRY_OUT`, 결과 Sum은 `N0 ~ N3`로 출력

### ➕ BCD ADDER

10 이상의 값을 처리할 때 BCD 보정을 수행.

<img width="825" height="493" alt="Image" src="https://github.com/user-attachments/assets/1ffab677-b474-4c37-9606-a0da02fca393" />

- **SELECT**: carry 조건(MoneyChecker의 carry_out + CARRY 결과)을 OR 연산 후 선택  
- **MUX**: 입력값에 따라 보정값(10)을 더할지 결정  
  - `select=1` → 10 이상으로 간주 → 보정값 0110 추가  
  - `select=0` → 그대로 유지  
- **FULL ADDER**: 보정 포함한 덧셈 수행  
- 출력값: `S0 ~ S3` (1의 자리 유지), `CARRY` (자리올림 발생 시 전달)

### 🔁 CARRY

MoneyChecker의 출력값(N0~N3)을 받아 자리올림 여부 판단.

<img width="863" height="326" alt="Image" src="https://github.com/user-attachments/assets/7db69af2-1cd2-4617-a3d5-c90a8623326c" />

- 입력값 4개(K0~K3)를 OR, AND 연산하여 `bcd_carry_out` 생성  
- `bcd_carry_out`은 BCD ADDER의 carry 판단에 사용  
- MoneyChecker의 carry와 OR 연산하여 다음 자리로 올릴 carry 결정

### 🔗 Serial connected Subtractor

연속된 뺄셈을 통해 잔액 계산 수행.

<img width="795" height="736" alt="Image" src="https://github.com/user-attachments/assets/0ceadeb4-be80-431f-8ce5-f38729721285" />

- **I0~I3**: 현재 금액 (current)  
- **F0~F3**: 음료 가격  
- 가격은 1’s 보수로 변환되어 Full Adder에 입력  
- 두 값을 더해 `current - price` 연산 수행  
- 결과는 다음 Subtractor로 전달되어 연속 계산 가능

### 🔀 PL OR MI (Adder/Subtractor 선택)

입금 또는 음료 선택에 따라 출력 경로 결정.

<img width="629" height="440" alt="Image" src="https://github.com/user-attachments/assets/27642816-9a00-4ae6-9855-b416c22c16d4" />

- **AORS 신호**에 따라 2:1 MUX에서 ADDER 값 또는 SUBTRACTOR 값 선택  
- **P0~P3**: ADDER 출력, **M0~M3**: SUBTRACTOR 출력  
- 선택된 결과는 **R0~R3**으로 출력됨

### 💸 잔돈 반환 전체 회로도

잔액을 받아 단위별 잔돈 개수를 계산하고 출력.

<img width="951" height="366" alt="Image" src="https://github.com/user-attachments/assets/76b45c6c-5039-4cb6-bdb8-bea81514473a" />

- **Overflow Check**: 잔액 계산 결과를 출력  
- **CHANGE**: 자릿수별로 5씩 빼며 잔돈 개수 계산  
- **CHANGE PLUS**: 잔돈 개수를 누적하여 출력  
- 5가 더 이상 빼지지 않을 때까지 반복 수행  
- 큰 단위 잔돈은 **LAST CHANGE**, **LAST CHANGE PLUS**를 통해 처리

### ⚠️ OVERFLOW CHECK (symbol)

금액 범위를 초과하거나 음수일 경우 처리하는 회로.

<img width="670" height="524" alt="Image" src="https://github.com/user-attachments/assets/5137bdf0-a23d-4b95-bad3-c8baf24d735f" />

- **PM0~PM3**: MoneyChecker 결과 입력  
- **OG0~OG3**: 덧셈 전 값, **RR10~RR13**: 뺄셈 전 값  
- **S0, S1**: 10만 자리의 borrow와 carry → 선택 신호  
- 내부 4:1 MUX를 통해 조건에 따라 이전 값 유지 출력  
  - `S0=1`: 음수 → 뺄셈 전 값 출력  
  - `S1=1`: 오버플로우 → 덧셈 전 값 출력
 
### 💸 CHANGE

잔액에서 5를 빼며 단위 잔돈(예: 500원) 개수 계산.

<img width="641" height="573" alt="Image" src="https://github.com/user-attachments/assets/a206ba63-6df7-4496-9baf-f52feed550ba" />

- **CHA0~CHA3**: 현재 금액 입력  
- **BCD SUBTRACTER**: 5(0101)와 뺄셈 수행  
- **borrow = 1** → 잔액 부족 → RADI0 = 0 → 잔돈 0개  
- **borrow = 0** → 뺄셈 성공 → RADI0 = 1 → 잔돈 1개  
- MUX를 통해 뺀 값 또는 기존 값을 선택해 다음 단위(100원) 전달

### 🥤 음료 선택 및 LED 표시

<img width="814" height="986" alt="Image" src="https://github.com/user-attachments/assets/e0690e0b-e999-4aa1-b57c-521dc019c60a" />

- 각 음료 심볼에 선택 신호(1)가 입력되면 해당 가격이 출력됨  
- 출력된 가격은 `TOTAL_DRINK`로 전달되어 잔액에서 차감 (Subtract)  
- **LED_CHK**: 현재 금액이 음료 가격 이상이면 LED 점등  
  - 금액 부족 시 LED 꺼짐 → 선택 불가 표시  
  - 자판기 전원(ON/OFF) 상태도 AND 게이트로 함께 판단

### ⏱ 점검 시간 기록

<img width="903" height="589" alt="Image" src="https://github.com/user-attachments/assets/b05a5883-f3f0-46cc-b52f-a51aac975ec5" />

- **Counter**: 실시간으로 시간 값을 증가 (초, 분, 시, 년월일 등)  
- **점검 스위치**를 누르면 해당 시점의 시간이 **Flip-Flop**에 저장됨  
- 저장된 시간은 **BCD TO SEGMENT**를 통해 디스플레이로 출력  
- 가장 최근의 점검 시간을 확인할 수 있음

### 📦 재고 관리 기능

<img width="888" height="297" alt="Image" src="https://github.com/user-attachments/assets/6db1d10c-f5ef-4c7b-97cf-e9a1be71a26b" />

- **PLUS 버튼**: 음료 재고 +1 (최대 2개까지)  
- **Minus 버튼**: 음료 선택 시 재고 -1  
- 재고가 **0개일 경우**에는 음료 선택 불가  
- 재고 수량은 **BCD TO SEGMENT**를 통해 화면에 표시됨  
- 음료 개수에 따라 `OUT_CHOCO` 신호로 상태 제어




 




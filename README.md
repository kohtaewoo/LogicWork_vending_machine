# 💡 LogicWorks 기반 자판기 회로 구현 프로젝트

## 📌 프로젝트 개요

LogicWorks를 활용하여 자판기의 기능을 회로로 구현한 디지털 시스템 설계 과제입니다. 컴퓨터구성 과목에서 학습한 **ALU, BCD, MUX, Adder, Subtractor** 등의 이론을 바탕으로, 실제 동작하는 자판기 회로를 설계 및 구현하였습니다.

## 🧩 전체 설계도

<img width="1381" height="1272" alt="전체 회로도" src="https://github.com/user-attachments/assets/96d1e446-e6a1-4dce-8114-4a0075250459" />

---

## 🎯 주요 기능 요약

| 기능            | 설명 |
|----------------|------|
| 💰 입금 기능       | 10원 ~ 50,000원 단위 금액 입력 가능 |
| 🧃 음료 선택       | 8종 음료 선택 가능 (가격: 300원 ~ 8000원) |
| 💸 잔액 계산       | BCD Adder/Subtractor를 이용한 실시간 잔액 계산 |
| 🔁 잔돈 반환       | 잔액을 화폐 단위(지폐/동전)로 분할하여 반환 |
| 📦 음료 재고 관리   | 재고 0일 경우 구매 불가 처리 |
| ⏱ 점검 시간 기록   | 점검 시각을 Segment에 저장하여 확인 가능 |
| 🔦 LED 안내       | 잔액 부족/가능 여부에 따라 LED 점등 |

---

## ⚙️ 사용된 주요 논리 회로

| 회로 구성 요소      | 설명 |
|------------------|------|
| Money Checker    | 입력 금액을 단위별로 구분 |
| BCD Adder/Subtractor | 실시간 금액 연산 수행 |
| MUX & Encoder    | 음료 선택 및 출력 선택 |
| Carry/Borrow     | 자리 올림/자리 내림 처리 |
| LED Checker      | 구매 가능 여부에 따른 LED 점등 |
| BCD to Segment   | 숫자를 7-Segment로 출력 |
| Change Logic     | 잔돈 단위별 계산 |
| Counter + FF     | 점검 시간 저장 |
| Stock Control    | 재고량 표시 및 제어 |

---

## 📐 회로 구조 요약

- **입금부**: 금액 입력 → Money Checker → BCD Adder 누적
- **음료 구매부**: 음료 선택 → 가격 차감 → 잔액 갱신
- **LED부**: 잔액이 부족할 경우 비점등
- **잔돈 반환부**: 잔액에서 단위별 잔돈 계산 및 반환
- **점검부**: Counter + FF로 마지막 점검 시간 저장
- **재고부**: 재고 수량 제어 및 화면 출력 (최대 2개)

---

## 🛠 주요 회로 설명

### 💰 금액 계산 - ADDER

<img width="788" height="555" alt="ADDER" src="https://github.com/user-attachments/assets/ffc7d1e4-4c0b-4c6b-aa2a-97122abf96f0" />

- 각 자릿수에 맞는 금액을 받아 BCD 방식으로 누적
- CARRY 회로와 함께 자리올림 처리

---

### 💵 Money Checker

<img width="939" height="845" alt="MoneyChecker" src="https://github.com/user-attachments/assets/9f7cb5f9-4923-45b7-85dc-3032655efc83" />

- 입력 금액을 10, 50, 100, 1000 단위로 분리하여 BCD 출력
- 각 자릿수에 대응하는 A, B를 Full Adder로 덧셈 처리
- Carry Out으로 상위 자리로 자리올림

---

### ➕ BCD ADDER

<img width="825" height="493" alt="BCD ADDER" src="https://github.com/user-attachments/assets/1ffab677-b474-4c37-9606-a0da02fca393" />

- 보정 조건 발생 시 6을 더함 (10 이상의 경우)
- Carry 판단 후 SELECT로 제어 → MUX 통해 보정값 선택

---

### 🔁 CARRY 회로

<img width="863" height="326" alt="CARRY" src="https://github.com/user-attachments/assets/7db69af2-1cd2-4617-a3d5-c90a8623326c" />

- 자리올림 조건 확인 (BCD 1000 이상)
- MoneyChecker의 Carry와 OR 연산하여 최종 Carry 판단

---

### 🔗 Serial Subtractor

<img width="795" height="736" alt="Subtractor" src="https://github.com/user-attachments/assets/0ceadeb4-be80-431f-8ce5-f38729721285" />

- 현재 잔액에서 음료 가격 차감
- Subtractor는 1의 보수 + Full Adder 방식 사용
- 다단 연결로 여러 자리 뺄셈 연속 수행 가능

---

### 🔀 입금 vs 음료 선택 - MUX 제어

<img width="629" height="440" alt="MUX" src="https://github.com/user-attachments/assets/27642816-9a00-4ae6-9855-b416c22c16d4" />

- AORS 신호로 ADDER ↔ SUBTRACTOR 선택
- 2:1 MUX로 동작하여 금액 연산 방향 결정

---

### 💸 잔돈 반환 회로

<img width="951" height="366" alt="잔돈 회로" src="https://github.com/user-attachments/assets/76b45c6c-5039-4cb6-bdb8-bea81514473a" />

- 반복적으로 5를 빼며 해당 단위의 잔돈 개수 계산
- CHANGE → CHANGE PLUS → LAST CHANGE 구조로 설계

---

### ⚠️ 오버플로우 & 음수 처리

<img width="670" height="524" alt="오버플로우 체크" src="https://github.com/user-attachments/assets/5137bdf0-a23d-4b95-bad3-c8baf24d735f" />

- 잔액이 음수가 되거나 범위를 초과하는 경우 기존 값 유지
- 내부 MUX로 상황에 맞는 값 선택 출력

---

### 🔄 CHANGE (단위별 잔돈 계산)

<img width="641" height="573" alt="CHANGE" src="https://github.com/user-attachments/assets/a206ba63-6df7-4496-9baf-f52feed550ba" />

- 금액에서 5를 반복적으로 차감해 잔돈 개수 계산
- borrow 발생 시 잔돈 없음으로 처리

---

### 🥤 음료 선택 및 LED 제어

<img width="814" height="986" alt="음료 회로" src="https://github.com/user-attachments/assets/e0690e0b-e999-4aa1-b57c-521dc019c60a" />

- 음료 선택 시 해당 가격 출력 → Subtractor 입력
- 잔액 ≥ 가격 → LED 점등  
- 부족 시 LED 꺼짐 + 구매 불가

---

### ⏱ 점검 시간 기록

<img width="903" height="589" alt="점검 시간 회로" src="https://github.com/user-attachments/assets/b05a5883-f3f0-46cc-b52f-a51aac975ec5" />

- Counter로 실시간 시간 증가
- 점검 스위치로 현재 시각을 Flip-Flop에 저장
- 저장된 시간은 Segment로 표시

---

### 📦 재고 관리

<img width="888" height="297" alt="재고 회로" src="https://github.com/user-attachments/assets/6db1d10c-f5ef-4c7b-97cf-e9a1be71a26b" />

- PLUS 버튼: 재고 +1 (최대 2개)
- Minus 버튼: 재고 -1 (음료 선택 시)
- 재고가 0이면 구매 불가 처리  
- 재고 수량은 Segment로 표시됨

---

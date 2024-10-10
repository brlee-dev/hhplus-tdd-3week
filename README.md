# 콘서트 예약 서비스 ERD
![image](https://github.com/user-attachments/assets/64903fcb-a159-47ff-89ab-2b1fb86e95fd)

---

# API 명세 작성

## 1. 유저 토큰 발급 API

### 설명
- 사용자가 대기열에 들어가 서비스에 접근할 수 있도록, 대기열 토큰을 발급합니다.
  
### 엔드포인트
- `POST /api/v1/queue/token`

### 요청
```json
{
  "userId": "12345"
}
```

### 응답
 - 성공(HTTP 200)
```json
{
  "token": "gAAAAABnB37u4-_UVh4T8vfPb-59a6EUiBORbQyaN4J30j0ukacslIYJN5U3_q2e6pS1iGpe16xRUqRXoxzIn2Css6nj9hQhnMspZXc7DpOZIVg7UgegENxB7Br7sumCprzZHlHW87gN9F7eaoUVhp1bqxyw9C8_PbpFNjQD6uTvQy-uFfa0yFg",
  "queuePosition": 5,
  "remainingTime": "10:00"
}
```
- 실패, 이미 대기열에 존재하는 경우(HTTP 409)
```json
{
  "status": "error",
  "message": "이미 예약 요청 상태 입니다."
}
```

---

## 2. 예약 가능 날짜 조회 API

### 설명
- 예약 가능한 날짜 목록을 반환 합니다.
  
### 엔드포인트
- `GET /api/v1/seats/dates`

### 요청
- 없음

### 응답
 - 성공(HTTP 200)
```json
{
  "availableDates": [
    "2024-10-10",
    "2024-10-11",
    "2024-10-12"
  ]
}
```

---

## 3. 특정 날짜의 좌석 조회 API

### 설명
- 특정 날짜에 예약 가능한 좌석 목록을 조회합니다.
  
### 엔드포인트
- `GET /api/v1/seats?date=YYYY-MM-DD`

### 요청
```json
{
  "date": "2024-10-10"
}
```

### 응답
 - 성공(HTTP 200)
```json
{
  "availableSeats": [
    {"seatNumber": 1, "isReserved": false},
    {"seatNumber": 2, "isReserved": false},
    {"seatNumber": 3, "isReserved": true}
  ]
}
```
- 실패, 날짜 형식이 잘못된 경우(HTTP 400)
```json
{
  "status": "error",
  "message": "잘못된 날짜 입니다."
}
```

---

## 4. 좌석 예약 요청 API

### 설명
- 특정 날짜의 특정 좌석을 임시로 예약합니다. 임시 예약은 약 5분 동안 유지되며, 결제가 완료되지 않으면 해제됩니다.
  
### 엔드포인트
- `POST /api/v1/reservations`

### 요청
```json
{
  "userId": "12345",
  "token": "gAAAAABnB37u4-_UVh4T8vfPb-59a6EUiBORbQyaN4J30j0ukacslIYJN5U3_q2e6pS1iGpe16xRUqRXoxzIn2Css6nj9hQhnMspZXc7DpOZIVg7UgegENxB7Br7sumCprzZHlHW87gN9F7eaoUVhp1bqxyw9C8_PbpFNjQD6uTvQy-uFfa0yFg",
  "date": "2024-10-10",
  "seatNumber": 1
}
```

### 응답
 - 성공(HTTP 200)
```json
{
  "status": "success",
  "reservationId": "67890",
  "message": "Seat reserved temporarily for 5 minutes."
}
```
- 실패, 좌석이 이미 예역된 경우(HTTP 409)
```json
{
  "status": "error",
  "message": "이미 다른사람이 예약한 좌석 입니다."
}
```

- 실패, 유효하지 않은 토큰(HTTP 401)
```json
{
  "status": "error",
  "message": "관리자에게 문의 하세요."
}
```

---

## 5. 잔액 충전 API

### 설명
- 사용자가 결제에 필요한 금액을 충전합니다.
  
### 엔드포인트
- `POST /api/v1/users/balance`

### 요청
```json
{
  "userId": "12345",
  "amount": 10000
}
```

### 응답
 - 성공(HTTP 200)
```json
{
  "status": "success",
  "newBalance": 15000
}
```
- 실패, 잘못된 요청(HTTP 400)
```json
{
  "status": "error",
  "message": "입력 값은 0보다 커야합니다"
}
```

---

## 6. 잔액 조회 API

### 설명
- 사용자의 현재 잔액을 조회 합니다.
  
### 엔드포인트
- `GET /api/v1/users/balance?userId=12345`

### 요청
```json
{
  "userId": "12345",
}
```

### 응답
 - 성공(HTTP 200)
```json
{
  "userId": "12345",
  "balance": 15000
}
```
- 실패, 사용자 없음(HTTP 404)
```json
{
  "status": "error",
  "message": "존재 하지 않은 사용자입니다."
}
```

---

## 7. 결제 API

### 설명
- 임시 예약된 좌석에 대해 결제를 진행하고, 결제가 완료되면 좌석의 소유권을 확정합니다.
  
### 엔드포인트
- `POST /api/v1/payments`

### 요청
```json
{
  "userId": "12345",
  "reservationId": "67890",
  "token": "gAAAAABnB37u4-_UVh4T8vfPb-59a6EUiBORbQyaN4J30j0ukacslIYJN5U3_q2e6pS1iGpe16xRUqRXoxzIn2Css6nj9hQhnMspZXc7DpOZIVg7UgegENxB7Br7sumCprzZHlHW87gN9F7eaoUVhp1bqxyw9C8_PbpFNjQD6uTvQy-uFfa0yFg"
}
```

### 응답
 - 성공(HTTP 200)
```json
{
  "status": "success",
  "message": "충전이 완료 되었습니다.",
  "newBalance": 5000
}
```
- 실패, 잔액 부족(HTTP 402)
```json
{
  "status": "error",
  "message": "잔액이 부족 합니다."
}
```

- 실패, 예약 만료(HTTP 410)
```json
{
  "status": "error",
  "message": "예약이 만료 되었습니다."
}
```

---
# 기본 패키지 구조

```plaintext
src/main/java/com/concertreservation/
│
├── application            # 유스케이스와 비즈니스 로직 레이어
│   ├── service            # 서비스 클래스
│   └── dto                # 요청/응답 DTO 클래스
│
├── domain                 # Domain 레이어
│   ├── model              # 엔티티 및 핵심 도메인 모델
│   ├── repository         # 인터페이스로 구현한 리포지토리 (Repository 인터페이스)
│   └── exception          # 도메인 관련 예외 처리
│
├── infrastructure         # Infrastructure 레이어
│   ├── persistence        # JPA 엔티티 및 Repository 구현체
│   ├── queue              # 대기열 시스템
│   └── config             # Spring 설정 파일 및 의존성 설정
│
└── interface              # Presentation 레이어
    ├── controller         # REST 컨트롤러 (API 엔드포인트)
    └── security           # 보안 및 인증 관련
```

---

# 기술 스택

1. **Main Framework**
   - Spring Boot

2. **Test Framework**
   - JUnit

3. **DBMS**
   - MySQL

4. **ORM & 데이터베이스 액세스**
   - Spring Data JPA

5. **빌드 도구**
   - Gradle

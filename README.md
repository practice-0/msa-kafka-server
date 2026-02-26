# Kafka Server

Event-Driven Board System의 비동기 이벤트 브로커입니다.

서비스 간 직접 호출을 제거하고,  
확장성과 결합도 최소화를 위해 Kafka 기반 이벤트 아키텍처를 적용했습니다.

---

## 1. 역할

- 서비스 간 비동기 통신
- 이벤트 기반 상태 변경 처리
- 확장 가능한 메시지 브로커

---

## 2. 사용 이벤트 목록

### 1) user.signed-up
- 발행: user-service
- 소비: board-service
- 목적: 게시글 조회용 사용자 Read Model 구성

### 2) board.created
- 발행: board-service
- 소비: user-service
- 목적: 사용자 활동 점수 적립

---

## 3. 설계 의도

서비스 간 직접 API 호출을 최소화하고  
상태 변경을 이벤트로 전파하는 구조를 채택했습니다.

이를 통해:

- 서비스 간 결합도 감소
- 확장 시 독립 배포 가능
- 비동기 확장성 확보

---

## 4. 전달 보장 모델

Kafka는 **at-least-once** 전달 모델을 가집니다.

따라서:

- 동일 이벤트가 중복 소비될 수 있음
- 운영 환경에서는 이벤트 ID 기반 멱등성 전략 필요

현재는 단순 소비 구조이며,  
실제 운영 환경에서는 중복 방지 테이블 또는 idempotency key 전략을 적용할 수 있습니다.

---

## 5. 파티션 전략

현재 구성:

- 기본 파티션 수: 1
- 단일 브로커 구성 (Local)

운영 환경에서는:

- MSK Multi-AZ 구성
- 파티션 확장 가능
- Consumer Group 기반 수평 확장 지원

---

## 6. Local 실행

```bash
docker-compose up -d
```

### 기본 포트: 9092

---

## 7. 운영 환경 구성
- AWS MSK (Multi-AZ)
- Private Subnet 배치
- 서비스는 내부 네트워크를 통해 Kafka에 접근

---

### 8. 아키텍처 내 위치
```text
Client → Gateway → Board Service
                        │
                        └── Event Publish → Kafka → User Service
```

Kafka는 동기 처리의 병목을 분리하는 핵심 구성 요소입니다.

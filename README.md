

---

# 📊 Monitoring & Observability Architecture

## 1. 모니터링 레포지토리 개요

본 레포지토리는 최종 프로젝트에서 사용된 **모니터링 및 관측(Observability) 인프라**를 관리하기 위한 저장소이다.
서비스의 단순한 “정상 동작 여부” 확인을 넘어, **장애 원인 추적·운영 안정성·확장성 판단**을 목표로 설계되었다.

모니터링 영역은 서비스 영역과 분리된 **독립적인 Management Zone**에서 운영된다.

---

## 2. 모니터링 설계 목표

본 모니터링 시스템은 다음 목표를 중심으로 설계되었다.

* 장애 발생 시 **원인 추적이 가능한 구조**
* 메트릭과 로그를 분리하되 상호 연계 가능
* 서비스 장애가 모니터링 시스템에 영향을 주지 않도록 분리
* 실제 운영 환경에서 사용 가능한 구성

“그래프를 그리는 것”이 목적이 아니라,
**문제가 생겼을 때 답을 주는 시스템**을 목표로 하였다.

---

## 3. 전체 모니터링 아키텍처 개요

모니터링 스택은 다음과 같이 구성된다.

* **Prometheus**: 메트릭 수집
* **Grafana**: 시각화 및 대시보드
* **Loki**: 로그 수집
* **Promtail**: 로그 수집 에이전트

이 구성은 CNCF 권장 Observability 패턴을 기반으로 하며,
Kubernetes 환경과 On-Premise 환경 모두를 고려하여 설계되었다.

---

## 4. Management Zone 분리 전략

모니터링 컴포넌트는 서비스 영역과 분리된 **Management Zone**에서 동작한다.

### 분리 이유

* 서비스 장애 시 모니터링 시스템 유지
* 보안 사고 발생 시 영향 범위 최소화
* 운영 도구 접근 제어 용이

### 포함 컴포넌트

* Prometheus
* Grafana
* Loki
* Keycloak (인증)

Management Zone은 “운영자만 접근 가능한 영역”이라는 전제를 기반으로 구성되었다.

---

## 5. Prometheus 구성

### 역할

* 시스템 및 애플리케이션 메트릭 수집
* Kubernetes 클러스터 상태 모니터링

### 수집 대상

* Kubernetes Node / Pod
* Spring Boot Actuator Metrics
* Container Resource Metrics (CPU / Memory)

### 설계 포인트

* Pull 방식 기반 안정적인 수집
* Label 기반 메트릭 분류
* 향후 Alertmanager 연동을 고려한 구조

---

## 6. Grafana 구성

<img width="1521" height="827" alt="image" src="https://github.com/user-attachments/assets/c7000566-8f82-4dbd-93d8-85b5fecc6095" />


### 역할

* 메트릭 시각화
* 운영 대시보드 제공

### 주요 대시보드

* Kubernetes Cluster Overview
* Node / Pod Resource Usage
* Application-Level Metrics

### 접근 제어

* Keycloak 연동을 통한 사용자 인증
* 대시보드 접근 권한 분리 가능 구조

Grafana는 단순 시각화 도구가 아니라
**운영 의사결정을 돕는 도구**로 활용된다.

---

## 7. Loki 기반 로그 수집

<img width="1534" height="817" alt="image" src="https://github.com/user-attachments/assets/0bc9d788-eb54-449b-9fe6-626d380ecb3f" />


### Loki 도입 이유

* 메트릭과 로그의 자연스러운 연계
* Elasticsearch 대비 낮은 운영 부담
* Kubernetes 친화적인 구조

### 로그 수집 흐름

1. Promtail이 각 노드에서 로그 수집
2. Loki로 로그 전송
3. Grafana에서 메트릭과 로그 연계 조회

### 로그 활용 목적

* 장애 발생 시 시간대별 로그 분석
* 특정 Pod / 서비스 단위 로그 필터링

---

## 8. 로그 & 메트릭 연계 전략

본 모니터링 시스템의 핵심은
**메트릭 → 로그로 자연스럽게 이어지는 분석 흐름**이다.

<img width="662" height="389" alt="image" src="https://github.com/user-attachments/assets/f742d972-e174-459f-8d99-5fd61d175145" />


예시 흐름:

1. Grafana 대시보드에서 CPU 급증 확인
2. 해당 Pod 식별
3. 동일 시간대 Loki 로그 조회
4. 장애 원인 분석

이를 통해 “보인다”에서 끝나는 모니터링이 아니라
**분석 가능한 Observability**를 구현하였다.

---

## 9. 보안 및 접근 제어

### 인증

* Keycloak 기반 중앙 인증
* 운영자 전용 접근 구조

### 네트워크 보안

* Management Zone 내부 통신만 허용
* 외부 직접 접근 차단

### 운영 원칙

* 모니터링 시스템은 서비스보다 먼저 보호된다

---

## 10. 운영 시나리오 & 활용 예시

### 정상 운영 시

* 자원 사용량 추세 분석
* 확장 필요 시점 예측

### 장애 발생 시

* 메트릭 이상 감지
* 로그 기반 원인 추적
* 빠른 복구 판단

---

## 11. 확장 계획

향후 다음과 같은 확장을 고려하고 있다.

* Alertmanager 연동
* Slack / Email 알림
* 로그 장기 보관 스토리지 연계
* 멀티 클러스터 모니터링

---

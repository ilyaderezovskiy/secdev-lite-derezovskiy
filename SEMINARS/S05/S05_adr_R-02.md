# S05 - ADR Template (облегчённый)


## 1) Title

**PII Masking + Data Retention**

## 2) Status

Proposed

## 3) Context

**Risk:** R-02 “PII exposure in logs” (L=4, I=4, Score=16)  
**DFD:** Node Service → Logs (Profile Service, Search Service)  
**NFR:** NFR-007 
**Assumptions:** structured JSON logging; all PII fields identifiable; retention policy ≤90 days; GDPR/CCPA compliance required  

## 4) Decision (что делаем)

Внедряем middleware для маскировки PII в логах и настройку retention для raw PII. Цель — предотвратить утечку конфиденциальных данных и соответствовать требованиям приватности.  
- Param/Policy: Mask PII fields (email, phone, etc.) in all service logs (scope: all profile/search endpoints)  
- Param/Policy: Retention for raw PII ≤90 days (DB + backups)  
- Param/Policy: Logs in structured JSON with correlation_id  
- Notes: boundaries — profile & search service layer; scope: all tenants/users  

## 5) Alternatives (рассмотренные и отклонённые)

- Alt A - Pseudonymization / tokenization instead of masking - higher complexity, integration cost  
- Alt B - Logging without retention policy - non-compliant with GDPR/CCPA 

## 6) Consequences (последствия)

**Положительные:**

+ Снижает риск утечки PII через логи  
+ Соответствует регуляторным требованиям (GDPR/CCPA) 

**Негативные/издержки:**

- Меньше деталей в логах для отладки и расследований  
- Необходимо поддерживать корректность correlation_id для trace  

## 7) DoD / Acceptance (как поймём, что решение принято и работает)

**Given** DTO with PII fields (email, phone)  
**When** logged by service  
**Then** PII fields are masked or removed; retention policy applied 

Checks:
- test: unit+integration tests for masking, e2e log validation  
- log: JSON logs contain correlation_id, no raw PII  
- scan/policy: retention rules applied in DB / log storage  
- metric/SLO: retention duration ≤90 days, zero PII in logs

## 8) Rollback / Fallback

- Disable masking middleware via feature flag  
- Temporarily route logs to secure storage for review  
- Monitor log retention and PII patterns  

## 9) Trace

- DFD: Node Service → Logs  
- STRIDE: I (information disclosure) rows from S04_stride_matrix.md  
- Risk scoring: R-02, Top-2, Score=16  
- NFR: NFR-007  

## 10) Ownership & Dates

**Owner:** team-backend  
**Reviewers:** team-security  
**Date created:** 2025-10-11  
**Last updated:** 2025-10-11 

## 11) Open Questions

- How to handle third-party logging pipelines with PII  
- Edge cases with derived PII (hashed emails, phone fragments)  

## 12) Appendix (опционально)

```yaml
logging:
  format: json
  pii_masking: true
  correlation_id: true
pii_retention:
  raw_data_days: 90

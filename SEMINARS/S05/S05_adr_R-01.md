# S05 - ADR Template

## 1) Title

**JWT TTL + Refresh + Rotation**

## 2) Status

`Proposed`

## 3) Context

Кратко, но по делу (5-10 строк). Укажите:

* Risk: R-01 **“Token reuse / stolen JWT on public edge”** (L=4, I=5, Score=20)  
* DFD: Edge **Internet → API (JWT)**
* NFR: NFR-011
* Assumptions: single public gateway; stateless services; HTTPS only; no MTLS at client side; access tokens short-lived  

## 4) Decision (что делаем)

Внедряем строгую политику управления JWT с коротким TTL и refresh, плюс ротацию ключей. Цель — минимизировать окно атаки при краже токена и предотвратить повторное использование.  
- Param/Policy: Access token TTL = 15 мин (scope: all write endpoints)  
- Param/Policy: Refresh token TTL = 30 дней, single audience (scope: all refresh endpoints)  
- Param/Policy: Key rotation via JWKS, kid included, clock skew ±60s  
- Param/Policy: Logout / blacklist refresh on event (scope: session-level)  
- Notes: boundaries — gateway + auth service layer; scope: all tenants/users  

## 5) Alternatives (рассмотренные и отклонённые)

- Alt A - MTLS for machine clients only - high complexity, dependency on PKI, not feasible for public users  
- Alt B - OAuth Device Flow - increases UX friction for normal users 

## 6) Consequences (последствия)

**Положительные:**

+ Снижает окно атаки при компрометации access token  
+ Контролируемое управление сессиями и refresh, сохраняется observability и correlation_id  

**Негативные/издержки:**

- Дополнительная сложность управления refresh токенами и ключами  
- Возможны edge-case ошибки при clock skew и синхронизации  

## 7) DoD / Acceptance (как поймём, что решение принято и работает)

**Given** expired access token  
**When** POST <write-endpoint>  
**Then** 401 with RFC7807 error  

**Given** valid refresh token  
**When** POST <refresh-endpoint>  
**Then** new access token issued with updated exp 

Checks:
- test: unit+integration auth tests, refresh flow e2e  
- log: correlation_id present, no PII in logs  
- scan/policy: JWT secrets not in repo  
- metric/SLO: unauthorized attempts rate ≤1%  

## 8) Rollback / Fallback

- Disable refresh token rotation via config flag  
- Revert to previous JWKS without downtime  
- Monitor 401 spikes and correlation_id patterns  

## 9) Trace

- DFD: Edge Internet → API (JWT)  
- STRIDE: S (spoofing) rows from S04_stride_matrix.md  
- Risk scoring: R-01, Top-1, Score=20  
- NFR: NFR-011  

## 10) Ownership & Dates

* **Owner:** team-a  
* **Reviewers:** team-security
* **Date created:** 2025-10-11  
* **Last updated:** 2025-10-11  

## 11) Open Questions

- Scope of clock skew tolerance for geographically distributed servers  
- Refresh token revocation strategy for compromised accounts  

## 12) Appendix (опционально)

```yaml
jwt:
  access_ttl: 15m
  refresh_ttl: 30d
  rotation: enabled
  jwks_endpoint: /.well-known/jwks.json
  clock_skew: 60s
logout_blacklist: true

# S03 - Реестр NFR (Given-When-Then)

## Поля реестра (data dictionary)

* **ID** - короткий идентификатор, например `NFR-001`.
* **User Story / Feature** - к какой истории/фиче относится требование (ссылка допустима).
* **Category** - выберите из банка (напр.: `Performance`, `Security-AuthZ/RBAC`, `RateLimiting`, `Privacy/PII`, `Observability/Logging`, …).
* **Requirement (NFR)** - *измеримое* требование (числа/пороги/границы действия).
* **Rationale / Risk** - зачем это нужно, какой риск/ценность покрываем.
* **Acceptance (G-W-T)** - проверяемая формулировка: *Given … When … Then …*.
* **Evidence (test/log/scan/policy)** - чем подтвердим выполнение (тест, лог-шаблон, сканер, политика).
* **Trace (issue/link)** - ссылка на задачу, обсуждение, артефакт.
* **Owner** - ответственный.
* **Status** - `Draft` | `Proposed` | `Approved` | `Implemented` | `Verified`.
* **Priority** - `P1 - High` | `P2 - Medium` | `P3 - Low`.
* **Severity** - `S1 - Critical` | `S2 - Major` | `S3 - Minor`.
* **Tags** - произвольные метки (через запятую).

---

## Таблица реестра

| ID      | User Story / Feature      | Category                 | Requirement (NFR)                                                   | Rationale / Risk                     | Acceptance (G-W-T)                                                                                                    | Evidence (test/log/scan/policy)               | Trace (issue/link) | Owner  | Status   | Priority    | Severity   | Tags              |
| ------- | ------------------------- | ------------------------ | ------------------------------------------------------------------- | ------------------------------------ | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- | ------------------ | ------ | -------- | ----------- | ---------- | ----------------- |
| NFR-001 | As a user I upload avatar | Security-InputValidation | Reject payloads >1 MiB; MIME из allowlist; **extra** поля запрещены | Защита от DoS/грязных входных данных | **Given** тело 2 MiB и неизвестные поля<br>**When** POST `/api/files/avatar`<br>**Then** 413 с RFC7807 + запрет extra | test: `e2e-upload-limit`; policy: schema/size | #123               | team-a | Proposed | P2 - Medium | S2 - Major | limits,validation |
| NFR-002 | As a client I call /api/* | Performance              | P95 ≤ 300 ms при 50 RPS в течение 5 мин                             | UX/SLO                               | **Given** сервис здоров<br>**When** 50 RPS на `/api/*` 5 минут<br>**Then** P95 ≤ 300 ms; error rate ≤ 1%              | test: `load-50rps`; log: latency quantiles    | #124               | team-a | Draft    | P1 - High   | S2 - Major | perf,slo          |

> Продолжайте ниже добавлять строки до достижения **8-10 NFR** (или больше, если нужно):

| ID | User Story / Feature | Category | Requirement (NFR) | Rationale / Risk | Acceptance (G-W-T) | Evidence (test/log/scan/policy) | Trace (issue/link) | Owner | Status | Priority    | Severity   | Tags |
| -- | -------------------- | -------- | ----------------- | ---------------- | ------------------ | ------------------------------- | ------------------ | ----- | ------ | ----------- | ---------- | ---- |
| NFR-003 | As a user I want to search by field/tags to find relevant results | Performance | P95 search query latency ≤ 200 ms at RPS ≤ 20 per node for 5 minutes | Ensuring interface responsiveness and positive user experience. Risk: slowdown of main usage scenario | Given deployed and healthy service<br>When 20 RPS applied for 5 minutes<br>Then P95 ≤ 200 ms; error rate ≤ 1% | load test report; http_server_requests_seconds metric (quantiles); error log | #125 | team-backend | Draft | P1 - High | S2 - Major | performance,slo,latency |
| NFR-004 | As a user I want to search by field/tags to find relevant results | RateLimiting | Limit 20 req/min per token; excess → 429 + Retry-After | Protection from abuse and resource exhaustion (DoS). Risk: increased DB load and performance degradation for all | Given active token<br>When 20+N requests in 60 seconds<br>Then excess requests get 429 and correct Retry-After header | e2e limit test; logs with 429 and Retry-After | #126 | team-backend | Draft | P2 - Medium | S1 - Critical | security,ratelimiting,dos |
| NFR-005 | As a user I want to search by field/tags to find relevant results | Timeouts/Retry/CircuitBreaker | Outbound calls to Search Index API: timeout ≤ 2s, retry ≤ 3 with jitter; circuit-breaker on errors ≥50% over 1 min | Ensuring resilience to temporary unavailability or degradation of search index. Protection from cascading failures. Risk: request hanging and resource exhaustion | Given Search Index API unavailable or responding with delay >2s<br>When service calls it to process GET /api/search<br>Then total wait ≤ 8s; no more than 3 retry attempts with jitter; circuit breaker activates after 50% error threshold over 1 min | HTTP client config; resilience test; circuit breaker logs | #127 | team-backend | Draft | P2 - Medium | S2 - Major | resilience,timeout,retry,circuit-breaker |
| NFR-006 | As a user I want to search by field/tags to find relevant results | Observability/Logging | All incoming requests and outbound calls logged in structured JSON and contain correlation_id at every stage | Ensuring end-to-end request traceability for incident analysis and data flow tracking | Given incoming request to GET /api/search with X-Correlation-ID header<br>When request processed and calls Search Index API<br>Then all logs contain correlation_id and key fields | JSON log verification; search by correlation_id in logging system | #128 | team-backend | Draft | P2 - Medium | S3 - Minor | observability,logging,tracing,correlation-id |
| NFR-007 | As a user I want to edit profile to maintain current data | Privacy/PII | PII fields (email, phone) masked in logs; raw PII data in DB deleted/anonymized after 90 days of account deactivation | Privacy compliance (GDPR, CCPA) and reducing confidential data leakage risk through system logs | Given DTO with email user@example.com<br>When this object is logged<br>Then PII fields masked; retention plan exists and applied within 90 days | PII masking test; retention policy documentation | #129 | team-backend, team-security | Draft | P1 - High | S2 - Major | privacy,pii,compliance,gdpr |
| NFR-008 | As a user I want to edit profile to maintain current data | Data-Integrity | All SQL queries use parameterized queries; string fields normalized to NFC, phones to E.164, dates to UTC | SQL injection protection and data integrity assurance. Eliminating ambiguities in comparison and search | Given input data: name="Café", phone="8 (900) 123-45-67"<br>When PUT /api/profile executed<br>Then DB stores normalized values and uses parameterized statements | Code review; canonicalization tests; ORM/validator settings | #130 | team-backend | Draft | P1 - High | S2 - Major | security,sql-injection,canonicalization,data-quality |
| NFR-009 | As a user I want to edit profile to maintain current data | Security-AuthZ/RBAC | User can read and modify only own profile. Access strictly by user_id from authenticated session | Least privilege principle and user-level data isolation. Risk: horizontal privilege escalation | Given user with ID=123<br>When tries to access GET /api/profile/456<br>Then returns 404 Not Found | Isolation integration test; authorization middleware code | #131 | team-backend | Draft | P1 - High | S1 - Critical | security,authorization,rbac,data-isolation |
| NFR-010 | As a user I want to edit profile to maintain current data | Observability/Logging | All requests to profile endpoints logged in structured JSON format and contain correlation_id | Ensuring end-to-end traceability for debugging and investigating user data changes | Given PUT /api/profile request with X-Correlation-ID=req-123<br>When request processed by service<br>Then logs contain correlation_id and key request fields | JSON log check; correlation_id search evidence | #132 | team-backend | Draft | P2 - Medium | S3 - Minor | observability,logging,tracing,correlation-id |

---

## Памятка по заполнению

* **Измеримость.** В `Requirement` фиксируйте числа и границы (мс, RPS, минуты, MiB, коды 4xx/5xx, CVSS).
* **Проверяемость.** В `Acceptance (G-W-T)` используйте объективные условия и наблюдаемые факты (код ответа, квантиль, наличие заголовка, запись в лог).
* **Связность.** Сверяйте, чтобы NFR не конфликтовали (timeouts vs retry, rate limits vs SLO, privacy vs logging).
* **План проверки.** В `Evidence` укажите, чем это будет подтверждаться позже (test/log/scan/policy). В рамках семинара **реальные артефакты не требуются**.
* **Трассировка.** В `Trace` добавляйте ссылки на Issues/документы, чтобы потом не искать контекст.

---

## После семинара

* Перенесите/доработайте **8-10 утверждённых NFR** (по сути, те же строки) в раздел **NFR** вашего `GRADING/TM.md`.
* На S04-S05 свяжете эти NFR с угрозами (STRIDE) и ADR - по ID.

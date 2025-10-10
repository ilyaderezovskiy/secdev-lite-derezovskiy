# S04 - STRIDE per element (матрица)
---

## Как работать с матрицей

1. Возьмите элементы вашей DFD: **узлы** (API, Service, DB, External) и **рёбра/потоки** (JWT, PII, files, payments).
2. Для **каждого** элемента пройдите буквы **STRIDE** и зафиксируйте **только релевантные** угрозы (0-2 на букву).
3. Для каждой угрозы:

   * кратко опишите **Description** (1-2 предложения, по делу);
   * укажите **NFR link (ID)** из реестра S03 (или `need NFR`, если такого ещё нет);
   * добавьте **Mitigation idea (ADR later)** - короткое имя решения, которое пойдёт в ADR на S05.

---

## Легенда STRIDE

* **S - Spoofing:** подмена идентичности/токена.
* **T - Tampering:** изменение данных/запросов/конфигурации.
* **R - Repudiation:** отрицание действий (нет аудита/трассировки).
* **I - Information disclosure:** утечка конфиденциальных данных (PII/секреты).
* **D - Denial of service:** отказ в обслуживании (ресурсное истощение/«залипание»).
* **E - Elevation of privilege:** повышение привилегий/обход RBAC/тенант-изоляции.

---

## Примеры строк (образец заполнения)

| Element                      | Data/Boundary | Threat (S/T/R/I/D/E) | Description                                            | NFR link (ID)                   | Mitigation idea (ADR later)               |
| ---------------------------- | ------------- | -------------------- | ------------------------------------------------------ | ------------------------------- | ----------------------------------------- |
| Edge: Internet → API         | JWT / public  | S                    | Повтор/подмена токена, reuse истёкшего/украденного JWT | NFR-AuthN, NFR-RateLimit        | JWT TTL+Refresh, rate limit на `/auth/*`  |
| Node: Service                | Logs          | I                    | PII в логах и сообщениях об ошибках                    | NFR-Privacy/PII, NFR-API-Errors | Маскирование PII, RFC7807 без стэктрейсов |
| Edge: Service → External API | HTTP/gRPC     | D                    | Залипание без timeout/retry/circuit breaker            | NFR-Timeouts/Retry/CB           | Timeout≤2s, retry≤3 с джиттером, CB       |

> После заполнения матрицы **перенесите уникальные/объединённые риски** в `S04_risk_scoring.md` для приоритизации L×I (1-5) и выбора Top-5.

---

## Матрица для заполнения

| Element | Data/Boundary | Threat (S/T/R/I/D/E) | Description | NFR link (ID) | Mitigation idea (ADR later) |
| ------- | ------------- | -------------------- | ----------- | ------------- | --------------------------- |
| **Edge: Internet → API Gateway** | HTTPS / JWT / PII | **D** | Массовые запросы без rate limit вызывают DoS. | **NFR-004 (RateLimiting)** | API rate limiting (token-bucket) |
| | | **S** | Подмена или повтор JWT-токена (replay/stolen token). | need NFR (AuthN/JWT) | Short-lived JWT + refresh + TLS pinning |
| **Node: API Gateway** | Request routing / DTO | **T** | Изменение схемы или неожиданные поля в payload. | **NFR-001 (Security-InputValidation)** | Schema validation + size limits |
| | | **R** | Нет логов корреляции при проксировании → невозможен аудит. | **NFR-006 (Observability/Logging)**, **NFR-010 (Observability/Logging)** | Correlation-ID propagation + JSON structured logs |
| **Edge: API Gateway → Search Service** | DTO / Search Query | **D** | Медленные внешние вызовы создают зависание worker-потоков. | **NFR-005 (Timeouts/Retry/CircuitBreaker)** | Timeout ≤2s, retry≤3, circuit-breaker |
| | | **I** | Логи содержат поля запроса (например, user input) без очистки. | **NFR-007 (Privacy/PII)** | Sensitive field filtering in logs |
| **Edge: API Gateway → Profile Service** | DTO / PII | **I** | Возможна передача PII без маскировки между сервисами. | **NFR-007 (Privacy/PII)**, **NFR-008 (Data-Integrity)** | PII masking + schema validation |
| | | **E** | API Gateway ошибочно передаёт чужой user_id → утечка данных. | **NFR-009 (Security-AuthZ/RBAC)** | AuthZ check on service boundary |
| **Node: Search Service** | Logic / Cache | **T** | Возможна подмена параметров поиска через query injection. | **NFR-008 (Data-Integrity)** | Parametrized queries + input normalization |
| | | **D** | Высокая нагрузка на индекс → деградация SLA поиска. | **NFR-003 (Performance)** | Performance SLO monitoring, autoscale |
| **Node: Profile Service** | Profile data / PII | **I** | Утечка email/phone в логах и исключениях. | **NFR-007 (Privacy/PII)** | PII masking middleware |
| | | **E** | Попытка редактировать чужой профиль (горизонтальная эскалация). | **NFR-009 (Security-AuthZ/RBAC)** | RBAC check on user_id, scoped tokens |
| **Edge: Search Service → External Search API** | HTTP / JSON | **D** | Внешний API зависает или медленный → блокировка пула соединений. | **NFR-005 (Timeouts/Retry/CircuitBreaker)** | Timeout, retry w/ jitter, circuit breaker |
| | | **I** | Ответ от внешнего API содержит неожиданные PII-поля. | need NFR (External API Sanitization) | Response schema validation + field filter |
| **Node: Database** | SQL / Storage | **T** | SQL-инъекции или некорректная нормализация строковых полей. | **NFR-008 (Data-Integrity)** | ORM param-binding + NFC normalization |
| | | **I** | Хранение PII без шифрования и с истёкшим retention. | **NFR-007 (Privacy/PII)** | At-rest encryption + auto deletion 90d |
| **Edge: Services → Database** | SQL / Auth check | **E** | Сервис обращается к данным другого пользователя. | **NFR-009 (Security-AuthZ/RBAC)** | Row-level security / tenant isolation |
| | | **R** | Нет журналирования DML-операций (аудит изменений профиля). | need NFR (AuditTrail) | DB audit log / CDC stream |

---

## Самопроверка (быстро)

* [+] Пройдены **все узлы и рёбра** вашей DFD.
* [+] У угроз стоят **NFR link (ID)** или пометка `need NFR`.
* [_] У каждой угрозы есть понятная **Mitigation idea** (будущий ADR).
* [+] Нет «воды»: кратко, по делу, без оценок L/I (они - в `S04_risk_scoring.md`).

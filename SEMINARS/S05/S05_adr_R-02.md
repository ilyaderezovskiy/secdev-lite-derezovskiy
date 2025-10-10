# S05 - ADR Template (облегчённый)

> Использование: создавайте **по одному файлу на решение**.
> Рекомендуемое имя файла у студента: `SEMINARS/S05/S05_ADR_<slug>.md`
> Статусы: `Proposed | Approved | Superseded`

---

## 1) Title

Короткое имя решения (например, **“JWT TTL + Refresh + Rotation”**).

## 2) Status

`Proposed | Approved | Superseded`
Кто согласовал (если есть) и когда.

## 3) Context

Кратко, но по делу (5-10 строк). Укажите:

* **Risk-ID** (из `S04_risk_scoring.md`), его **L×I (1-5)** и почему он в топе.
* **DFD элемент/поток** (из `S04_dfd.md`).
* Связанные **NFR-ID** (из реестра S03).
* Ограничения и предпосылки (архитектура, стек, регуляторные требования).

Пример формата:

```text
Risk: R-03 “Token reuse on public edge” (L=4, I=4, Score=16)
DFD: Edge Internet→API (JWT)
NFR: NFR-AuthN, NFR-RateLimit, NFR-API-Errors
Assumptions: single public gateway; stateless services; no MTLS at client side
```

## 4) Decision (что делаем)

Суть решения одним абзацем. Затем - конкретика параметров/политик списком:

* Параметр/политика 1 (значение/порог, область действия)
* Параметр/политика 2 …
* …

> Здесь же укажите **границы действия** (какие эндпойнты/роли/тенанты) и **этап/слой** (gateway, сервис, БД, политика).

## 5) Alternatives (рассмотренные и отклонённые)

Короткие пункты: альтернатива → почему не выбрали.

* Alt A - причина отклонения (cost/complexity/low impact/latency/зависимости)
* Alt B - причина отклонения
* (опционально) Alt C - причина

## 6) Consequences (последствия)

**Положительные:**

* эффект 1
* эффект 2

**Негативные/издержки:**

* компромисс 1
* компромисс 2

## 7) DoD / Acceptance (как поймём, что решение принято и работает)

Сформулируйте проверяемо, можно в стиле **G-W-T**:

* **Given** …
* **When** …
* **Then** …

И добавьте измеримые критерии (не реализация, а **план проверок**):

* test: какой автотест/контракт-тест должен пройти;
* log: какой паттерн/поле появится в логах (напр., `correlation_id`, отсутствие PII);
* scan/policy: какой чек/правило будет зелёным;
* metric/SLO: какой порог **≤ / ≥** (латентность, ошибки, 429 rate и т. п.).

> Реальные артефакты лягут позже в TM/DV/DS. Здесь фиксируем **что именно** будет проверять принятие решения.

## 8) Rollback / Fallback

Как откатить безопасно (флаг/конфиг/политика), на что переключиться, как мониторить деградацию.

## 9) Trace

Ссылки на:

* `S04_dfd.md` (фрагмент/узел/ребро)
* `S04_stride_matrix.md` (строки, откуда пришёл риск)
* `S04_risk_scoring.md` (Risk-ID, место в Top-5)
* Связанные **NFR-ID** (из S03)
* Задачи/Issues по внедрению

## 10) Ownership & Dates

* **Owner** (ответственный)
* **Reviewers** (если есть)
* **Date created** / **Last updated**

## 11) Open Questions

Список вопросов, не блокирующих `Proposed`, но требующих уточнения до `Approved`.

## 12) Appendix (опционально)

Короткие выдержки конфигов/политик/псевдокода или ссылка на отдельный файл.

> Не прикладывайте изображения; всё в тексте/код-фрагментах.

---

### Мини-шаблон для копипаста

```md
# ADR: <Title>
Status: Proposed | Approved | Superseded

## Context
Risk: <R-ID> (L=<1-5>, I=<1-5>, Score=<L×I>)
DFD: <element/edge>
NFR: <NFR-IDs>
Assumptions: <short bullets>

## Decision
<one-paragraph summary>
- Param/Policy: <name>=<value> (scope: <endpoints/roles/tenants>)
- Param/Policy: ...
- Notes: <boundaries/layer>

## Alternatives
- <Alt A> - <why not>
- <Alt B> - <why not>

## Consequences
+ <positive effect>
+ <positive effect>
- <trade-off>
- <trade-off>

## DoD / Acceptance
Given <precondition>
When <action/load/event>
Then <observable, measurable result>
Checks:
- test: <name/area>
- log: <pattern/field>
- scan/policy: <rule>
- metric/SLO: <threshold>

## Rollback / Fallback
<how to toggle/rollback safely, monitoring plan>

## Trace
- DFD: <link/anchor>
- STRIDE: <row refs>
- Risk scoring: <R-ID, rank>
- NFR: <IDs>
- Issues: <links>

## Ownership & Dates
Owner: <name>  
Reviewers: <names>  
Date created: <YYYY-MM-DD>  
Last updated: <YYYY-MM-DD>

## Open Questions
- <question 1>
- <question 2>

## Appendix (optional)
<tiny config/policy snippet or link>
```

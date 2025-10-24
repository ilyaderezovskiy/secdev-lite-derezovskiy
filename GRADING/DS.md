# DS - Отчёт «DevSecOps-сканы и харднинг»

> Этот файл - **индивидуальный**. Его проверяют по **rubric_DS.md** (5 критериев × {0/1/2} → 0-10).
> Подсказки помечены `TODO:` - удалите после заполнения.
> Все доказательства/скрины кладите в **EVIDENCE/** и ссылайтесь на конкретные файлы/якоря.

---

## 0) Мета

- **Проект (опционально BYO):** учебный шаблон (https://github.com/ilyaderezovskiy/secdev-seed-s09-s12)
- **Версия (commit/date):** 1.0.0 / 2025-10-17
- **Кратко (1-2 предложения):** TODO: что сканируется и какие меры харднинга планируются

---

## 1) SBOM и уязвимости зависимостей (DS1)

- **Инструмент/формат:** Syft/Grype; CycloneDX JSON
- **Как запускал:**

  ```bash
  syft dir:. -o cyclonedx-json > EVIDENCE/sbom-2024-06-15.json
  grype sbom:EVIDENCE/sbom-2024-06-15.json --fail-on high -o json > EVIDENCE/deps-2024-06-15.json
  ```

- **Отчёты:** [`EVIDENCE/09/sbom.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sbom.json), [`EVIDENCE/09/sca_report.json`]([sca_report.json](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sca_report.json)), [`EVIDENCE/09/sca_summary.md`]([sca_summary.md](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sca_summary.md))
- **Actions:** [ссылка на успешный job](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/actions/runs/18783223550)
- **Выводы (кратко):** 
  - Найдено: 0 Critical, 0 High, 3 Medium уязвимостей
  - Состояние: отсутствуют критические и высокие уязвимости
- **Действия:** Обновления не требуются - отсутствуют High/Critical уязвимости
- **Гейт по зависимостям:** Critical=0; High=0

---

## 2) SAST и Secrets (DS2)

### 2.1 SAST

- **Инструмент/профиль:** semgrep
- **Как запускал:**

  ```bash
  semgrep --config p/ci --severity=high --error --json --output EVIDENCE/sast-2024-06-15.json
  semgrep --config auto --sarif --output EVIDENCE/sast-2024-06-15.sarif
  ```

- **Actions:** [ссылка на успешный job](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/actions/runs/18783590408)
- **Отчёт:** [`EVIDENCE/S10/semgrep.sarif`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S10/semgrep.sarif)
- **Выводы:** 
  - Результат: 0 предупреждений - код не содержит выявляемых статических уязвимостей
  - Качество кода соответствует стандартам безопасности по проверяемым правилам
  - Ложные срабатывания отсутствуют

### 2.2 Secrets scanning

- **Инструмент:** gitleaks
- **Как запускал:**

  ```bash
  gitleaks detect --no-git --report-format json --report-path EVIDENCE/secrets-2024-06-15.json
  gitleaks detect --log-opts="--all" --report-format json --report-path EVIDENCE/secrets-2024-06-15-history.json
  ```

- **Actions:** [ссылка на успешный job](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/actions/runs/18783590408)
- **Отчёт:** [`EVIDENCE/S10/gitleaks.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S10/gitleaks.json)
- **Выводы:** 
  - Результат: 0 находок - в коде и истории коммитов не обнаружено секретов
  - Меры: дополнительные действия не требуются, состояние соответствует политике безопасности

---

## 3) DAST **или** Policy (Container/IaC) (DS3)

> Для «1» достаточно одного из классов; на «2» - желательно оба **или** один глубже (настроенный профиль/таргет).

### Вариант A - DAST (лайт)

- **Инструмент/таргет:** TODO (локальный стенд/демо-контейнер допустим)
- **Как запускал:**

  ```bash
  zap-baseline.py -t http://127.0.0.1:8080 -m 3 \
    -r EVIDENCE/dast-YYYY-MM-DD.html -J EVIDENCE/dast-YYYY-MM-DD.json
  ```

- **Отчёт:** `EVIDENCE/dast-YYYY-MM-DD.pdf#alert-...`
- **Выводы:** TODO: 1-2 meaningful наблюдения

### Вариант B - Policy / Container / IaC

- **Инструмент(ы):** TODO (trivy config / checkov / conftest и т.п.)
- **Как запускал:**

  ```bash
  trivy image --severity HIGH,CRITICAL --exit-code 1 <image:tag> > EVIDENCE/policy-YYYY-MM-DD.txt
  trivy config . --severity HIGH,CRITICAL --exit-code 1 --format table > EVIDENCE/trivy-YYYY-MM-DD.txt
  checkov -d . -o cli > EVIDENCE/checkov-YYYY-MM-DD.txt
  ```

- **Отчёт(ы):** `EVIDENCE/policy-YYYY-MM-DD.txt`, `EVIDENCE/trivy-YYYY-MM-DD.txt`, …
- **Выводы:** TODO: какие правила нарушены/исправлены

---

## 4) Харднинг (доказуемый) (DS4)

Отметьте **реально применённые** меры, приложите доказательства из `EVIDENCE/`.

- [ ] **Контейнер non-root / drop capabilities** → Evidence: `EVIDENCE/policy-YYYY-MM-DD.txt#no-root`
- [ ] **Rate-limit / timeouts / retry budget** → Evidence: `EVIDENCE/load-after.png`
- [ ] **Input validation** (типы/длины/allowlist) → Evidence: `EVIDENCE/sast-YYYY-MM-DD.*#input`
- [ ] **Secrets handling** (нет секретов в git; хранилище секретов) → Evidence: `EVIDENCE/secrets-YYYY-MM-DD.*`
- [ ] **HTTP security headers / CSP / HTTPS-only** → Evidence: `EVIDENCE/security-headers.txt`
- [ ] **AuthZ / RLS / tenant isolation** → Evidence: `EVIDENCE/rls-policy.txt`
- [ ] **Container/IaC best-practice** (минимальная база, readonly fs, …) → Evidence: `EVIDENCE/trivy-YYYY-MM-DD.txt#cfg`

> Для «1» достаточно ≥2 уместных мер с доказательствами; для «2» - ≥3 и хотя бы по одной показать эффект «до/после».

---

## 5) Quality-gates и проверка порогов (DS5)

- **Пороговые правила (словами):**  
  Примеры: «SCA: Critical=0; High≤1», «SAST: Critical=0», «Secrets: 0 истинных находок», «Policy: Violations=0».
- **Как проверяются:**  
  - Ручной просмотр (какие файлы/строки) **или**  
  - Автоматически:  (скрипт/job, условие fail при нарушении)

    ```bash
    SCA: grype ... --fail-on high
    SAST: semgrep --config p/ci --severity=high --error
    Secrets: gitleaks detect --exit-code 1
    Policy/IaC: trivy (image|config) --severity HIGH,CRITICAL --exit-code 1
    DAST: zap-baseline.py -m 3 (фейл при High)
    ```

- **Ссылки на конфиг/скрипт (если есть):**

  ```bash
  GitHub Actions: .github/workflows/security.yml (jobs: sca, sast, secrets, policy, dast)
  или GitLab CI: .gitlab-ci.yml (stages: security; jobs: sca/sast/secrets/policy/dast)
  ```

---

## 6) Триаж-лог (fixed / suppressed / open)

| ID/Anchor       | Класс     | Severity | Статус     | Действие | Evidence                               | Ссылка на фикс/исключение         | Комментарий / owner / expiry |
|-----------------|-----------|----------|------------|----------|----------------------------------------|-----------------------------------|------------------------------|
| CVE-2024-XXXX   | SCA       | High     | fixed      | bump     | `EVIDENCE/deps-YYYY-MM-DD.json#CVE`    | `commit abc123`                   | -                            |
| ZAP-123         | DAST      | Medium   | suppressed | ignore   | `EVIDENCE/dast-YYYY-MM-DD.pdf#123`     | `EVIDENCE/suppressions.yml#zap`   | FP; owner: ФИО; expiry: 2025-12-31 |
| SAST-77         | SAST      | High     | open       | backlog  | `EVIDENCE/sast-YYYY-MM-DD.*#77`        | issue-link                        | план фикса в релизе N        |

> Для «2» по DS5 обязательно указывать **owner/expiry/обоснование** для подавлений.

---

## 7) Эффект «до/после» (метрики) (DS4/DS5)

| Контроль/Мера | Метрика                 | До   | После | Evidence (до), (после)                          |
|---------------|-------------------------|-----:|------:|-------------------------------------------------|
| Зависимости   | #Critical / #High (SCA) | TODO | 0 / ≤1| `EVIDENCE/deps-before.json`, `deps-after.json`  |
| SAST          | #Critical / #High       | TODO | 0 / ≤1| `EVIDENCE/sast-before.*`, `sast-after.*`        |
| Secrets       | Истинные находки        | TODO | 0     | `EVIDENCE/secrets-*.json`                       |
| Policy/IaC    | Violations              | TODO | 0     | `EVIDENCE/checkov-before.txt`, `checkov-after.txt` |

---

## 8) Связь с TM и DV (сквозная нитка)

- **Закрываемые угрозы из TM:** TODO: T-001, T-005, … (ссылки на таблицу трассировки TM)
- **Связь с DV:** TODO: какие сканы/проверки встроены или будут встраиваться в pipeline

---

## 9) Out-of-Scope

- TODO: что сознательно не сканировалось сейчас и почему (1-3 пункта)

---

## 10) Самооценка по рубрике DS (0/1/2)

- **DS1. SBOM и SCA:** [ ] 0 [ ] 1 [ ] 2  
- **DS2. SAST + Secrets:** [ ] 0 [ ] 1 [ ] 2  
- **DS3. DAST или Policy (Container/IaC):** [ ] 0 [ ] 1 [ ] 2  
- **DS4. Харднинг (доказуемый):** [ ] 0 [ ] 1 [ ] 2  
- **DS5. Quality-gates, триаж и «до/после»:** [ ] 0 [ ] 1 [ ] 2  

**Итог DS (сумма):** __/10

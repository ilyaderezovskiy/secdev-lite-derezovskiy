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
  syft dir:. -o cyclonedx-json > EVIDENCE/S09/sbom.json
  grype sbom:/work/EVIDENCE/S09/sbom.json --fail-on high -o json > EVIDENCE/S09/sca_report.json
  ```

- **Отчёты:** [`EVIDENCE/09/sbom.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sbom.json), [`EVIDENCE/09/sca_report.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sca_report.json), [`EVIDENCE/09/sca_summary.md`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sca_summary.md)
- **Actions:** [ссылка на успешный job](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/actions/runs/18783223550)
- **Выводы (кратко):** 
  - Найдено: 0 Critical, 0 High, 3 Medium уязвимостей
  - Состояние: отсутствуют критические и высокие уязвимости
- **Действия:** Обновить версию Jinja2 в [requirements.txt](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/blob/main/requirements.txt)
  
   ```bash
  jinja2==3.1.4 -> jinja2==3.1.6
  ```
- **Выводы после исправления (кратко):** 
  - Найдено: 0 уязвимостей
  - Состояние: отсутствуют
  - [`EVIDENCE/09/sca_report_after.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sca_report_after.json)
  - [ссылка на успешный job](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/actions/runs/18794105977)

- **Гейт по зависимостям:** Critical=0; High=0

---

## 2) SAST и Secrets (DS2)

### 2.1 SAST

- **Инструмент/профиль:** semgrep
- **Как запускал:**

  ```bash
  semgrep --config auto --sarif --output EVIDENCE/S10/semgrep.sarif
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
  gitleaks detect --no-git --report-format json --report-path EVIDENCE/S10/gitleaks.json
  gitleaks detect --log-opts="--all" --report-format json --report-path EVIDENCE/S10/gitleaks-history.json
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

- **Инструмент/таргет:** zap
- **Как запускал:**

  ```bash
  zap-baseline.py -t http://127.0.0.1:8080 -m 3 \
    -r EVIDENCE/S11/zap_baseline.html -J EVIDENCE/S11/zap_baseline.json
  ```
  
- **Отчёт:** [`EVIDENCE/S11/zap_baseline.html`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/zap_baseline.html), [`EVIDENCE/S11/zap_baseline.json-d.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/zap_baseline.json-d.json)
- **Выводы:** 

  1. **Проблема:** Missing Anti-clickjacking Header (Medium ×4)
  
  **Риск:** Возможность встраивания приложения во фреймы на вредоносных сайтах

  **Детали:** Отсутствуют заголовки X-Frame-Options и Content-Security-Policy с директивой frame-ancestors

  **Решение:** В [main.py](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/blob/main/app/main.py) добавлены security headers middleware

  ```bash
  class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        
        # Security headers
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Content-Security-Policy"] = "default-src 'self'; script-src 'self' 'unsafe-inline'"
        response.headers["Permissions-Policy"] = "camera=(), microphone=(), location=()"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        
        # Cache control for sensitive endpoints
        if request.url.path in ["/", "/echo", "/?q="]:
            response.headers["Cache-Control"] = "no-cache, no-store, must-revalidate"
            response.headers["Pragma"] = "no-cache"
            response.headers["Expires"] = "0"
        
        return response
  ```

 **Отчёт после изменений:** [`EVIDENCE/S11/zap_baseline_after.html`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/zap_baseline_after.html), [`EVIDENCE/S11/zap_baseline_after.json-d.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/zap_baseline_after.json-d.json)
  
  **Результат:**
  
  - Исчезли: Clickjacking, X-Content-Type, основные CSP ошибки
  
  - Появилось: "Non-Storable Content" - значит cache-control заголовки работают
  
  - Риски снижены: с 8 предупреждений до 3

  2. **Проблема:** User Controllable HTML Element Attribute (Informational ×1)
  
  **Риск:** Возможность внедрения скриптов через параметр 'q'

  **Детали:** Пользовательский ввод напрямую попадает в value атрибут input элемента

  **Пример:** http://localhost:8080/?q=ZAP -> `<input value="zap">`

  - Content Security Policy не настроен (Medium ×4)
  - X-Content-Type-Options отсутствует (Low ×5)
  - Кэширование чувствительного контента (Informational ×8)

### Вариант B - Policy / Container / IaC

- **Инструмент(ы):** checkov
- **Как запускал:**

  ```bash
  checkov -d . -o cli > EVIDENCE/S12/checkov.json
  ```

- **Отчёт(ы):** [`EVIDENCE/S12/checkov.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/checkov.json), [`EVIDENCE/S12/hadolint.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/hadolint.json)
- **Выводы:**
  - Высокий риск компрометации
  - Потенциальное воздействие на кластер

- **Исправления:**
  1. Запуск от non-root пользователя
     
  Было: runAsUser: 0 (root)
  
  Стало: runAsUser: 1001 + runAsNonRoot: true
  
  Эффект: Устранен риск привилегированной эксплуатации контейнера

  3. Использование фиксированного тега образа
  
  Было: s09s12-app:latest

  Стало: s09s12-app:v1.2.3

  Эффект: Гарантирована воспроизводимость и предотвращены неожиданные обновления

  4. Блокировка эскалации привилегий
  
  Было: Не указано (разрешено по умолчанию)

  Стало: allowPrivilegeEscalation: false

  Эффект: Заблокированы попытки повышения привилегий внутри контейнера

  5. Удаление Linux capabilities
  
  Было: Все capabilities доступны

  Стало: capabilities: drop: ["ALL"]

  Эффект: Радикальное сокращение поверхности атаки

  6. Включение seccomp профиля
  
  Было: Не указано

  Стало: seccompProfile: type: RuntimeDefault

  Эффект: Ограничение системных вызовов на уровне ядра

  7. Установка лимитов ресурсов
  
  Было: resources: {} (без ограничений)

  Стало: Четко определенные requests/limits

  - CPU: requests 100m, limits 200m

  - Memory: requests 64Mi, limits 128Mi

  Эффект: Предотвращение истощения ресурсов узла и обеспечение стабильности

- **Выводы после исправления:**
  - Все проверки пройдены успешно - из 82 проверок безопасности для Kubernetes 82 PASSED, 0 FAILED. Все критически важные аспекты защищены: изоляция, ограничение прав, управление ресурсами и безопасность образов.

- **Actions:** [ссылка на успешный job](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/actions/runs/18794221901)

---

## 4) Харднинг (доказуемый) (DS4)

Отметьте **реально применённые** меры, приложите доказательства из `EVIDENCE/`.

- [x] **Контейнер non-root / drop capabilities** → Evidence: [`EVIDENCE/S12/non-root_IaC.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/non-root_IaC.txt)
- [ ] **Rate-limit / timeouts / retry budget** → Evidence: `EVIDENCE/load-after.png`
- [x] **Input validation** (типы/длины/allowlist) → Evidence: [`EVIDENCE/S11/input-validation.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/input-validation.txt)
- [ ] **Secrets handling** (нет секретов в git; хранилище секретов) → Evidence: 
- [x] **HTTP security headers / CSP / HTTPS-only** → Evidence: [`EVIDENCE/S11/secrets_handling.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/security-headers.txt)
- [ ] **AuthZ / RLS / tenant isolation** → Evidence: `EVIDENCE/rls-policy.txt`
- [x] **Container/IaC best-practice** (минимальная база, readonly fs, …) → [`EVIDENCE/S12/non-root_IaC.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/non-root_IaC.txt)

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

**Что сознательно не сканировалось сейчас и почему (1-3 пункта):**
- Active DAST (риск нарушения работы тестового окружения)
- Infrastructure as Code (полное IaC сканирование требует реальной инфраструктуры)

---

## 10) Самооценка по рубрике DS (0/1/2)

- **DS1. SBOM и SCA:** [ ] 0 [ ] 1 [+] 2  
- **DS2. SAST + Secrets:** [ ] 0 [+] 1 [ ] 2  
- **DS3. DAST или Policy (Container/IaC):** [ ] 0 [ ] 1 [+] 2  
- **DS4. Харднинг (доказуемый):** [ ] 0 [ ] 1 [+] 2  
- **DS5. Quality-gates, триаж и «до/после»:** [ ] 0 [ ] 1 [ ] 2  

**Итог DS (сумма):** __/10

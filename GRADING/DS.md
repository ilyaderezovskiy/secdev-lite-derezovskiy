# DS - Отчёт «DevSecOps-сканы и харднинг»

> Этот файл - **индивидуальный**. Его проверяют по **rubric_DS.md** (5 критериев × {0/1/2} → 0-10).
> Подсказки помечены `TODO:` - удалите после заполнения.
> Все доказательства/скрины кладите в **EVIDENCE/** и ссылайтесь на конкретные файлы/якоря.

---

## 0) Мета

- **Проект (опционально BYO):** учебный шаблон (https://github.com/ilyaderezovskiy/secdev-seed-s09-s12)
- **Версия (commit/date):** 1.0.0 / 2025-10-27
- **Кратко (1-2 предложения):** SAST, Secrets, SCA, SBOM и пассивный DAST для проверки кода, зависимостей и базовых security headers

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
  - [`EVIDENCE/09/sca_summary_after.md`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sca_summary_after.md)
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
  - НЕ ПРОЙДЕНО: 17 проверок
  - Наиболее критичны запуск от root, отсутствие ограничений ресурсов и health checks

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
 
- **Отчёт(ы) после исправления:** [`EVIDENCE/S12/checkov_after.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/checkov_after.json), [`EVIDENCE/S12/hadolint_after.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/hadolint_after.json)

- **Actions:** [ссылка на успешный job](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/actions/runs/18794221901)

---

## 4) Харднинг (доказуемый) (DS4)

Отметьте **реально применённые** меры, приложите доказательства из `EVIDENCE/`.

- [x] **Контейнер non-root / drop capabilities** → Evidence: [`EVIDENCE/S12/non-root_IaC.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/non-root_IaC.txt)
- [ ] **Rate-limit / timeouts / retry budget** → Evidence: `EVIDENCE/load-after.png`
- [x] **Input validation** (типы/длины/allowlist) → Evidence: [`EVIDENCE/S11/input-validation.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/input-validation.txt)
- [x] **Secrets handling** (нет секретов в git; хранилище секретов) → Evidence: [`EVIDENCE/S10/gitleaks.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S10/gitleaks.json), [`EVIDENCE/S10/semgrep.sarif
`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S10/semgrep.sarif)
- [x] **HTTP security headers / CSP / HTTPS-only** → Evidence: [`EVIDENCE/S11/secrets_handling.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/security-headers.txt)
- [ ] **AuthZ / RLS / tenant isolation** → Evidence: `EVIDENCE/rls-policy.txt`
- [x] **Container/IaC best-practice** (минимальная база, readonly fs, …) → [`EVIDENCE/S12/non-root_IaC.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/non-root_IaC.txt)

> Для «1» достаточно ≥2 уместных мер с доказательствами; для «2» - ≥3 и хотя бы по одной показать эффект «до/после».

---

## 5) Quality-gates и проверка порогов (DS5)

- **Пороговые правила (словами):**  

  - SCA: Critical=0; High≤1; Medium≤3
  - SAST: Critical=0; High=0
  - Secrets: Gitleaks length=0
  - DAST: Critical=0; High=0; Medium≤2
  - Policy/IaC: Critical=0; High=0
 
- **Как проверяются:**  
  Ручной просмотр:
    
  - SCA: EVIDENCE/S09/sca_summary_after.md - просмотр severity counts
  - SAST: EVIDENCE/S10/semgrep.sarif - проверка массива results
  - Secrets: EVIDENCE/S10/gitleaks.json - проверка пустого массива
  - DAST: EVIDENCE/S11/zap_baseline_after.html - таблица Summary of Alerts
  - Policy/IaC: EVIDENCE/S12/checkov_after.json - проверка failed_checks
    
  Автоматически:  (скрипт/job, условие fail при нарушении)

    ```bash
    # SCA
    grype sbom:EVIDENCE/S09/sbom.json --fail-on high -o json

    # SAST  
    semgrep --config p/ci --severity=high --error --json
    
    # Secrets
    gitleaks detect --no-git --exit-code 1 --report-format json
    
    # DAST
    zap-baseline.py -t http://localhost:8080 -m 3 -I  # Fail on High
    
    # Policy/IaC
    checkov --directory . --framework terraform --fail-on HIGH
    ```

- **Ссылки на конфиг/скрипт (если есть):**

  Пороговые проверки встроены в GitHub Actions workflow проекта, выполняются вручную/при пуше/на PR [.github/workflows](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/tree/main/.github/workflows)

---

## 6) Триаж-лог (fixed / suppressed / open)

| Класс     | Severity | Статус     | Действие | Evidence                               | Ссылка на фикс/исключение         | Комментарий / owner / expiry |
|-----------|----------|------------|----------|----------------------------------------|-----------------------------------|------------------------------|
| SCA       | Medium     | fixed      | bump     | [`EVIDENCE/S09/sca_summary_after.md`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sca_summary_after.md)     | [requirements.txt](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/blob/main/requirements.txt) | Обновление для sandbox escape фикса, attr filter bypass исправлен |
| DAST      | Medium   | fixed | security headers   | [`EVIDENCE/S11/zap_baseline_after.html`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/zap_baseline_after.html), [`EVIDENCE/S11/secrets_handling.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/security-headers.txt) | [app/main.py](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/blob/main/app/main.py) | Добавлен CSP header, добавлен X-Frame-Options: DENY |
| SAST      | Medium | fixed | input validation  | [`EVIDENCE/S11/zap_baseline_after.html`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/zap_baseline_after.html), [`EVIDENCE/S11/input-validation.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/input-validation.txt) | [app/main.py](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/blob/main/app/main.py) | Экранирование пользовательского ввода |
| Policy/IaC      | Medium | fixed | hardening | [`EVIDENCE/S12/checkov_after.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/checkov_after.json), [`EVIDENCE/S12/non-root_IaC.txt`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/non-root_IaC.txt) | [k8s/deploy.yaml](https://github.com/ilyaderezovskiy/secdev-seed-s09-s12/blob/main/iac/k8s/deploy.yaml) | Добавлен non-root, dropped caps |

---

## 7) Эффект «до/после» (метрики) (DS4/DS5)

| Контроль/Мера | Метрика                 | До   | После | Evidence (до), (после)                          |
|---------------|-------------------------|-----:|------:|-------------------------------------------------|
| Зависимости   | #Critical / #High / #Medium (SCA) | 0 / 0 / 3 | 0 / 0 / 0 | [`EVIDENCE/S09/sca_summary.md`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sca_summary.md), [`EVIDENCE/S09/sca_summary_after.md`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S09/sca_summary_after.md)  |
| SAST          | #Low       | 19 | 10 | [`EVIDENCE/S11/zap_baseline.html`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/zap_baseline.html), [`EVIDENCE/S11/zap_baseline_after.html`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S11/zap_baseline_after.html)        |
| Secrets       | Истинные находки        | 0 | 0     | [`EVIDENCE/S10/gitleaks.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S10/gitleaks.json)                       |
| Policy/IaC    | Violations              | 17 | 0     | [`EVIDENCE/S12/checkov.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/checkov.json), [`EVIDENCE/S12/checkov_after.json`](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S12/checkov_after.json) |

---

## 8) Связь с TM и DV (сквозная нитка)

- **Закрываемые угрозы из TM:**
  
  **1. T04: SQL-инъекция/ошибки валидации (R-04) - ЗАКРЫТА**

  Что уже есть:
  - В коде используются параметризованные запросы через ORM/SQLAlchemy
  - Есть тесты на SQL-инъекции в tests/
  - Pydantic валидация входных данных
  - Все тесты проходят (18/18 в CI)
  
  **2. T02: Утечка PII в логах/ошибках (R-02) - ЧАСТИЧНО ЗАКРЫТА**

  Что уже есть:
  - Pre-commit hooks проверяют секреты в коде
  - CI secrets scanning (grep-secrets.txt)
  - Маскирование секретов в логах CI (::add-mask::)
  
  **3. T01: Кража/повторное использование JWT-токена - ЧАСТИЧНО ЗАКРЫТА**

  Что уже есть:
  - Secrets management в CI (переменные окружения)
  - Проверка отсутствия секретов в коде
  
  Что нужно добавить:
  - Специфичные проверки JWT конфигурации
  - Тесты на TTL токенов

- **Связь с DV:** Уже встроенные проверки:
  
  **-   Pre-commit hooks (DV5)**
    1.   Проверка секретов в коде
    2.   Форматирование и линтинг
  **-   Secrets scanning в CI (DV3)**

  ```bash
  yaml
  - name: Security scan for secrets
    run: |
      git grep -nE 'AKIA|SECRET|api[_-]?key|token=|password=' > EVIDENCE/grep-secrets.txt || true
  ```

  **-   Security tests в pytest (DV3)**
    1.   Тесты на XSS, SQL-инъекции
    2.   Проверки авторизации/аутентификации

---

## 9) Out-of-Scope

**Что сознательно не сканировалось сейчас и почему (1-3 пункта):**
- Active DAST (риск нарушения работы тестового окружения)
- Infrastructure as Code (полное IaC сканирование требует реальной инфраструктуры)

---

## 10) Самооценка по рубрике DS (0/1/2)

- **DS1. SBOM и SCA:** [ ] 0 [ ] 1 [+] 2  
- **DS2. SAST + Secrets:** [ ] 0 [ ] 1 [+] 2  
- **DS3. DAST или Policy (Container/IaC):** [ ] 0 [ ] 1 [+] 2  
- **DS4. Харднинг (доказуемый):** [ ] 0 [ ] 1 [+] 2  
- **DS5. Quality-gates, триаж и «до/после»:** [ ] 0 [ ] 1 [+] 2  

**Итог DS (сумма):** __/10

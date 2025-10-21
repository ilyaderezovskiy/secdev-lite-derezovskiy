# DV - Мини-проект «DevOps-конвейер»

> Этот файл - **индивидуальный**. Его проверяют по **rubric_DV.md** (5 критериев × {0/1/2} → 0-10).
> Подсказки помечены `TODO:` - удалите после заполнения.
> Все доказательства/скрины кладите в **EVIDENCE/** и ссылайтесь на конкретные файлы/якоря.

---

## 0) Мета

- **Проект:** учебный шаблон (https://github.com/Bquaith/secdev-2025)
- **Версия (commit/date):** 1.0.0 / 2025-10-17
- **Кратко (1-2 предложения):** Это веб-приложение на FastAPI с тестами на безопасность, где собираем и отображаем данные о пользователях и объектах, проверяем защиту от XSS и SQL-инъекций, и пакуем проект с отчётами тестов, логами и патчами.

---

## 1) Воспроизводимость локальной сборки и тестов (DV1)

- **Одна команда для сборки/тестов:**

```bash
  python3 -m venv venv && source venv/bin/activate && pip install -r requirements.txt && python -m pytest tests/ -v --junitxml=EVIDENCE/S06/test-report.xml
```

- **Версии инструментов (фиксация):**

  ```bash
  python --3.9.6
  fastapi==0.115.0
  jinja2==3.1.4
  pydantic==2.9.2
  pytest==8.3.2
  httpx==0.27.2
  ```

- **Описание шагов (кратко):** 
1. Установите Python 3.9+ и выполните one-liner команду из паспорта:

```bash
   python3 -m venv venv && source venv/bin/activate && pip install -r requirements.txt && python -m pytest tests/ -v --junitxml=EVIDENCE/S06/test-report.xml
```

2. One-liner автоматически создаст виртуальное окружение, установит зависимости и запустит тесты безопасности
3. Проверьте отчет тестов в EVIDENCE/S06/test-report.xml - все тесты должны быть зелеными

---

## 2) Контейнеризация (DV2)

- **Dockerfile:** [Dockerfile]([https://github.com/2gury/secdev-seed-s06-s08/blob/main/Dockerfile](https://github.com/Bquaith/secdev-2025/blob/main/Dockerfile)) ./Dockerfile — базовый образ python:3.11-slim, non-root appuser, переменная DB_PATH=/home/appuser/data/app.db, uvicorn app.main:app, минимальный образ (slim)
- 
- **Сборка/запуск локально:**

  ```bash
  docker build -t app:local .
  docker run --rm -p 8080:8080 app:local
  ```

- **Docker-compose:** [Docker-compose](https://github.com/Bquaith/secdev-2025/blob/main/docker-compose.yml). Healthcheck: exec‑form, проверяет HTTP-доступность порта 8000 внутри контейнера каждые 10 секунд. ./docker-compose.yml — сервис app, порты 8080:8000 - автоматически перезапускается, имя контейнера: seed, переменные окружения из .env.

**Сервисы в docker-compose.yml:**

web - основное приложение:

- Собирается из текущей директории (Dockerfile), образ secdev-seed:latest

- Пробрасывает порт 8000, автоматически перезапускается

- Устанавливает переменные APP_NAME и DEBUG

- Healthcheck - проверяет HTTP-доступность каждые 10 секунд

- Безопасность - отключены все права, запрещены новые привилегии

- Загружает переменные из .env файла

- Имя контейнера: seed
  
---

## 3) CI: базовый pipeline и стабильный прогон (DV3)

- **Платформа CI:** GitHub Actions
- **Файл конфига CI:** https://github.com/Bquaith/secdev-2025/blob/main/.github/workflows/ci.yml
- **Стадии (минимум):** checkout → deps → **build** → **test** (артефакты выгружаются в отдельных экшенах)
- **Фрагмент конфигурации (ключевые шаги):**

  ```yaml
  # TODO: укоротите под себя
  jobs:
  build_test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - name: Cache deps
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: pip-${{ hashFiles('**/requirements*.txt') }}
      - run: pip install -r requirements.txt
      - run: pytest -q

  ```

- **Стабильность:** последние 3 запуска зелёные; кэш pip сработал.
- **Ссылка/копия лога прогона:**  
  - [ci_run.png](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S08/ci_run.png)
  - [ci_tests_run.png](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S08/ci_tests_run.png)
  - [GitHub Actions (Project repo)](https://github.com/Bquaith/secdev-2025/actions)

---

## 4) Артефакты и логи конвейера (DV4)


| Артефакт/лог                    | Путь в `EVIDENCE/`            | Комментарий                                  |
|---------------------------------|-------------------------------|----------------------------------------------|
| Лог успешной сборки/тестов (CI) | [S08/ci_tests_run.png](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S08/ci_tests_run.png)     | Все 18 тестов пройдены. Время выполнения: 20 секунд |
| Локальный лог сборки (опц.)     | [S06/screenshots/Running tests locally.png](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S06/screenshots/Running%20tests%20locally.png)  | Все 10 тестов пройдены - система демонстрирует хорошую защиту |
| Описание результата сборки      | [S07/build.log] | Образ: secdev-seed:latest. Размер: 229MB. Собран на: 2025-10-20. Базовый образ: python:3.11-slim. Пользователь: appuser (non-root). Порты: 8000. Healthcheck: [настроен](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S07/health.json) |
| Freeze/версии инструментов      | [S06/requirements.txt](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S06/requirements.txt) | Версии зафиксированы - воспроизводимость обеспечена |
| Отчёт тестов                    | [S06/test-report.xml](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S06/test-report.xml) | 0 ошибок, 0 провалов, 0 пропущенных тестов. Все тесты за 0.33 секунды |

---

## 5) Секреты и переменные окружения (DV5 - гигиена, без сканеров)

- **Шаблон окружения:** добавлен файл `/.env.example` со списком переменных (без значений), например:
  - `REG_USER=`
  - `REG_PASS=`
  - `API_TOKEN=`
- **Хранение и передача в CI:**  
  - секреты лежат в настройках репозитория/организации (**masked**),  
  - в pipeline они **не печатаются** в явном виде.
- **Пример использования секрета в job (адаптируйте):**

  ```yaml
  - name: Login to registry (masked)
    env:
      REG_USER: ${{ secrets.REG_USER }}
      REG_PASS: ${{ secrets.REG_PASS }}
    run: |
      echo "::add-mask::$REG_PASS"
      echo "$REG_PASS" | docker login -u "$REG_USER" --password-stdin registry.example.com
  ```

- **Быстрая проверка отсутствия секретов в коде (любой простой способ):**

  ```bash
  # пример: поиск популярных паттернов
  git grep -nE 'AKIA|SECRET|token=|password=' || true
  ```

  _Сохраните вывод в `EVIDENCE/grep-secrets.txt`._
- **Памятка по ротации:** TODO: кто/как меняет secrets при утечке/ревоке токена.

---

## 6) Индекс артефактов DV


| Тип     | Файл в `EVIDENCE/`            | Дата/время         | Коммит/версия | Runner/OS    |
|---------|--------------------------------|--------------------|---------------|--------------|
| CI-лог  | [S08/ci_tests_run.png](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S08/ci_tests_run.png) , [ci_run.png](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S08/ci_run.png) | `2025-10-21` | v01.00.00      | `gha-ubuntu` |
| Лок.лог | [S06/screenshots/Running tests locally.png](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S06/screenshots/Running%20tests%20locally.png) | `2025-10-17` | v01.00.00 | `local` |
| Package | `package-notes.txt`            | …                  | v01.00.00      | -            |
| Freeze  | [S06/requirements.txt](https://github.com/ilyaderezovskiy/secdev-lite-derezovskiy/blob/main/EVIDENCE/S06/requirements.txt)  | `2025-10-17` | v01.00.00 | - |
| Grep    | `grep-secrets.txt`             | …                  | v01.00.00      | -            |

---

## 7) Связь с TM и DS (hook)

- **TM:** этот конвейер обслуживает риски процесса сборки/поставки (например, культура работы с секретами, воспроизводимость).  
- **DS:** сканы/гейты/триаж будут оформлены в `DS.md` с артефактами в `EVIDENCE/`.

---

## 8) Самооценка по рубрике DV (0/1/2)

- **DV1. Воспроизводимость локальной сборки и тестов:** [ ] 0 [ ] 1 [ ] 2  
- **DV2. Контейнеризация (Docker/Compose):** [ ] 0 [ ] 1 [ ] 2  
- **DV3. CI: базовый pipeline и стабильный прогон:** [ ] 0 [ ] 1 [ ] 2  
- **DV4. Артефакты и логи конвейера:** [ ] 0 [ ] 1 [ ] 2  
- **DV5. Секреты и конфигурация окружения (гигиена):** [ ] 0 [ ] 1 [ ] 2  

**Итог DV (сумма):** __/10

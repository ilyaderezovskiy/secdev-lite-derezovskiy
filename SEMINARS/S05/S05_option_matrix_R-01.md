# S05 - Матрица вариантов (Option Matrix)

---

## 0) Контекст риска (из S04)

* **Risk-ID:** R-01  
* **Threat:** S (Spoofing)  
* **DFD element/edge:** Edge: Internet → API Gateway  
* **NFR link (ID):** need NFR (AuthN/Token Security)  
* **L×I:** L=4, I=5, Score=20 
* **Ограничения/предпосылки:**  
  - Стек: REST API, JWT Bearer tokens  
  - Нет централизованного Identity Provider (IDP)  
  - Клиенты — браузеры и внешние сервисы  
  - Требования: доступность 99.9%, без MFA на старте  

---

## 1) Критерии и шкалы (1-5)

**Польза (чем больше - тем лучше, ↑):**

* **Security impact (↑):** насколько альтернатива снижает риск (предотвращает/обнаруживает/сдерживает).
* **Blast radius reduction (↑):** насколько сужает возможный ущерб (только один тенант/аккаунт/фича вместо всего сервиса).

**Стоимость/сложность (чем меньше - тем лучше, ↓):**

* **Complexity (↓):** сложность внедрения (код/конфиг/политики).
* **Time-to-mitigate (↓):** сколько времени до эффекта (в днях/спринтах).
* **Dependencies (↓):** внешние/внутренние зависимости (команды, провайдеры, миграции).

> Оценка **1-5**: 1 - минимально / 5 - максимально. Для «стоимостных» критериев **меньше - лучше**.

**Итоговые метрики (подсказка):**

* **Benefit = Security impact + Blast radius reduction**
* **Cost = Complexity + Time-to-mitigate + Dependencies**
* **Net = Benefit − Cost**  *(чем больше, тем привлекательнее)*

---

## 2) Таблица сравнения вариантов

| Alternative | Summary (1-2 фразы) | Security impact (↑,1-5) | Blast radius reduction (↑,1-5) | Complexity (↓,1-5) | Time-to-mitigate (↓,1-5) | Dependencies (↓,1-5) | **Benefit** | **Cost** | **Net** | Notes |
| ----------- | ------------------- | ----------------------: | -----------------------------: | -----------------: | -----------------------: | -------------------: | ----------: | -------: | ------: | ----- |
| **A** | JWT TTL ≤ 15 min + Refresh flow + Key rotation каждые 24ч | 4 | 3 | 2 | 2 | 2 | **7** | **6** | **+1** | Простое внедрение на gateway/service; минимальные зависимости |
| **B** | Взаимная TLS аутентификация (mTLS) для клиент–сервер | 5 | 4 | 4 | 4 | 5 | **9** | **13** | **−4** | Высокий эффект, но сложно и дорого; требует PKI и клиентской поддержки |
| **C** | Детекция reuse токенов + revoke при подозрении (via Redis/jti) | 3 | 4 | 3 | 3 | 3 | **7** | **9** | **−2** | Улучшает видимость, но не предотвращает первую компрометацию |

> **Как считать:**
> `Benefit = Security impact + Blast radius reduction`
> `Cost = Complexity + Time-to-mitigate + Dependencies`
> `Net = Benefit − Cost`

---

## 3) Тай-брейкеры при равенстве Net

* **Compliance/Privacy:** JWT TTL/rotation не затрагивает персональные данные — безопасно.  
* **Maintainability:** лёгкий откат и централизованный контроль через конфиг gateway.  
* **Team fit:** команда уже умеет работать с JWT и Refresh flow.  
* **Rollback safety:** безопасный rollback (TTL можно увеличить без даунтайма).

---

## 4) Решение (для переноса в ADR)

* **Chosen alternative:** **A — JWT TTL ≤ 15 min + Refresh flow + Rotation**
* **Почему:** обеспечивает быстрое снижение окна атаки при минимальной сложности. Простая реализация через конфигурацию gateway и поддержку refresh endpoint. Остальные варианты требуют слишком много зависимостей и инфраструктуры.  
* **ADR candidate:** `JWT TTL + Refresh + Rotation`
* **Связки:** Risk-ID R-01, NFR-ID need NFR (AuthN/Token Security), DFD: Edge Internet→API Gateway  

**Следующие шаги:**
1. Настроить TTL access-токенов на 15 мин, refresh на 24 ч.  
2. Включить key rotation для JWT signing keys каждые 24 ч.  
3. Добавить refresh endpoint `/api/auth/refresh`.  
4. Обновить документацию API (lifetime, refresh flow).  
5. Настроить мониторинг reuse токенов по claim `jti`.

---

## 6) Самопроверка

* [+] Риск и контекст из S04 указаны (Risk-ID, DFD, NFR-ID, L×I).
* [+] Сравнили **минимум 2 альтернативы**.
* [+] Заполнены все критерии 1-5; посчитаны Benefit/Cost/Net.
* [+] Описаны тай-брейкеры (если Net равен).
* [+] Принято решение и зафиксирован **ADR candidate**.
* [+] Готовы перенести решение в **`S05_ADR_<slug>.md`** (шаблон ADR).

---

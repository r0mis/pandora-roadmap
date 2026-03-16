# Pandora — Sessions Log

## Сесія 15.03.2026 (Claude + Gemini + Роман)

### Проблеми з якими прийшли:
- Пандора видавала помилку API 500 (predictionsServices.buildChatflow - fetch failed)
- ChromaDB і Tennis API не запускались після перезавантаження
- Ollama працював в CPU режимі (15-30 сек на відповідь)
- Не було автозапуску сервісів

### Що зробили:
1. Діагностика — знайшли що ChromaDB (порт 8000) і Tennis API (порт 5001) не запущені
2. Виявили ChromaDB v1→v2 міграцію (endpoint змінився)
3. Виправили конфлікт порту 5001 (дублюючий процес)
4. Виправили OLLAMA_URL з 172.17.0.1 на localhost
5. Переключили монітор на iGPU (Radeon 780M) — RX 9070 XT повністю вільна для AI
6. Налаштували Ollama GPU режим через systemd (ROCR_VISIBLE_DEVICES=0)
7. Знайшли CHAT_ID=7243726821 в users.json
8. Написали start_pandora.sh з Telegram повідомленням (статус GPU, сервіси, моделі)
9. Додали автозапуск через crontab @reboot
10. Додали WiFi watchdog (cron кожні 3 хвилини)
11. Виявили що Tailscale вже працює (100.127.189.25) — WireGuard скасований
12. Написали ingest_tennis_csv.py — RAG pipeline для ATP матчів
13. Проіндексували 67336 матчів (2019-2024) через nomic-embed-text
14. Зробили backup на GitHub (r0mis/pandora-bot)
15. Виправили .gitignore (venv, logs, sqlite3 більше не в репо)

### Архітектурні рішення:
- Фаза 2 (купівля другої відеокарти) — скасована через iGPU рішення
- WireGuard VPN — скасований, Tailscale вже є
- Flashscore парсер — залишаємо як запасний, шукаємо легальну альтернативу

### Брейншторм (нові ідеї):
- Triple Brain Оркестратор: Пандора → Claude + Gemini паралельно → Consensus Aggregator
- Confidence scoring для кожної відповіді
- Smart routing за типом задачі (код→Claude, аналіз→Gemini, теніс→qwen)
- Fallback ланцюжок якщо API недоступне
- Cost tracker (Redis: токени і USD)
- Session memory між сесіями
- Audit log всіх рішень оркестратора
- Self-learning loop (прийнятий/відхилений → покращення ваг)
- Langfuse для трасування (критично для Betting Brain)
- daily_ingest.py для щоденного оновлення ATP бази
- Semantic Router через власні ChromaDB вектори
- User memory окрема колекція ChromaDB
- A/B тестування моделей

### Gemini інсайти цієї сесії:
- Observability (Langfuse/LangSmith) — критично для фінансової аналітики
- Continuous Ingestion Pipeline — база без оновлення деградує
- Semantic Router бібліотека (ми вирішили робити власний через ChromaDB)
- Епізодична пам'ять користувача (Zep/Mem0 або своя колекція)

### Статус на кінець сесії:
- ChromaDB: OK (v2, порт 8000)
- Tennis API: OK (порт 5001)
- Telegram Bot: OK
- GPU: RX 9070 XT вільна, iGPU тримає монітор
- ATP база: 67336 матчів (100%)
- Flowise: запущено але ChromaDB ще не підключена до chatflow
- GitHub: r0mis/pandora-bot + r0mis/pandora-roadmap

### Наступна сесія починати з:
1. Підключити ChromaDB до Flowise (колекція tennis_atp)
2. Написати daily_ingest.py
3. Інтегрувати Langfuse
4. Проектування Triple Brain Оркестратора

    ## Сесія 15.03.2026 — Частина 2 (Claude + Gemini + Роман)

### Що зробили:
1. Діагностика: Claude зміг прочитати raw.githubusercontent.com через web_fetch (прямий HTTP GET)
2. Gemini підтвердив: його інструмент `browsing` спирається на пошуковий індекс Google,
   тому raw файли без індексу повертають URL_FETCH_STATUS_NOT_IN_SEARCH_INDEX
3. Підтверджена архітектура Triple Brain — різні інструменти моделей компенсують слабкі місця одна одної
4. Прийнята схема синхронізації між моделями через GitHub (ROADMAP + SESSIONS_LOG)

### Схема синхронізації Claude ↔ Gemini:
- Claude читає файли сам через web_fetch (raw GitHub URL)
- Gemini отримує вміст від Романа (copy-paste тексту або скріншот)
- Наприкінці кожної сесії: Claude генерує апдейт → Роман робить git commit → обидві моделі синхронізовані

### Розподіл сильних сторін (підтверджено):
- Claude → прямі HTTP запити, читання raw файлів, логів, API endpoints
- Gemini → глибокий веб-пошук, складний рефакторинг, аналітика
- Qwen → швидкий локальний агент для рутинних задач

### Статус на кінець сесії:
- Схема синхронізації: прийнята ✅
- Flowise → ChromaDB: ще не підключено (чекаємо деталей від Романа) ⏳

### Наступна сесія починати з:
1. Підключити ChromaDB до Flowise (колекція tennis_atp) — Роман надасть деталі конфігурації
2. daily_ingest.py
3. Langfuse
4. Triple Brain Оркестратор

## Сесія 15.03.2026 — Частина 2 (Claude + Gemini + Роман)
- Claude читає raw GitHub через web_fetch (прямий HTTP GET)
- Gemini browsing спирається на Google індекс → raw файли не читає
- Прийнята схема синхронізації: Claude сам читає URL, Gemini отримує copy-paste
- Flowise → ChromaDB: ще не підключено ⏳
### Наступна сесія:
1. Flowise → ChromaDB (tennis_atp)
2. daily_ingest.py
3. Langfuse
4. Triple Brain Оркестратор

---
## Сесія 16.03.2026

### Апдейти архітектури:
1. Kitchen SaaS — NBC + ABC в ChromaDB, перевірка DXF на відповідність нормам
2. Бізнес-розвідка — Tavily/Serper + Web Scraper для Reddit/конкурентів
3. Agentic Swarm — Пандора як оркестратор у Flowise
4. Core Identity — системний промпт з бекграундом Романа

### Інцидент та фікс:
- Бот не надіслав стартове повідомлення — два екземпляри конфліктували
- Фікс: захист від дублів додано в start_pandora.sh

### Постійний фікс /tmp:
- pandora-roadmap перенесено з /tmp до /mnt/pandora_db/pandora-roadmap
- /tmp очищається при перезавантаженні — більше не використовуємо

### Наступні задачі:
1. Flowise → ChromaDB (tennis_atp)
2. daily_ingest.py
3. Tavily/Serper агенти
4. Core Identity промпт
5. night_training.sh

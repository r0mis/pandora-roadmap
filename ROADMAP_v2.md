> **[SYSTEM INSTRUCTIONS FOR AI]**
> Ти — технічний партнер, CTO та архітектор у проєкті Pandora AI System.
> **Творець:** Роман, 42 роки, Канада. Інженер, екс-прикордонник, інсталятор кухонь.
> **Ціль:** 6,000 CAD/міс, вихід з "корпоративного рабства", автоматизація рутини.
> **Режим роботи:** Максимальна економія часу. Тільки готові команди для термінала (без ручного редагування). Усі токени передаються тільки через `.env`.
> 
> **ДВІ ПІВКУЛІ (автоматично перемикайся залежно від контексту запиту):**
> 1. **[TECH CORE]**: Залізо (Ryzen 7800X3D, RX 9070 XT), Мережа (Tailscale 100.127.189.25, Docker 172.17.0.1), Сервіси (ChromaDB v2 :8000, Ollama :11434, Flowise :3001, Tennis API :5001). Завжди генеруй код на Python асинхронно, без галюцинацій.
> 2. **[MIND & STRATEGY]**: Стратегія, монетизація (Kitchen SaaS, Трейдинг), екосистема (Penta-Brain Orchestrator, Pandora Swarm на FreeBSD, Health-Bot Garmin, romis_core_memory).
>
> *Після прочитання цього файлу: підтвердь синхронізацію, увійди в роль та чекай на вказівки.*

# Pandora AI System — Roadmap v2.0
**Дата:** 15.03.2026 | **Сесія:** Triple Brainstorm (Роман + Claude + Gemini)

## Система та залізо
- Сервер: AMD Ryzen 7 7800X3D, Ubuntu 24.04 LTS
- GPU AI: AMD RX 9070 XT 16GB (Device 0) — вільна для AI
- GPU Монітор: AMD Radeon 780M iGPU (Device 1)
- ROCm: 7.1 | Ollama: localhost:11434
- Tailscale VPN: 100.127.189.25
- Репо: github.com/r0mis/pandora-bot

## Що закрито (Фаза 1)
1. iGPU для монітора — RX 9070 XT вільна. Фаза 2 скасована
2. Ollama GPU режим — systemd: ROCR_VISIBLE_DEVICES=0, HIP_VISIBLE_DEVICES=0, HSA_OVERRIDE_GFX_VERSION=12.0.0
3. ChromaDB v2 — /api/v2/heartbeat, порт 8000, /mnt/pandora_db/memory/
4. ATP база — 67336 матчів (2019-2024), колекція tennis_atp
5. Tennis API — Flask, порт 5001, /analyze
6. start_pandora.sh — автозапуск + Telegram статус GPU
7. Автозапуск — crontab @reboot sleep 20
8. WiFi watchdog — cron кожні 3 хвилини
9. Tailscale VPN — WireGuard скасований
10. CHAT_ID=7243726821, users.json admin
11. amdgpu — /etc/modules-load.d/amdgpu.conf
12. GitHub backup — r0mis/pandora-bot

## Структура проекту
/mnt/pandora_db/
  bot/
    bot.py, tennis_api.py, tennis_analyzer.py
    ingest_tennis_csv.py, collector.py
    flashscore_parser.py, start_pandora.sh
    .env: TELEGRAM_TOKEN, CHAT_ID=7243726821
          GOOGLE_API_KEY, OLLAMA_URL=localhost:11434
          FLOWISE_URL=localhost:3001
          FLOWISE_CHATFLOW_ID=42de74c8-80ae-40da-b203-b05dcad627af
    knowledge/tennis_archive/ — CSV 1968-2024
  memory/chroma.sqlite3 — 67336 матчів
  logs/
  projects/04_System_Master/users.json

## Моделі Ollama
- qwen2.5:14b — основна модель
- nomic-embed-text — векторизація
- llava:latest — аналіз зображень
- qwen2.5:7b, llama3:latest — резервні

## Поточні проблеми
1. ChromaDB не підключена до Flowise (колекція tennis_atp)
2. Ollama CPU для ембеддингів (потрібен OLLAMA_NUM_PARALLEL=2)
3. ATP 2025 — порожній файл
4. Flashscore парсер — хрупкий через Cloudflare

## Пріоритети (Фаза 2)
### Терміново:
1. Flowise → ChromaDB (tennis_atp)
2. daily_ingest.py — оновлення бази щодня
3. Langfuse — трасування для Betting Brain

### Тиждень:
4. Triple Brain Оркестратор
5. Semantic Router через ChromaDB вектори
6. User memory колекція

### Пізніше:
7. Bootstrap install.sh
8. Whisper medium
9. DeepSeek R1 14B
10. TTS голос укр/рус/англ

## Triple Brain Оркестратор
Запит → Semantic Router → Claude + qwen + Gemini паралельно
→ Consensus Aggregator (confidence score + вагові коефіцієнти)
→ Верифікований результат + audit log + cost tracker
→ Self-learning loop (прийнятий/відхилений → покращення ваг)

Ваги по типу задачі:
- Код/архітектура → Claude вага вища
- Аналіз даних/веб → Gemini вага вища
- Теніс/локальне → qwen вага вища
- Fallback → локальна модель якщо API впало

API: GOOGLE_API_KEY вже є. Потрібен ANTHROPIC_API_KEY.

Додаткові компоненти:
- Cost tracker (Redis: токени і USD)
- Rate limiting
- Session memory (Redis)
- Audit log
- A/B тестування моделей

## Betting Brain
Фундамент: tennis_analyzer.py + ChromaDB + collector.py
Цикл: прогноз → ChromaDB → реальний результат → 👍/👎 → покращення промптів
Потрібно: Langfuse трасування (критично для фінансової аналітики)

## Правила роботи
- Скрипти тільки для копіпасту в термінал
- venv: /mnt/pandora_db/bot/venv
- Python файли через: sudo python3 << 'EOF'
- Логи: /mnt/pandora_db/logs/

## Синхронізація моделей
- Claude читає ROADMAP + SESSIONS_LOG самостійно через web_fetch (raw GitHub URL)
- Gemini отримує вміст через copy-paste від Романа на початку сесії
- Наприкінці кожної сесії: Claude генерує апдейт обох файлів → git commit → push
- Команда для commit:
  cd /mnt/pandora_db/bot
  git add ROADMAP_v2.md SESSIONS_LOG.md
  git commit -m "Session YYYY-MM-DD update"
  git push

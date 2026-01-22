---
sidebar_position: 2
---

# Автоматические посты в Telegram-канал с помощью нейросети

TeleDrive фокусируется на файловом хранилище, но вы можете автоматизировать публикации в Telegram-канал, связав бота Telegram и AI‑провайдера, который генерирует текст и картинки. Ниже — пошаговая инструкция «как пользоваться».

## Быстрый старт (пошагово)

### Что нужно

- Аккаунт Telegram и **канал**, которым вы управляете.
- **Токен Telegram‑бота** от @BotFather.
- **API‑ключ** AI‑провайдера (например, OpenAI).
- Node.js 18+ на вашем компьютере/сервере.

### Шаг 1. Создайте бота и добавьте его в канал

1. Откройте Telegram и напишите **@BotFather**.
2. Отправьте `/newbot` и следуйте инструкциям.
3. Скопируйте токен бота (выглядит как `123456:ABC-DEF...`).
4. В настройках канала откройте **Администраторы** → **Добавить**.
5. Добавьте бота и дайте ему право публиковать сообщения.
6. Запомните идентификатор канала:
   - Публичный канал: `@your_channel`
   - Приватный канал: числовой ID (см. подсказку ниже).

> Подсказка: чтобы узнать ID приватного канала, перешлите любое сообщение из канала боту @userinfobot или @getidsbot.

### Шаг 2. Подготовьте папку и установите зависимости

```bash
mkdir telegram-ai-poster
cd telegram-ai-poster
npm init -y
npm install node-fetch
```

Создайте файл **scripts/auto-post.js** и вставьте этот пример:

```js
const OPENAI_API_KEY = process.env.OPENAI_API_KEY
const TELEGRAM_BOT_TOKEN = process.env.TELEGRAM_BOT_TOKEN
const TELEGRAM_CHANNEL = process.env.TELEGRAM_CHANNEL

async function generatePost() {
  const textResponse = await fetch('https://api.openai.com/v1/responses', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${OPENAI_API_KEY}`,
    },
    body: JSON.stringify({
      model: 'gpt-4.1-mini',
      input: 'Напиши пост на 2–3 предложения про продуктивность.',
    }),
  })

  const textData = await textResponse.json()
  const text = textData.output?.[0]?.content?.[0]?.text ?? 'Привет от нейросети!'

  const imageResponse = await fetch('https://api.openai.com/v1/images', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${OPENAI_API_KEY}`,
    },
    body: JSON.stringify({
      model: 'gpt-image-1',
      prompt: 'Минималистичная иллюстрация блокнота и кофе, пастельные цвета',
      size: '1024x1024',
    }),
  })

  const imageData = await imageResponse.json()
  const imageUrl = imageData.data?.[0]?.url

  return { text, imageUrl }
}

async function sendToTelegram({ text, imageUrl }) {
  const payload = {
    chat_id: TELEGRAM_CHANNEL,
    caption: text,
    photo: imageUrl,
    parse_mode: 'HTML',
  }

  await fetch(`https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendPhoto`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload),
  })
}

async function main() {
  const post = await generatePost()
  await sendToTelegram(post)
}

main()
```

### Шаг 3. Создайте `.env` с ключами

Создайте файл **.env** в этой же папке:

```bash
OPENAI_API_KEY=your_openai_key
TELEGRAM_BOT_TOKEN=your_bot_token
TELEGRAM_CHANNEL=@your_channel
```

Запуск:

```bash
export $(grep -v '^#' .env | xargs)
node scripts/auto-post.js
```

Если всё правильно, в канале появится новый пост.

### Шаг 4. Автопубликация по расписанию

Варианты:

- **Cron** на сервере или VPS.
- **GitHub Actions** по расписанию.
- **Cloud schedulers** (Google Cloud Scheduler, AWS EventBridge).

Пример cron (ежедневно в 9:00):

```bash
0 9 * * * cd /path/to/telegram-ai-poster && export $(grep -v '^#' .env | xargs) && node scripts/auto-post.js
```

## Важно про безопасность

- Храните ключи только в переменных окружения или секрет‑хранилищах.
- Никогда не коммитьте `.env` в Git.
- Давайте боту только нужные права.

# Система уведомлений 🔔

Гибкая система отправки уведомлений с множественными каналами, шаблонами и логикой условий.

## 📝 Описание

Система обрабатывает события (платежи, ошибки, уведомления), оценивает правила срабатывания, выбирает оптимальный канал отправки, форматирует сообщение по шаблону и отправляет с retry логикой.

## 🎯 Возможности

- ✅ **Множественные каналы**: Email, SMS, Telegram, Slack, Push
- ✅ **Правила срабативания**: If-then логика для разных событий
- ✅ **Приоритет**: Urgent (SMS + Telegram), Normal (Email), Low (Push)
- ✅ **Шаблоны**: С переменными для персонализации
- ✅ **Батчинг**: Группировка нескольких уведомлений
- ✅ **Rate limiting**: Не спамим пользователей
- ✅ **Retry логика**: Экспоненциальный backoff при ошибках
- ✅ **Tracking**: Статус доставки каждого уведомления
- ✅ **Unsubscribe**: Пользователь может отписаться
- ✅ **Do Not Disturb**: Уважение к часовым поясам и настройкам

## 🏗️ Архитектура

```
Event triggered
    ↓
Apply Rules (if-then)
    ↓
Select Channel(s)
    ↓
Get Template
    ↓
Substitute Variables
    ↓
Check Rate Limit
    ↓
Queue Notification
    ↓
Send via Channel
    ↓
Track Delivery
    ↓
Log to DB
```

## 🔧 Требуемые интеграции

- **Email**: SMTP (Gmail, SendGrid, AWS SES)
- **SMS**: Twilio, Amazon SNS
- **Telegram**: Bot API
- **Slack**: Bot/App API
- **Push**: Firebase Cloud Messaging
- **Database**: n8n DB или PostgreSQL (для шаблонов и правил)
- **Redis**: Для rate limiting и queue (опционально)

## 📋 Примеры событий и правил

### Правило 1: Payment Success
```yaml
event: payment.completed
condition: amount > 100
action:
  channels: [email, telegram]
  template: payment_success_large
  priority: normal
  delay: 0  # сразу
```

**Шаблон:**
```
Спасибо за покупку на сумму ${amount} ${currency}!

Номер заказа: ${order_id}
Статус: ${status}
Дата: ${date}

Отследить заказ: ${tracking_link}
```

**Результат:**
```
Email:
  From: noreply@shop.com
  To: user@example.com
  Subject: Подтверждение платежа #12345
  Body: [шаблон с подставленными переменными]

Telegram:
  Спасибо за покупку на сумму $99.99 USD!
  Номер заказа: 12345
  ...
```

### Правило 2: System Error Alert
```yaml
event: system.error
condition: severity >= 'critical' AND error_count > 5
action:
  channels: [sms, telegram, slack]  # все каналы сразу!
  template: alert_critical
  priority: urgent
  delay: 0
  retry: 3  # 3 попытки
```

### Правило 3: Weekly Digest
```yaml
event: user.weekly_digest
condition: true
action:
  channels: [email]
  template: weekly_digest
  priority: low
  delay: 3600  # отправить через час
  batch: true  # группировать
```

## 📊 Примеры шаблонов

### Шаблон 1: Email подтверждение
```html
<h1>Добро пожаловать, {{first_name}}!</h1>

<p>Спасибо что зарегистрировался в нашем сервисе.</p>

<p>
  <a href="{{verification_link}}" style="background: blue; color: white; padding: 10px 20px;">
    Подтвердить email
  </a>
</p>

<p>Если кнопка не работает, скопируй эту ссылку:<br>
{{verification_link}}</p>

<p>С уважением,<br>
Команда {{company_name}}</p>
```

### Шаблон 2: SMS alert
```
⚠️ {{alert_type}}

{{message}}

ID: {{incident_id}}
Время: {{timestamp}}
```

### Шаблон 3: Telegram rich message
```
✅ Заказ {{order_id}} отправлен!

📦 Трекинг номер: {{tracking_number}}
🚚 Компания: {{shipping_company}}
📍 Адрес: {{delivery_address}}
⏰ Ожидаемая доставка: {{expected_date}}

👉 {{tracking_url}}
```

## 🚀 План реализации

- [ ] Система правил (if-then logic)
- [ ] Шаблонизация сообщений
- [ ] Email интеграция (SMTP)
- [ ] SMS интеграция (Twilio)
- [ ] Telegram интеграция
- [ ] Slack интеграция
- [ ] Firebase Push интеграция
- [ ] Queue обработка
- [ ] Rate limiting
- [ ] Retry логика
- [ ] Delivery tracking
- [ ] Unsubscribe механизм
- [ ] Do Not Disturb расписание

## 💡 Примеры сложных сценариев

### Сценарий 1: Каскадные уведомления
```
Event: High CPU usage detected

Time 1 (alert): Send Email
If no ack after 5 min → SMS
If no ack after 10 min → Telegram urgent
If no ack after 15 min → Call on-call engineer
```

### Сценарий 2: Умная маршрутизация по времени
```
Event: Daily digest

IF user.timezone.hour in 08:00-10:00:
  Send immediately via Email (утро, проверяет почту)
ELSE:
  Queue for next morning 08:00
```

### Сценарий 3: Батчинг уведомлений
```
Events: 
  - Item shipped (16:45)
  - Payment received (16:50)
  - Review requested (16:55)

Batched notification at 17:00:
  "Hi! 3 updates: ..."
```

### Сценарий 4: A/B тестирование текста
```
Rule: Promotional email

Subject variants:
  A: "Special offer just for you! 🎉"
  B: "Limited time: 30% off today"
  C: "Your exclusive deal"

Track: Opens, clicks, conversions
Winner: используем в production
```

## 📊 Метрики для отслеживания

| Метрика | Цель |
|---------|------|
| Delivery Rate | > 98% |
| Open Rate (Email) | > 25% |
| Click Rate | > 5% |
| Bounce Rate | < 2% |
| Unsubscribe Rate | < 0.5% |
| Avg Time to Delivery | < 30 sec |

## 🔐 Безопасность

### Чувствительные данные
```
✅ DO:
- Использовать переменные (${user_id}, не реальные ID)
- Хранить templates в DB с версионированием
- Логировать кто отправляет и когда
- Шифровать email адреса в логах

❌ DON'T:
- Не встраивать реальные данные в шаблоны
- Не отправлять API ключи в уведомлениях
- Не хранить пароли в шаблонах
```

### Rate Limiting
```
- 5 уведомлений в день для одного пользователя
- 1 SMS в час (дорогущие!)
- Email батчится (максимум 10 в день)
- Telegram не лимитим (бесплатен)
```

## 📚 Примеры реальных кейсов

### Компания E-commerce
```
1. Order placed → "Thanks! Processing..."
2. Payment confirmed → Email + SMS
3. Shipped → Email + Telegram + Push
4. Delivered → Push (самое важное)
5. 1 неделя → Email "How was your experience?"
```

### Компания SaaS
```
1. Signup → Email verification
2. Trial ending → Email (3 дня до конца)
3. Payment required → Email + SMS
4. Important feature added → Email + Slack (для team members)
5. System error → SMS + Slack (для engineers)
```

### Компания Logistics
```
1. Pickup scheduled → SMS + Email
2. Picked up → Push
3. In transit → Real-time GPS tracking push
4. Delivery attempt → SMS
5. Delivered → Email + SMS
6. Problem → SMS + call
```

---

**Статус:** 📅 Планируется
**Приоритет:** Высокий (используется везде)
**Сложность:** Средняя (много интеграций но стандартные)

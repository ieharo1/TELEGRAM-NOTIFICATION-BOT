# 📱 Telegram Notification Bot

Bot multifuncional para enviar notificaciones, alertas y mensajes automatizados a través de Telegram. Desarrollado por **Isaac Esteban Haro Torres**.

---

## 📝 Description

Bot profesional para automatización de notificaciones via Telegram. Ideal para sistemas de monitoreo, alertas y comunicación automatizada.

### What does this project do?

- **Envío de notificaciones**: Mensajes a usuarios o grupos
- **Alertas automáticas**: Notificaciones basadas en eventos
- **Programación**: Envía mensajes en horarios específicos
- **Botones interactivos**: Inline keyboards para acciones
- **Webhook integration**: Recibe eventos de otros sistemas
- **Logs persistentes**: Historial de notificaciones enviadas

---

## ✨ Características Principales

| Característica | Descripción |
|----------------|-------------|
| 📢 **Notificaciones** | Envía a usuarios, grupos, canales |
| ⏰ **Programación** | Envía en horarios específicos |
| 🔗 **Webhooks** | Recibe eventos de APIs externas |
| 🎛️ **Botones** | Inline keyboards interactivos |
| 📊 **Estadísticas** | Métricas de notificaciones |
| 🔄 **Auto-retry** | Reintenta envíos fallidos |

---

## 🛠️ Stack Tecnológico

- **Lenguaje**: Python 3.10+
- **Bot API**: python-telegram-bot (v20+)
- **Webhooks**: Flask, aiohttp
- **Programación**: schedule, apscheduler
- **Base de datos**: SQLite (logs)
- **Async**: asyncio

---

## 🚀 Instalación y Uso

### Instalación

```bash
pip install python-telegram-bot flask apscheduler sqlalchemy
```

### Configuración

```python
# config.py
from telegram import Update
from telegram.ext import ApplicationBuilder

TOKEN = 'TU_BOT_TOKEN'  # Obtener de @BotFather

app = ApplicationBuilder().token(TOKEN).build()
```

### Envío de notificaciones básico

```python
from telegram import Bot

bot = Bot(token='TU_TOKEN')

# Enviar mensaje simple
async def enviar_alerta():
    await bot.send_message(
        chat_id='ID_DEL_USUARIO',
        text='⚠️ Alerta: Tu sistema requiere atención'
    )
```

### Bot con comandos interactivos

```python
from telegram.ext import Application, CommandHandler, MessageHandler, filters

app = ApplicationBuilder().token(TOKEN).build()

# Comando /start
async def start(update, context):
    await update.message.reply_text(
        '¡Hola! Soy tu bot de notificaciones.\n'
        'Usa /ayuda para ver comandos disponibles.'
    )

# Comando /estado
async def estado(update, context):
    await update.message.reply_text(
        '✅ Sistema operativo\n'
        '📊 Estado: Normal'
    )

app.add_handler(CommandHandler("start", start))
app.add_handler(CommandHandler("estado", estado))

app.run_webhook(
    listen="0.0.0.0",
    port=8443,
    url_path="webhook"
)
```

### Botones interactivos

```python
from telegram import InlineKeyboardButton, InlineKeyboardMarkup

keyboard = [
    [
        InlineKeyboardButton("✅ Aprobar", callback_data="aprobar"),
        InlineKeyboardButton("❌ Rechazar", callback_data="rechazar")
    ]
]

reply_markup = InlineKeyboardMarkup(keyboard)

await update.message.reply_text(
    "¿Apruebas esta acción?",
    reply_markup=reply_markup
)
```

---

## 📁 Estructura del Proyecto

```
telegram-notification-bot-python/
├── Bot_Telegram_ISERQ.ipynb
├── bot/
│   ├── __init__.py
│   ├── handlers.py           # Manejadores de comandos
│   ├── notifications.py     # Funciones de notificación
│   ├── scheduler.py         # Programación de mensajes
│   └── database.py          # SQLite para logs
├── config.py
├── requirements.txt
└── README.md
```

---

## 💡 Casos de Uso

1. **Monitoreo de sistemas**: Alertas de servidores, aplicaciones
2. **Notificaciones de negocio**: Pedidos, pagos, inventario
3. **Recordatorios**: Tareas, citas, eventos
4. **Reportes**: Envío automático de reportes diarios
5. **Soporte**: Tickets de helpdesk

---

## 📊 Dashboard de Notificaciones

```
╔═══════════════════════════════════════════════════╗
║      📱 BOT NOTIFICACIONES - PANEL               ║
╠═══════════════════════════════════════════════════╣
║                                                   ║
║  📤 Enviadas hoy:        1,247                   ║
║  ✅ Exitosas:            1,230 (98.6%)           ║
║  ❌ Fallidas:              17 (1.4%)             ║
║                                                   ║
║  📅 Esta semana:         8,432                   ║
║  📆 Este mes:          32,891                    ║
║                                                   ║
║  👥 Usuarios activos:      156                   ║
║  💬 Chats registrados:     42                    ║
╚═══════════════════════════════════════════════════╝
```

---

## 🔗 Integración con Webhooks

```python
# Receives notifications from external systems
from flask import Flask, request
from telegram import Update

app = Flask(__name__)

@app.post("/webhook")
def webhook():
    update = Update.de_json(request.get_json(force=True), bot)
    
    # Procesar según tipo de evento
    if update.message:
        handle_message(update)
    
    return {"ok": True}

# Endpoint para alertas de sistema
@app.post("/alert/<chat_id>")
def send_alert(chat_id):
    mensaje = request.json.get('message', 'Alerta!')
    nivel = request.json.get('level', 'info')
    
    # Enviar con formato según nivel
    emoji = {'info': 'ℹ️', 'warning': '⚠️', 'error': '❌'}
    await bot.send_message(chat_id, f"{emoji.get(nivel)} {mensaje}")
    
    return {"sent": True}
```

---

## ⏰ Programación de Mensajes

```python
from apscheduler.schedulers.asyncio import AsyncIOScheduler

scheduler = AsyncIOScheduler()

# Enviar reporte diario
scheduler.add_job(
    enviar_reporte_diario,
    'cron',
    hour=8,
    minute=0,
    args=[chat_id]
)

# Recordatorio cada hora
scheduler.add_job(
    check_pending_tasks,
    'interval',
    hours=1
)

scheduler.start()
```

---

## 🛡️ Manejo de Errores

```python
# Retry automático con backoff
async def send_with_retry(chat_id, message, max_retries=3):
    for attempt in range(max_retries):
        try:
            await bot.send_message(chat_id, message)
            return True
        except Exception as e:
            wait_time = 2 ** attempt
            await asyncio.sleep(wait_time)
    log_error(f"Failed after {max_retries} attempts")
    return False
```

---

## ⚠️ Notas Importantes

- Obtén el token de @BotFather en Telegram
- Necesitas HTTPS para webhooks (usa ngrok para desarrollo)
- Respeta los límites de mensajes de Telegram
- Implementa rate limiting para evitar bloqueos


---

## 👨‍💻 Desarrollado por Isaac Esteban Haro Torres

**Ingeniero en Sistemas · Full Stack · Automatización · Data**

- 📧 Email: zackharo1@gmail.com
- 📱 WhatsApp: 098805517
- 💻 GitHub: https://github.com/ieharo1
- 🌐 Portafolio: https://ieharo1.github.io/portafolio-isaac.haro/

---

© 2026 Isaac Esteban Haro Torres - Todos los derechos reservados.

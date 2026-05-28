# Aura Bot — Agente de IA para Florería en WhatsApp

> Flujo de n8n que convierte tu Chatwoot + WhatsApp en un agente de ventas inteligente, capaz de cotizar, registrar pedidos y derivar al humano cuando es necesario.

---

## ✨ ¿Qué hace este bot?

Aura es un agente conversacional para florerías que atiende a tus clientes por WhatsApp de forma automática. Clasifica la intención de cada mensaje y responde de manera distinta según el contexto:

| Intención detectada | Qué hace el bot |
|---------------------|-----------------|
| 💬 **INFO** | Consulta el catálogo (flores, precios, stock) y el documento de la tienda (horarios, ubicación, políticas) |
| 🛒 **WANT_BUY** | Extrae los datos del pedido, lo registra en la base de datos y confirma la compra |
| 🚚 **Logística** | Detecta si el cliente prefiere delivery o retiro, y actualiza el pedido |
| 🙋 **HUMAN_INTERVENTION** | Transfiere la conversación a un agente humano y apaga el bot |
| ❓ **DONT_UNDERSTAND** | Responde amablemente explicando que está fuera de su alcance |

---

## 🧠 Arquitectura del flujo

```
WhatsApp → Chatwoot Webhook
              │
              ▼
        [Filtro de eventos]
              │
       ┌──────┴──────┐
       │             │
   Mensaje       Mensaje
   entrante      saliente (agente)
       │
       ▼
  [Cola Redis] ← acumula mensajes simultáneos
       │
       ▼
  [AI Brain - GPT-4o] ← clasifica intención
       │
  ┌────┼────┬────────┐
  │    │    │        │
INFO WANT HUMAN  DONT
  │    │    │    UNDERSTAND
  ▼    ▼    ▼        ▼
 [Agente  [Extrae  [Deriva  [Responde
  Info]    pedido]  humano]  breve]
  │    │
  ▼    ▼
[Google  [Google
 Docs]   Sheets]
```

---

## 🛠️ Stack tecnológico

- **[n8n](https://n8n.io)** — Orquestador del flujo
- **[Chatwoot](https://www.chatwoot.com)** — CRM / bandeja de WhatsApp
- **OpenAI GPT-4o** — Motor de IA (clasificación + redacción)
- **PostgreSQL** — Base de datos de clientes, pedidos e historial
- **Redis** — Cola de mensajes para agrupar inputs simultáneos
- **Google Sheets** — Catálogo de flores y ramos
- **Google Docs** — Documento con info general de la tienda

---

## 📋 Requisitos previos

Antes de importar el flujo necesitas tener configurado:

- [ ] Instancia de **n8n** (self-hosted o cloud)
- [ ] Cuenta de **Chatwoot** con inbox de WhatsApp conectado
- [ ] API Key de **OpenAI**
- [ ] Base de datos **PostgreSQL** con las tablas del esquema (ver abajo)
- [ ] Instancia de **Redis**
- [ ] **Google Sheets** con el inventario de productos
- [ ] **Google Docs** con la info general de tu tienda
- [ ] Service account de **Google Cloud** con acceso a Sheets y Docs

---

## 🗄️ Esquema de base de datos (PostgreSQL)

Ejecuta este SQL en tu base de datos antes de activar el flujo:

```sql
CREATE TABLE clients (
  id SERIAL PRIMARY KEY,
  phone_number VARCHAR(20) UNIQUE NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  client_id INTEGER REFERENCES clients(id),
  product_details TEXT,
  total_amount NUMERIC(10,2),
  delivery_method VARCHAR(20),
  status VARCHAR(50) DEFAULT 'validando_pedido',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE chat_history (
  id SERIAL PRIMARY KEY,
  client_id INTEGER REFERENCES clients(id),
  sender VARCHAR(20),
  content TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 🚀 Cómo importar el flujo

1. Descarga el archivo `aura-bot.json`
2. En tu instancia de n8n: **Workflows → Import from file**
3. Selecciona el archivo descargado
4. Configura las credenciales (ver sección siguiente)
5. Activa el workflow

---

## 🔑 Credenciales a configurar

Al importar, n8n te pedirá vincular estas credenciales:

| Credencial | Tipo | Qué poner |
|------------|------|-----------|
| `Postgres account` | PostgreSQL | Host, puerto, DB, usuario y contraseña |
| `OpenAi account` | OpenAI API | Tu API Key de OpenAI |
| `Chatwood Aura Agent` | HTTP Header Auth | Header: `api_access_token` — Valor: tu token de Chatwoot |
| `Google Sheets account` | Google Service Account | JSON de tu service account de Google Cloud |
| `Redis account` | Redis | Host, puerto y contraseña de tu Redis |

---

## ⚙️ Variables a reemplazar en el flujo

Busca y reemplaza estos placeholders en los nodos correspondientes:

| Placeholder | Dónde reemplazar | Valor |
|-------------|-----------------|-------|
| `YOUR_CHATWOOT_DOMAIN` | Todos los nodos HTTP Request | Tu dominio de Chatwoot (ej: `mi-crm.com`) |
| `YOUR_GOOGLE_DOC_ID` | Nodo `Get Aura Info` | ID de tu Google Doc con info de la tienda |
| `YOUR_GOOGLE_SHEET_ID` | Nodo `Get Aura Data` | ID de tu Google Sheet con el inventario |

---

## 📄 Estructura del Google Sheets (Inventario)

El sheet debe tener al menos estas hojas:

- **`Flores`** — columnas: `Nombre`, `Precio`, `Stock`, `Descripción`
- **`Ramos`** — columnas: `Nombre`, `Precio`, `Contenido`, `Descripción`

---

## 💡 Funcionalidades destacadas

- **Cola inteligente con Redis**: agrupa mensajes enviados en ráfaga para responder una sola vez
- **Memoria de conversación**: guarda historial en PostgreSQL para contexto entre sesiones
- **Gestión de pedidos completa**: crea, actualiza y hace seguimiento del estado del pedido
- **Bot on/off por conversación**: el bot se apaga automáticamente cuando interviene un humano
- **Notas privadas en Chatwoot**: el bot anota el razonamiento interno visible solo para el agente

---

## 🤝 Contribuciones

¿Mejoraste el flujo? ¡Los PRs son bienvenidos! Puedes:
- Agregar soporte para más intenciones
- Mejorar los prompts de los agentes
- Añadir integración con sistemas de pago
- Traducir los prompts a otros idiomas

---

<p align="center">Muchas gracias por avanzar y llegar hasta aquí 🤓</p>

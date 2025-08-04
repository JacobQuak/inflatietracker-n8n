# 🧠 Automatisch aanmaken van Postgres Chat Memory in n8n met Supabase

Als je een connectie hebt met een PostgreSQL-database (zoals Supabase) in n8n, dan hoef je de `chat_memory`-tabel meestal **niet handmatig** aan te maken.

---

## ✅ Voorwaarden

De `Postgres Chat Memory` node in n8n maakt automatisch de tabel aan als:

- Je een werkende PostgreSQL-verbinding hebt (bijv. Supabase)
- De opgegeven tabel nog niet bestaat
- Je de workflow uitvoert met correcte instellingen

---

## 📦 Wat wordt automatisch aangemaakt?

Bij het eerste gebruik van de node genereert n8n een tabel met de volgende structuur:

| Kolomnaam     | Type       | Opmerking                    |
|---------------|------------|------------------------------|
| `id`          | UUID       | Primaire sleutel             |
| `session_id`  | TEXT       | Groepeert de sessies         |
| `type`        | TEXT       | `human` of `ai`              |
| `content`     | TEXT       | Inhoud van het bericht       |
| `created_at`  | TIMESTAMP  | Automatisch tijdstip         |

> ℹ️ De structuur kan worden uitgebreid met `metadata` (JSONB) indien je dat zelf toevoegt.

---

## 🔧 Hoe stel je het in?

In de `Postgres Chat Memory` node:

- **Table name**: `chat_memory` (of een eigen naam)
- **Session ID**: bijv. `={{ $runId }}` of `={{ $json.userId }}`
- **inputField**: de AI-invoer (bijv. prompt)
- **outputField**: het AI-antwoord

De node gebruikt deze info om automatisch de opslag te beheren.

---

## 🔍 Controleer de tabel in Supabase

1. Ga naar je Supabase dashboard
2. Navigeer naar **Table Editor**
3. Zoek naar `chat_memory`
4. Bekijk de kolommen en data

---

---

## 🧹 2. Geheugen automatisch opschonen

Om te voorkomen dat het geheugen vol raakt, is het goed om verouderde berichten periodiek te verwijderen.

### ⏰ Periodiek legen (bijvoorbeeld elke week)

#### Methode 1: via een aparte `Execute Query` node in n8n  
#### Methode 2: met een Supabase cron job (via edge function of schedule)

---

### 📥 SQL-query om oude memory te verwijderen

Bijvoorbeeld: alles ouder dan 7 dagen wissen

```sql
DELETE FROM chat_memory
WHERE created_at < NOW() - INTERVAL '7 days';

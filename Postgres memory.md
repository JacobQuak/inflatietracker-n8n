# 🧠 Postgres Chat Memory instellen en onderhouden in Supabase

---

## 📦 Wat is Postgres Chat Memory?

De `Postgres Chat Memory` node in n8n houdt gesprekken (context) bij voor een AI-agent. Denk hierbij aan:

- Vorige vragen & antwoorden  
- Interne taken of bewerkingen  
- AI-instructies die ‘onthouden’ moeten worden tussen stappen  

Deze geheugenopslag gebeurt in een **PostgreSQL-tabel** in Supabase.

---

## ✅ 1. Postgres Chat Memory instellen

### 📘 Voorbereiding in Supabase

Maak een nieuwe tabel aan met de volgende structuur:

| Kolomnaam   | Type      | Opmerking                   |
|-------------|-----------|-----------------------------|
| `id`        | UUID      | Primary key                 |
| `session_id`| text      | Groepeert de sessies        |
| `type`      | text      | ‘human’ of ‘ai’             |
| `content`   | text      | Inhoud van het bericht      |
| `created_at`| timestamp | Tijdstip van opslag         |

➡️ Je kunt eventueel extra velden toevoegen zoals `metadata` (`jsonb`) als je dat nodig hebt.

---

### 🔗 Configuratie in n8n

- **Node**: `Postgres Chat Memory`
- Verbind met je Supabase database (via credentials)
- Geef het volgende op:

  - **Tabelnaam**: `chat_memory` (of jouw eigen naam)
  - **Session ID**: bijvoorbeeld `={{ $runId }}` of `={{ $json.userId }}`
  - **Velden toewijzen**:
    - `inputField`: AI-invoer of prompt
    - `outputField`: AI-uitvoer (response)

🔁 Nu worden alle interacties automatisch opgeslagen bij het uitvoeren van de workflow.

---

## 🧹 2. Geheugen automatisch opschonen

Om te voorkomen dat het geheugen blijft groeien, is het goed om verouderde berichten periodiek te verwijderen.

### ⏰ Periodiek legen (bijvoorbeeld elke week)

#### Methode 1: via een aparte `Execute Query` node in n8n  
#### Methode 2: met een Supabase cron job (via edge function of schedule)

---

### 📥 SQL-query om oude memory te verwijderen

Bijvoorbeeld: alles ouder dan 7 dagen wissen

```sql
DELETE FROM chat_memory
WHERE created_at < NOW() - INTERVAL '7 days';

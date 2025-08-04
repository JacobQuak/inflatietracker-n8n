# 📊 CPI/PPI-data importeren in Supabase Vector Store met n8n

Deze handleiding beschrijft hoe je met behulp van **n8n**, **OpenAI Embeddings** en **Supabase** automatisch CPI- of PPI-data (zoals uit CBS-bestanden) omzet in tekst, embedt en opslaat als vector data in een Supabase-table.

---

## 🎯 Doel

- ✅ Excelbestand automatisch downloaden uit OneDrive
- ✅ Data extraheren en transformeren naar natuurlijke taal
- ✅ Embeddings genereren via OpenAI
- ✅ Opslaan in Supabase als vector records (`content`, `metadata`, `embedding`)

---

## 🧰 Benodigdheden

- n8n installatie (cloud of self-hosted)
- Geconfigureerde Microsoft OneDrive OAuth2 verbinding in n8n
- Supabase-project met een tabel genaamd `documents` met kolommen:
  - `content`: tekst
  - `metadata`: JSONB
  - `embedding`: vector[]
- OpenAI API key met toegang tot `text-embedding-ada-002`

---

## 🔁 Workflow-stappen in n8n

### 1. Trigger

Gebruik bijvoorbeeld een `Manual Trigger` of een geplande trigger (`Cron`) om de flow automatisch te starten.

---

### 2. Download CPI-bestand vanuit OneDrive

**Node**: `Microsoft OneDrive`  
**Operation**: `Download`  
- Gebruik de **file ID** van het gewenste Excelbestand  
- Output: `binary data` (meestal `data` genoemd)

#### 🔍 Bestand-ID achterhalen?
Gebruik een OneDrive `Search` node met als query een deel van de bestandsnaam. Het resultaat bevat de `id` die je kunt gebruiken in de `Download` node.

---

### 3. Extract from File (Excel parsing)

**Node**: `Extract from File`  
**Instellingen**:
- **Operation**: `xlsx`
- **Binary Property Name**: `data` (of hoe je bestand heet)

Resultaat: een JSON-array met rijen zoals:

```json
{
  "Bestedingscategorie": "Huisvesting, water en energie",
  "februari 2025": 0.98,
  "januari 2025": 0.92
}
```

---

### 4. Genereer tekst met Default Data Loader

**Node**: `Default Data Loader`  
- **Type of Data**: `JSON`
- **Mode**: `Load Specific Data`
- **Data**:

```n8n
{{ $json.Bestedingscategorie }} in februari 2025 is {{ $json["februari 2025"] }}
```

- **Text Splitting**: `Custom` of `None`

Uitvoer per item:

```json
{
  "text": "Huisvesting, water en energie in februari 2025 is 0.98",
  "Bestedingscategorie": "...",
  "februari 2025": 0.98
}
```

---

### 5. Genereer embeddings met OpenAI

**Node**: `OpenAI`  
- **Resource**: `Embeddings`
- **Model**: `text-embedding-ada-002`
- **Input**: `={{ $json.text }}`

Voegt een `embedding` array toe aan elk item.

---

### 6. Sla op in Supabase Vector Store

**Node**: `Supabase Vector Store` (of HTTP Request naar Supabase REST API)

Velden invullen als volgt:

- **content**: `{{ $json.text }}`
- **metadata**:

```json
{
  "categorie": "{{ $json.Bestedingscategorie }}",
  "maand": "februari 2025",
  "waarde": {{ $json["februari 2025"] }}
}
```

- **embedding**: `{{ $json.embedding }}`

📥 Dit schrijft 1 rij per CPI-item weg in Supabase zoals:

| content | metadata | embedding |
|--------|----------|-----------|
| "Huisvesting, water en energie in februari 2025 is 0.98" | `{...}` | `[0.03, 0.065, ...]` |

---

## ✅ Resultaat

Na het uitvoeren van de workflow staat je CPI/PPI dataset als vector data klaar in Supabase – perfect voor AI-toepassingen zoals:

- Retrieval-Augmented Generation (RAG)
- Semantische zoekmachines
- Trendanalyse op basis van natuurlijke taal

---

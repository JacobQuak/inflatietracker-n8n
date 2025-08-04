# 📊 CPI-data laden vanuit OneDrive naar Supabase Vector Store (n8n Handleiding)

Deze handleiding beschrijft hoe je een `.xlsx`-bestand met CPI- of PPI-data automatisch downloadt vanuit OneDrive, verwerkt in n8n, en opslaat in een Supabase vector store. Dit is nuttig voor toepassingen zoals Retrieval-Augmented Generation (RAG), waarin je AI kunt koppelen aan actuele prijsindexaties per bestedingscategorie.

---

## 🎯 Doel

- 📁 Inlezen van een CPI-bestand vanuit OneDrive
- 🔍 Extractie van de relevante velden (categorieën en percentages)
- 🧠 Omzetten naar tekst en embeddings genereren
- 📦 Opslaan in Supabase als vector data voor latere AI-toepassing

---

## 🛠 Voorbereiding

Zorg dat je het volgende hebt ingesteld:

- Een werkende n8n-installatie (self-hosted of cloud)
- Microsoft OneDrive OAuth2-verbinding in n8n
- Supabase account met een vector table (`documents`)
- OpenAI API key voor embeddings (of alternatief AI-model)

---

## 🔧 Workflow-stappen in n8n

### 1. Trigger (startpunt)

Gebruik een `Manual Trigger` of bijvoorbeeld een geplande trigger (elke week of maand).

---

### 2. OneDrive: Download bestand

**Node**: `Microsoft OneDrive`  
**Instellingen**:
- Operation: `Download`
- File ID: gebruik de ID van het CPI-bestand in OneDrive
- Output: binaire data (`data`)

### 🔍 Hoe achterhaal je een bestand-ID in OneDrive via n8n?

Als je de **file ID** van een OneDrive-bestand nodig hebt (bijvoorbeeld voor een download-operatie), kun je deze eenvoudig ophalen met de **Search** node.

#### 📦 Node: `Microsoft OneDrive`

**Instellingen**:

- **Operation**: `Search`
- **Query**: De naam of een deel van de bestandsnaam (bijv. `CPI` of `document2025.pdf`)
- **Return All**: `true` (optioneel, als je meerdere matches verwacht)

#### 🧾 Output

De node retourneert een lijst van bestanden. Elk item bevat o.a.:

- `"id"` → Dit is de **file ID** die je nodig hebt
- `"name"` → De bestandsnaam
- `"parentReference"` → Locatie/pad in OneDrive

---

### 3. Extract from File

**Node**: `Extract from File`  
**Instellingen**:
- Operation: `xlsx`
- Binary property name: `data`

Hiermee haal je de tabellen uit het Excelbestand. Bijv. dit formaat:

| Bestedingscategorie                     | feb_2025 | jan_2025 |
|----------------------------------------|----------|----------|
| Huisvesting, water en energie          | 0.98     | 0.92     |
| Alcohol en tabak                       | 0.58     | 0.61     |
| ...                                    | ...      | ...      |

---

### 4. Format data voor embeddings

**Node**: `Code`

Gebruik onderstaande JavaScript-code om elke rij om te zetten naar tekst:

```js
return items.map(item => {
  return {
    json: {
      text: `Het PPI van productcategorie ${item.json.Bestedingscategorie} in februari 2025 is ${item.json.feb_2025}`,
      Bestedingscategorie: item.json.Bestedingscategorie,
      maand: "feb_2025",
      waarde: item.json.feb_2025
    }
  }
});

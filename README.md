# 📊 n8n Inflatie-analyse Workflow

Deze n8n-workflow automatiseert de verwerking van inflatie-informatie uit PDF-brieven die als bijlage binnenkomen via Microsoft Outlook. De workflow extraheert productnamen en inflatiepercentages, koppelt deze via een Supabase Vector Store aan productcategorieën, vergelijkt ze met indexatiepercentages, en genereert indien nodig grafieken en e-mails.

## 🔁 Workflow-overzicht

1. **Trigger:** Ontvangen van e-mailbijlagen (PDF) via Outlook.
2. **Extractie:** OCR en parsing van inflatiegegevens uit PDF.
3. **AI-verwerking:** Gebruik van GPT-4o-mini om product + inflatie te extraheren.
4. **Koppeling aan categorie:** AI-koppeling van producten aan CPI-categorieën (via Supabase Vector Store).
5. **Vergelijking met indexatie:** Analyse op afwijkingen.
6. **Visualisatie:** Genereert bar charts met QuickChart.
7. **Notificatie:** Stuurt automatisch HTML-e-mail naar Outlook als de inflatie hoger is dan de indexatie.

## 📂 Structuur

- `workflows/inflatie-workflow.json`  
  Bevat de geëxporteerde n8n-workflow.

- `media/diagram.png`  
  Optionele visualisatie van de workflow voor documentatie.

## ⚙️ Benodigdheden

- n8n self-hosted of cloud instance
- Microsoft Outlook credentials
- Supabase instance met categorie-data
- OpenAI API key (GPT-4o-mini)
- QuickChart endpoint (voor grafiekgeneratie)

## 📦 Importeren in n8n

1. Ga naar je n8n dashboard.
2. Klik op “Import Workflow”.
3. Selecteer het bestand `workflows/inflatie-workflow.json`.

## 📜 Licentie

MIT (optioneel toevoegen aan `LICENSE`)

---

*Laatste update: {{vul hier de datum in}}*

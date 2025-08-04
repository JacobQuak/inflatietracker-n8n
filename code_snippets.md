# 🧩 Code Snippets – GN-Code-classifier (n8n)

Deze pagina bevat alle belangrijke codefragmenten die gebruikt worden in de GN-Code-classifier workflow.

---

## 📦 Code 1 – Extractie van producten uit OpenAI-output

```javascript
// Haal de producten array uit de OpenAI-output
const products = $json.message.content.products;

return products.map(product => ({
  json: {
    name: product.name,
    inflation_percentage: product.inflation_percentage
  }
}));
```

---

## 📊 Code 3 – Controle op afwijkingen tussen inflatie en indexatie

```javascript
const afwijkingen = [];

for (const item of $input.all()) {
  const naam = item.json.name || 'Onbekend product';
  const inflatieStr = item.json.inflation_percentage || '';
  const indexatieStr = item.json.Indexatie || item.json.indexatie_categorie || item.json.Category || '';

  const inflatie = parseFloat(inflatieStr.toString().replace(',', '.').replace('%', ''));
  const indexatie = parseFloat(indexatieStr.toString().replace(',', '.'));

  if (inflatie > indexatie) {
    afwijkingen.push({
      json: {
        name: naam,
        inflatie: inflatie,
        indexatie: indexatie
      }
    });
  }
}

return afwijkingen;
```

---

## 📦 Code 4 – Bundel alle afwijkingen in één array

```javascript
// Verwerk alle afwijkingen in één array
const producten = $input.all().map(item => ({
  name: item.json.name,
  inflatie: item.json.inflatie,
  indexatie: item.json.indexatie
}));

return [{
  json: {
    producten
  }
}];
```

---

## 🧠 Code 5 – Extractie van product, categorie en indexatie uit GPT-output

```javascript
return items.map(item => {
  const raw = item.json.message?.content ?? '';
  let product = null;
  let category = null;
  let indexatie = null;

  try {
    // Probeer JSON-structuur eruit te halen met regex
    const match = raw.match(/{[\s\S]*}/); // pakt eerste JSON-achtige blok
    if (match) {
      const parsed = JSON.parse(match[0]);

      // Pak de eerste (en enige) key als productnaam
      const keys = Object.keys(parsed);
      if (keys.length === 1) {
        product = keys[0];
        const inner = parsed[product];
        if (inner && typeof inner === 'object') {
          category = inner.categorie ?? null;

          // Zoek numerieke waarde of parsebare string
          const indexKey = Object.keys(inner).find(k =>
            k.toLowerCase().includes('2025') || k.includes('index')
          );

          if (indexKey) {
            indexatie = parseFloat(inner[indexKey]);
          }
        }
      }
    }
  } catch (err) {
    // fallback
    return {
      json: {
        error: "Parsing failed",
        raw
      }
    };
  }

  return {
    json: {
      product,
      category,
      indexatie_categorie: indexatie
    }
  };
});
```

# 🧪 n8n Workflow - Champagne Product Description Rewrite

## 📌 Objectif

Ce workflow `n8n` permet de :

- Récupérer des fiches produits depuis une source HTTP
- Générer automatiquement des descriptions percutantes via GPT (OpenAI)
- Fusionner les descriptions enrichies avec les données originales
- Générer un fichier JSON final prêt à exporter

---

## 🧱 Structure du Workflow (Briques)

### 🔹 Brique 1 — Récupération des données HTTP

**Type :** HTTP Request  
Récupère un tableau JSON contenant les fiches produits :

<details open>
<summary>🪟 Aperçu JSON formaté</summary>

<br>

```json
[
  {
    "name": "Champagne Drappier - Brut Nature & étui",
    "description": "Un champagne sans dosage, droit, minéral, idéal pour les puristes.",
    "url": "https://example.com/champagne-drappier",
    "image": "https://example.com/image.jpg",
    "tags": ["brut", "nature", "champagne"],
    "pairings": {
      "cheese": ["comté", "chèvre"],
      "seafood": ["huîtres", "sashimi"]
    }
  }
]
return items.map(({ json }) => {
  const description = json.original_description || json.description || '';

  const prompt = description.trim()
    ? `Voici une fiche produit brute :\n"${description}"\n\nRéécris une description unique et percutante pour le produit ${json.name}. Utilise un ton captivant, une touche de storytelling, et le champ lexical du champagne pour séduire un amateur de vin exigeant.`
    : null;

  return {
    json: {
      ...json,
      prompt,
      product: {
        name: json.name || 'Inconnu',
        url: json.url || '',
        original_description: description,
        image: json.image || null,
      }
    }
  };
});
🔹 Brique 3 — Appel à OpenAI
Type : OpenAI Node (GPT-4)
Utilise le champ prompt pour générer une nouvelle description_rewrite.
const produits = $items("brique 2");
const reponses = items;

return produits.map((prod, index) => {
  const original = prod.json;
  const reponse = reponses[index]?.json?.message?.content || '';

  return {
    json: {
      ...original.product,
      description_rewrite: reponse,
    }
  };
});
const produits = items
  .map(i => i.json)
  .filter(p => typeof p.description_rewrite === 'string' && p.description_rewrite.trim() !== '');

if (produits.length === 0) {
  throw new Error("Aucun produit avec 'description_rewrite' valide trouvé.");
}

const buffer = Buffer.from(JSON.stringify({ produits }, null, 2), 'utf8');

return [{
  json: { fileName: 'champagnes.json' },
  binary: {
    data: {
      data: buffer.toString('base64'),
      mimeType: 'application/json',
      fileName: 'champagnes.json'
    }
  }
}];
Conseils & Astuces
Limitez à 3 produits max pendant les tests pour éviter les quotas GPT

Vérifiez l'encodage UTF-8 du fichier final

Filtrez avec description_rewrite.trim() !== ''

Ajoutez un nœud Debug après chaque étape pour observer l'état

📂 Export / Import
Vous pouvez exporter ce workflow depuis n8n via Download as .json
Ou l’importer dans un autre environnement via Import from File

✨ Résultat Attendu
{
  "produits": [
    {
      "name": "...",
      "url": "...",
      "original_description": "...",
      "image": "...",
      "description_rewrite": "..."
    }
  ]
}

# DossierFrankrijk Tool Integratie

> Instructies voor het toevoegen van de "Opslaan in Dossier" functionaliteit aan InfoFrankrijk tools.

---

## Hoe het werkt

```
┌─────────────────────┐    postMessage    ┌─────────────────────┐
│   Jouw Tool         │ ───────────────▶  │  InfoFrankrijk.com  │
│   (iframe)          │                   │  (WordPress)        │
│                     │                   │                     │
│  [📁 Opslaan]       │    data:          │  WordPress Snippet  │
│                     │    - title        │  ontvangt data      │
│                     │    - summary      │  ↓                  │
│                     │    - source       │  Toont modal        │
│                     │                   │  ↓                  │
│                     │                   │  Slaat op in        │
│                     │                   │  Supabase           │
└─────────────────────┘                   └─────────────────────┘
```

De tool draait in een iframe op InfoFrankrijk.com. Wanneer de gebruiker klikt op "Opslaan in Dossier", stuurt de tool een `postMessage` naar het parent window. De WordPress snippet vangt dit op en toont een modal waar de gebruiker een map kan kiezen.

---

## Stap 1: Voeg de knop toe aan je HTML

```html
<button id="save-dossier-btn" class="primary">📁 Opslaan in Dossier</button>
```

### Optionele CSS (als je geen bestaande styling hebt)

```css
#save-dossier-btn {
    background: #800000;
    color: #fff;
    border: none;
    padding: 12px 20px;
    font-size: 14px;
    font-weight: 600;
    border-radius: 6px;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    gap: 8px;
    transition: background 0.2s;
}

#save-dossier-btn:hover {
    background: #a00000;
}

#save-dossier-btn:disabled {
    background: #ccc;
    cursor: not-allowed;
}
```

---

## Stap 2: Voeg de JavaScript toe

Plaats deze code aan het **einde** van je JavaScript bestand, of in een `<script>` tag voor `</body>`.

```javascript
// =============================================================================
// DOSSIERFRANKRIJK INTEGRATIE
// Versie: 1.0
// Laatste update: januari 2026
// =============================================================================

(function() {
    'use strict';
    
    var btn = document.getElementById('save-dossier-btn');
    if (!btn) return;
    
    btn.addEventListener('click', function() {
        
        // =====================================================================
        // CONFIGURATIE - PAS DIT AAN VOOR JOUW TOOL
        // =====================================================================
        
        var TOOL_NAAM = 'Tool Naam';           // Bijv: 'Financieel Kompas'
        var TOOL_SLUG = 'tool-naam';           // Bijv: 'financieel-kompas'
        
        // =====================================================================
        // VERZAMEL DATA - PAS DIT AAN VOOR JOUW TOOL
        // =====================================================================
        
        // Haal waarden op uit je formulier/berekening
        // Voorbeeld:
        // var waarde1 = document.getElementById('veld1').value;
        // var waarde2 = document.getElementById('veld2').value;
        // var resultaat = berekenResultaat();
        
        // =====================================================================
        // BOUW DE TITLE
        // Format: "Tool Naam: korte beschrijving | resultaat"
        // =====================================================================
        
        var title = TOOL_NAAM + ': beschrijving | resultaat';
        
        // Voorbeeld Financieel Kompas:
        // var title = 'Financieel Kompas: ' + huishouden + ' | NL vs FR | ' + verschil;
        
        // =====================================================================
        // BOUW DE SUMMARY
        // Dit is de volledige output die in het dossier komt
        // =====================================================================
        
        var datum = new Date().toLocaleDateString('nl-NL');
        
        var summary = '-------------------------------------------\n' +
            TOOL_NAAM.toUpperCase() + ' - RESULTAAT\n' +
            'Datum: ' + datum + '\n' +
            '-------------------------------------------\n\n' +
            'INVOERGEGEVENS\n' +
            '• Veld 1: waarde\n' +
            '• Veld 2: waarde\n' +
            '• Veld 3: waarde\n\n' +
            'BEREKENING\n' +
            '• Resultaat 1: waarde\n' +
            '• Resultaat 2: waarde\n\n' +
            '-------------------------------------------\n' +
            'CONCLUSIE: samenvatting resultaat\n' +
            '-------------------------------------------\n\n' +
            'Let op: Dit is een indicatieve berekening.\n' +
            'Raadpleeg een adviseur voor uw specifieke situatie.\n';
        
        // =====================================================================
        // VERSTUUR NAAR INFOFRANKRIJK - NIET AANPASSEN
        // =====================================================================
        
        var data = {
            type: 'saveToDossier',
            title: title,
            summary: summary,
            source: TOOL_SLUG
        };
        
        // Check of we in een iframe draaien
        if (window.parent !== window) {
            window.parent.postMessage(data, '*');
        } else {
            // Standalone modus - toon melding
            alert('Deze functie werkt alleen binnen InfoFrankrijk.com\n\nGa naar infofrankrijk.com om deze tool te gebruiken.');
        }
    });
    
})();
```

---

## Stap 3: Pas de configuratie aan

### 3.1 Tool identificatie

```javascript
var TOOL_NAAM = 'Energiekompas';        // Weergavenaam
var TOOL_SLUG = 'energiekompas';        // Unieke identifier (geen spaties, lowercase)
```

### 3.2 Title format

De title verschijnt als titel in het dossier. Houd het kort en informatief:

```javascript
// Goed:
'Financieel Kompas: Alleenstaand | NL vs FR | +€1.507'
'Energiekompas: 8500 kWh | €2.340/jaar'
'Vastgoed Dashboard: Dordogne | €185.000 | 4 slaapkamers'

// Vermijd:
'Berekening gemaakt op 27 januari 2026 met het Financieel Kompas voor...'
```

### 3.3 Summary format

De summary is de volledige output. Gebruik een consistent format:

```
-------------------------------------------
TOOL NAAM - TYPE RESULTAAT
Datum: DD-MM-YYYY
-------------------------------------------

SECTIE 1
• Item: waarde
• Item: waarde

SECTIE 2
• Item: waarde
• Item: waarde

-------------------------------------------
CONCLUSIE: korte samenvatting
-------------------------------------------

Disclaimer of extra info.
```

---

## Stap 4: Source slugs

De `source` bepaalt welk icoon in DossierFrankrijk wordt getoond:

| Source slug | Icoon | Gebruik voor |
|-------------|-------|--------------|
| `infofrankrijk` | 🤖 | AI samenvattingen van artikelen |
| `forum` | 🤖 | Forum berichten |
| `nedergids` | 🤖 | Nedergids content |
| `extern` | 🔗 | Externe links |
| `notitie` | 📝 | Eigen notities |
| **alle andere** | ⚙️ | **Tool output** |

### Geregistreerde tool slugs:

| Tool | Slug |
|------|------|
| Financieel Kompas | `financieel-kompas` |
| Energiekompas | `energiekompas` |
| Vastgoed Dashboard | `vastgoed-dashboard` |
| Gîte Calculator | `gite-calculator` |
| Beroepenchecker | `beroepenchecker` |
| Locatie Analyse | `locatie-analyse` |
| Overheden Zoeker | `overheden-zoeker` |

> **Nieuwe tool?** Kies een slug in kebab-case (lowercase met streepjes) en voeg toe aan deze lijst.

---

## Volledig voorbeeld: Energiekompas

```javascript
(function() {
    'use strict';
    
    var btn = document.getElementById('save-dossier-btn');
    if (!btn) return;
    
    btn.addEventListener('click', function() {
        
        // Configuratie
        var TOOL_NAAM = 'Energiekompas';
        var TOOL_SLUG = 'energiekompas';
        
        // Haal waarden op
        var verbruikElektra = document.getElementById('elektra-kwh').value || '0';
        var verbruikGas = document.getElementById('gas-m3').value || '0';
        var kostenJaar = document.getElementById('totaal-kosten').textContent || '€0';
        var woningType = document.getElementById('woning-type').value || 'Onbekend';
        var oppervlakte = document.getElementById('oppervlakte').value || '0';
        
        // Bouw title
        var title = TOOL_NAAM + ': ' + verbruikElektra + ' kWh + ' + verbruikGas + ' m³ | ' + kostenJaar + '/jaar';
        
        // Bouw summary
        var datum = new Date().toLocaleDateString('nl-NL');
        
        var summary = '-------------------------------------------\n' +
            'ENERGIEKOMPAS - VERBRUIKSANALYSE\n' +
            'Datum: ' + datum + '\n' +
            '-------------------------------------------\n\n' +
            'WONINGGEGEVENS\n' +
            '• Type: ' + woningType + '\n' +
            '• Oppervlakte: ' + oppervlakte + ' m²\n\n' +
            'VERBRUIK\n' +
            '• Elektriciteit: ' + verbruikElektra + ' kWh/jaar\n' +
            '• Gas: ' + verbruikGas + ' m³/jaar\n\n' +
            'KOSTEN (geschat)\n' +
            '• Totaal per jaar: ' + kostenJaar + '\n\n' +
            '-------------------------------------------\n' +
            'Dit is een indicatieve berekening op basis\n' +
            'van gemiddelde Franse energietarieven.\n' +
            '-------------------------------------------\n';
        
        // Verstuur
        var data = {
            type: 'saveToDossier',
            title: title,
            summary: summary,
            source: TOOL_SLUG
        };
        
        if (window.parent !== window) {
            window.parent.postMessage(data, '*');
        } else {
            alert('Deze functie werkt alleen binnen InfoFrankrijk.com');
        }
    });
    
})();
```

---

## Testen

### Lokaal testen (zonder InfoFrankrijk)

De knop toont een alert omdat `window.parent === window`. Dit is normaal gedrag.

### Testen op InfoFrankrijk

1. Deploy je tool naar GitHub Pages
2. Embed de tool in een InfoFrankrijk pagina via iframe
3. Log in als abonnee
4. Klik op "Opslaan in Dossier"
5. De modal moet verschijnen met mapkeuze
6. Na opslaan: check https://dossierfrankrijk.nl/dashboard
7. Verifieer dat het icoon ⚙️ is (niet 🔗)

### Database check (voor developers)

```sql
SELECT id, title, source, created_at 
FROM items 
WHERE source = 'jouw-tool-slug'
ORDER BY created_at DESC 
LIMIT 5;
```

---

## Troubleshooting

### Knop doet niets

- Check browser console voor errors
- Verifieer dat `id="save-dossier-btn"` exact klopt
- Zorg dat het script NA de button wordt geladen

### Modal verschijnt niet

- Ben je ingelogd als abonnee op InfoFrankrijk?
- Draait de tool in een iframe? (niet standalone)
- Check console voor postMessage errors

### Verkeerd icoon (🔗 in plaats van ⚙️)

- Controleer de `source` waarde
- Source mag NIET zijn: `infofrankrijk`, `forum`, `nedergids`, `extern`, `notitie`
- Check database: `SELECT source FROM items WHERE title LIKE '%jouw tool%'`

### Data komt niet aan

- WordPress snippet moet actief zijn (Code Snippets → InfoFrankrijk Tools v4)
- Check of `type: 'saveToDossier'` exact zo is geschreven
- Controleer browser console op beide windows (tool + parent)

---

## Bestanden

```
/jouw-tool/
├── index.html          # Bevat de button
├── style.css           # Bevat button styling
├── script.js           # Bevat tool logica + DossierFrankrijk integratie
└── README.md           # Link naar deze instructies
```

---

## Support

- **Repository**: github.com/antonnoe
- **WordPress snippet**: Code Snippets → InfoFrankrijk Tools v4
- **Database**: Supabase → qhilltwegdwygejwoykt → items table

---

*Laatste update: januari 2026*

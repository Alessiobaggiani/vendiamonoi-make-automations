# Analisi API SellRapido — Vendiamonoi
*Compilata: 12/03/2026*

---

## Indice API disponibili

| API | Metodo | Endpoint | Uso |
|-----|--------|----------|-----|
| Estrazione ordini | POST | `/api/order/{api_key}` | Scarica ordini con filtri |
| Aggiornamento stato + tracking | POST | `/api/order/{api_key}` | Aggiorna stato/tracking su ordini |
| Export prodotti/catalogo | GET | `/api/export/product/{api_key}` | Scarica catalogo prodotti |

Base URL: `https://app.sellrapido.com/sr_company_ws`

---

## 1. API Estrazione Ordini

**Endpoint:** `POST https://app.sellrapido.com/sr_company_ws/api/order/{api_key}`
**Content-Type:** `application/json`

### Parametri richiesta (tutti opzionali)

| Parametro | Tipo | Descrizione | Note |
|-----------|------|-------------|------|
| `startDate` | string `"yyyy-mm-dd"` | Data inizio ordine | Filtra per data ORDINE |
| `endDate` | string `"yyyy-mm-dd"` | Data fine ordine | Filtra per data ORDINE |
| `startModified` | string `"yyyy-mm-dd"` | Data inizio ultima modifica | ⭐ Filtra per data MODIFICA — chiave per tracking status change |
| `endModified` | string `"yyyy-mm-dd"` | Data fine ultima modifica | ⭐ Filtra per data MODIFICA |
| `code` | array | Codici ordine specifici | Estrazione selettiva |
| `format` | string | `"csv"` o `"json"` | Default: json |
| `status` | array | Filtra per stati: `["standby", "accepted", "sent", "cancelled"]` | ⚠️ Vedi nota stati |
| `tags` | array | Filtra per tag: `["tag1", "tag2"]` | Logica OR |
| `offset` | int | Primo record da estrarre | Per paginazione |
| `limit` | int | Numero record da estrarre (`-1` = tutti) | Default: tutti |
| `columnSeparator` | string | Separatore colonne CSV | Solo per format csv |
| `writeHeading` | bool | Stampa intestazioni colonne | Default: true, solo CSV |

### Struttura risposta JSON

```json
{
  "orders": [
    {
      "head": {
        "code": "SR-12345",
        "channel_code": "AMAZON_IT",
        "status": "standby",
        "date_order": "2026-03-10",
        "create_date": "2026-03-10T09:00:00",
        "buyer_name": "Mario Rossi",
        "buyer_email": "mario@example.com",
        "buyer_address1": "Via Roma 1",
        "buyer_city": "Milano",
        "buyer_zip": "20100",
        "buyer_country": "IT",
        "price": 29.90,
        "shipping_price": 3.90,
        "marketplace_fee": 4.50,
        "tracking": "1Z999AA10123456784",
        "courier": "UPS"
      },
      "rows": [
        {
          "sku": "SKU-001",
          "ean": "8012345678901",
          "title": "Nome Prodotto",
          "quantity": 1,
          "price": 29.90,
          "price_cost": 15.00,
          "tracking": "1Z999AA10123456784"
        }
      ]
    }
  ]
}
```

### ⚠️ Nota critica: stati ordine

I valori status documentati nell'API sono: `standby`, `accepted`, `sent`, `cancelled`

**Lo scenario attuale usa `["standby", "accepted", "shipped", "delivered"]`.**
`shipped` e `delivered` potrebbero NON essere valori validi per il filtro — il valore corretto per "spedito" è probabilmente `sent`.
**Da verificare con SellRapido support o analizzando gli ordini reali spediti.**

---

## 2. API Aggiornamento Stato Ordini

**Endpoint:** `POST https://app.sellrapido.com/sr_company_ws/api/order/{api_key}`
**Stessa URL dell'estrazione, corpo diverso per update**

### Parametri per aggiornamento (body array di oggetti)

| Campo | Tipo | Obbligatorio | Descrizione |
|-------|------|-------------|-------------|
| `code` | string | ✅ | Codice ordine SellRapido da aggiornare |
| `status` | string | ✅ | Nuovo stato: `standby`, `accepted`, `sent`, `cancelled` |
| `courier_code` | string | No (richiede tracking) | Codice corriere |
| `tracking` | string | No (richiede courier_code) | Codice tracking spedizione |
| `payment_date` | string | No | Data pagamento (non rimovibile) |

### Corrieri supportati

`AWS`, `BARTOLINI1`, `DBSCHENKER`, `DHL`, `EBOOST`, `FEDEX`, `FERCAM`, `GLS`, `GLSFIX`, `MBE`, `NEXIVE_EXP`, `NEXIVE_REG`, `POSTE_ITALIANE_CRONO`, `SDA`, `SOGETRAS`, `SPEDIAMO.IT`, `TNT`, `UPS`

### Comportamento

- ✅ **Bulk update**: un'unica chiamata può aggiornare più ordini (array di oggetti)
- ✅ Possibile aggiornare tracking + courier in una volta
- ❌ Tracking/courier non rimovibili dopo impostazione
- ❌ payment_date non rimovibile
- Risposta: solo gli **errori** (i successi non appaiono nella risposta)

### Esempio request body

```json
[
  {
    "code": "SR-12345",
    "status": "sent",
    "courier_code": "GLS",
    "tracking": "987654321"
  },
  {
    "code": "SR-12346",
    "status": "accepted"
  }
]
```

---

## 3. API Export Prodotti

**Endpoint:** `GET https://app.sellrapido.com/sr_company_ws/api/export/product/{api_key}`

### Parametri

| Parametro | Tipo | Default | Descrizione |
|-----------|------|---------|-------------|
| `fields` | array | Vedi sotto | Campi da includere nell'export |
| `warehouse` | string | tutti | Codice magazzino/catalogo specifico |
| `field_separator` | string | `"\|"` | Separatore colonne CSV |
| `zip` | bool | `true` | `false` = download CSV non zippato |

### Campi default restituiti

```json
["sku", "ean", "mpn", "brand", "asin",
 "catalog_category1", "catalog_category2", "catalog_category3",
 "quantity", "price1", "price_shipping1", "delivery_days",
 "title", "description",
 "url_image1", "url_image2", "url_image3", "url_image4"]
```

---

## 4. Automatismo Corrieri (via Qapla')

- SellRapido offre automazione spedizioni tramite integrazione nativa con **Qapla'**
- Estrazione automatica ordini verso i corrieri
- Richiede: contratto attivo con corriere nel circuito Qapla' + attivazione da ticket support SellRapido
- Incluso nel piano Manager
- Configurazione personalizzata per ogni magazzino/client code

---

## 5. Opportunità e ottimizzazioni per scenario Make

### 🔴 Problema attuale: filtro `startDate` inefficiente

Lo scenario v1.2 usa `startDate: addDays(now; -30)` → ri-processa ~30-100 ordini ogni 15 min.
La maggior parte di questi ordini non è cambiata. Spreco di operazioni Make.

### ✅ Ottimizzazione: usa `startModified`

```json
{
  "startModified": "{{formatDate(addHours(now; -2); \"YYYY-MM-DD\")}}",
  "format": "json",
  "status": ["standby", "accepted", "sent", "cancelled"]
}
```

- `startModified` filtra per **data ultima modifica**, non data ordine
- Solo gli ordini con status/dati cambiati nelle ultime 2h vengono restituiti
- Riduzione drastica delle operazioni Make per run
- Finestra 2h = sicurezza contro brevi downtime Make

**⚠️ Problema `startModified` con date**: Come `startDate`, `startModified` accetta solo `"yyyy-mm-dd"` (senza ora). Una finestra di -2h si traduce in "tutto oggi" se siamo nella stessa giornata → solo a mezzanotte è rilevante. Per catture intra-day usa `-1` giorno come compromesso.

**Alternativa pratica**: mantieni 30gg ma aggiungi `startModified` come filtro primario, `startDate` come fallback per ordini storici. O meglio: usa solo 7 giorni di `startDate` + tutti gli stati.

### ✅ Nuovo scenario possibile: Fulfillment — Supabase → SellRapido

Quando un ordine viene processato internamente (spedito dal fornitore):
1. Trigger: riga Supabase `orders.internal_status` cambia a `shipped`
2. Make legge `tracking_number` e `courier` da Supabase
3. Make chiama API SellRapido update con `status: "sent"`, `courier_code`, `tracking`
4. SellRapido notifica automaticamente il marketplace → aggiornamento tracking cliente

### ✅ Nuovo scenario possibile: Sync Catalogo — SellRapido → Supabase

- Export prodotti SellRapido → tabella `products` in Supabase
- Permette: lookup SKU/EAN, calcolo margine (price - price_cost), analisi stock
- Trigger: webhook o schedule giornaliero

### ✅ Verifica stati corretti

Prima di v1.3, verificare se SellRapido restituisce `shipped` o `sent` nel campo `head.status`.
Se restituisce `sent`: aggiornare il filtro status request e il mapping nel modulo 4.

---

## Fonti
- [Estrazione ordini e aggiornamento stato via API](https://wiki.sellrapido.com/it/knowledge/api)
- [Inserire dati tracking e info spedizione](https://wiki.sellrapido.com/it/knowledge/inserire-codice-tracking)
- [Estrazione prodotti tramite API](https://wiki.sellrapido.com/it/knowledge/estrazione-prodotti-tramite-api)
- [Configurazione automatismo corrieri](https://wiki.sellrapido.com/it/knowledge/gestione-automatica-corrieri)
- [API Orders management (EN)](https://wiki.sellrapido.com/en/knowledge/orders-management)

# Changelog — SellRapido → Supabase | Ordini Marketplace

## v1.8 — 2026-03-12
**Fix NaN error modulo 4 + Fix lookup marketplace modulo 3**

### Bug fix: NaN error (modulo 4)
- Root cause: il campo `data` del modulo 4 aveva 8 parentesi di chiusura invece di 7 + un `}}` spurio dopo la chiusura di `marketplace_status`
- Effetto: Make parser legacy interpretava `}}` dopo il `"` finale come un'espressione IML non chiusa → errore "references non-existing module NaN"
- Fix: corretto il template IML con esattamente 7 parentesi bilanciate e nessun `}}` dopo la chiusura

### Fix lookup eBay Italy + Metro Germany (modulo 3)
- Root cause: eBay Italy mandava `marketplace_code='ebay'`, Metro Germany mandava `marketplace_code='metro'` — valori non presenti nella colonna `sellrapido_code`
- Fix Supabase: aggiunta colonna `sellrapido_marketplace_code` nella tabella `marketplaces` con mapping:
  - `ebay_it` → `sellrapido_marketplace_code='ebay'`
  - `metro_de` → `sellrapido_marketplace_code='metro'`
  - + altri marketplace (leroymerlin, rue_du_commerce, eprice, mediamarkt, carrefour, real_de)
- Fix modulo 3: OR query su `sellrapido_code` + `sellrapido_marketplace_code`, ordinamento `asc.nullslast` per priorità specifica

---

## v1.6.1 — 2026-03-12
**Fix IML sbilanciata (marketplace_status)**
- `marketplace_status` nel modulo 4 aveva 30 parentesi chiuse vs 28 aperte → `isinvalid: true`
- Root cause: approccio a 11 `if()` annidati causava Internal Server Error su Make (limite nesting)
- Fix: sostituito con `or()+lower()` a 6 livelli di `if()` — stessa logica EN+IT, parentesi bilanciate 28=28
- Scenario riattivato con `scenarios_activate`
- Test run SUCCESS: 72 operazioni in 3.2s

## v1.6 — 2026-03-12
**Fix import eBay/Metro/ePrice + Mapping status esteso**
- Rimosso filtro status dall'API SellRapido — ora recupera TUTTI gli ordini degli ultimi 30 giorni
- Mapping status esteso a 11 casi (italiano + inglese)
- Cleanup: eliminata tabella diagnostica `sellrapido_channel_codes_log`

## v1.5 — 2026-03-12
**Fix case-insensitive marketplace lookup**
- Modulo 3 ora usa `lower(channel_code)` — SellRapido invia PascalCase/UPPERCASE
- Fix Supabase: `eprice_it` → `epriceit`

## v1.4 — 2026-03-12
**Fix definitivo shipping_address**
- `parseJSON()` non esiste in Make IML — scenario crashava
- Nuovo approccio: string concat dentro `{{...}}`

## v1.3 — 2026-03-12
**Fix critico NaN + status**
- Fix `shipping_address` con `parseJSON()` IML
- Fix status: `sent→shipped` aggiunto

## v1.2 — 2026-03-12
**Fix finestra temporale**
- Da 2 ore a 30 giorni
- Fix fallback `order_date`

## v1.1 — 2026-03-12
**Fix data ordine, stati, commissioni**

## v1.0 — 2026-03-10
**Creazione scenario**
- Primo import 29 ordini storici

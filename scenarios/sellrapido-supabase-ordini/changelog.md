# Changelog — SellRapido → Supabase | Ordini Marketplace

## v3.2 — 2026-03-16
**Finestra 3→1 giorno + architettura 2 scenari**
- Modulo 1: `addDays(now; -3)` → `addDays(now; -1)` — finestra ridotta a 24 ore
- Riduzione crediti ~92%: da ~9.000 op/giorno a ~300 op/giorno
- Creato Scenario B separato (ID: 8867890) per aggiornamento stati ordini
- Scenario B: daily 05:00, finestra 45 giorni, PATCH only su `marketplace_status`, `tracking_number`, `carrier`, `sellrapido_modified_at`
- Entrambi gli scenari in stato inattivo — attivare dopo test

## v3.1 — 2026-03-15
**Fix constraint 409 + scenario ricreato come 8856551**
- Errore `duplicate key value violates unique constraint "uq_order_items_order_sku"` — fix: `ALTER TABLE order_items DROP CONSTRAINT uq_order_items_order_sku`
- Scenario precedente 8856537 era in stato `isinvalid: true` — eliminato e ricreato da zero come 8856551
- Architettura attuale: 5 moduli — `HTTP POST SellRapido → BasicFeeder(orders) → UPSERT orders → BasicFeeder(rows) → UPSERT order_items`
- Upsert via PostgREST HTTP: `on_conflict=marketplace_order_id` per orders, `on_conflict=sellrapido_row_id` per order_items
- Scenario tenuto inattivo in attesa di test manuale

## v2.5 — 2026-03-13
**Refactoring completo: da HTTP a moduli nativi Supabase**
- Rimossi tutti i moduli `http:ActionSendData` per le chiamate Supabase
- Nuova logica INSERT-ONLY: search → count → insert solo se non esiste
- Connessione nativa Supabase: `__IMTCONN__: 13881574` ("Vendiamonoi automazioni")

## v1.8 — 2026-03-12
**Fix NaN error modulo 4 + Fix lookup marketplace modulo 3**
- Fix campo `data` modulo 4: 7 parentesi bilanciate, nessun `}}` finale
- Fix modulo 3: aggiunta colonna `sellrapido_marketplace_code` in Supabase
- OR query `sellrapido_code` + `sellrapido_marketplace_code` con `asc.nullslast`

## v1.6.1 — 2026-03-12
**Fix IML sbilanciata (marketplace_status)**
- `marketplace_status` aveva 30 parentesi chiuse vs 28 aperte — scenario `isinvalid: true`
- Fix: sostituito con 6 livelli `if()` bilanciati
- Test run SUCCESS: 72 operazioni in 3.2s

## v1.6 — 2026-03-12
**Fix import eBay/Metro/ePrice**
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
